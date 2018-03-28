Ember应用程序利用 [依赖注入](https://en.wikipedia.org/wiki/Dependency_injection)
("DI") 设计模式来声明和实例化对象类和它们之间的依赖关系。应用程序和应用程序实例均在Ember的DI实现中担任角色。

一个 [`Application`](https://emberjs.com/api/ember/release/classes/Application) 充当“注册表”的依赖声明。
工厂（即类）被应用程序注册，作为对象实例化时用来“注入”依赖的规则。

一个 [`ApplicationInstance`](https://emberjs.com/api/ember/release/classes/ApplicationInstance) 作为从工厂实例化的对象的“所有者”。
应用程序实例提供了一种“查找”（即实例化和/或检索）对象的方法。

> _注意： 虽然 `Application` 作为应用程序的主要注册表，
但每个 `ApplicationInstance` 同样可以充当注册表.
实例级别的注册对于提供实例级自定义非常有用，
例如功能的A / B测试。_

## Factory Registrations(工厂注册)

工厂可以表示应用程序的任何部分， 如 _路由_, _模板_, 或者自定义类。
每个工厂都通过一个特定的key被注册。例如：index 模板被注册通过key `template:index`,
而application 路由 被注册通过key `route:application`。

注册的 keys 有两个由 (`:`)分割的段。
第一部分是框架工厂类型，第二部分是特定工厂的名称
因此，  `index` 模板的 key是 `template:index`.
Ember 有几个内置的工厂类, 例如 `service`, `route`, `template`, 和 `component`.

您可以简单地注册新的工厂类别来创建自定义的工厂类。例如，要创建一个 `user` 类型,
您只需注册工厂 `application.register('user:user-to-register')`.

工厂注册必须在应用程序或应用程序实例初始化程序中执行（前者更常见）。

例如，应用程序初始化程序可以注册一个 `Logger` 工厂通过 key `logger:main`:

```app/initializers/logger.js
import EmberObject from '@ember/object';

export function initialize(application) {
  let Logger = EmberObject.extend({
    log(m) {
      console.log(m);
    }
  });

  application.register('logger:main', Logger);
}

export default {
  name: 'logger',
  initialize: initialize
};
```

### Registering Already Instantiated Objects(在实例化对象上注册)

默认情况下， Ember 将尝试去实例化一个注册工厂，当它被查找到。
当注册已经实例化的对象而不是类时，
使用该 `instantiate: false` 选项可避免尝试在查找过程中重新实例化它。

在下面的示例中，  `logger` 是一个普通的JavaScript对象，应该在查找时“按原样”返回：

```app/initializers/logger.js
export function initialize(application) {
  let logger = {
    log(m) {
      console.log(m);
    }
  };

  application.register('logger:main', logger, { instantiate: false });
}

export default {
  name: 'logger',
  initialize: initialize
};
```

### Registering Singletons vs. Non-Singletons(注册单例与非单例)

默认情况下， 注册被处理为 "单例".
这仅仅意味着一个实例将在第一次查找时被创建，并且这个实例将被缓存并从随后的查找中返回。

当您希望为每个查找创建新对象时，请使用该 `singleton: false` 选项将工厂注册为非单例。

在下面的例子中， `Message` 类被注册为非单例：

```app/initializers/notification.js
import EmberObject from '@ember/object';

export function initialize(application) {
  let Message = EmberObject.extend({
    text: ''
  });

  application.register('notification:message', Message, { singleton: false });
}

export default {
  name: 'notification',
  initialize: initialize
};
```

## Factory Injections(工厂注入)

一旦工厂被注册，它可以在需要的地方“注入”。

工厂可以被注入进所有的工厂类型通过 *类型注入*. 例如：

```app/initializers/logger.js
import EmberObject from '@ember/object';

export function initialize(application) {
  let Logger = EmberObject.extend({
    log(m) {
      console.log(m);
    }
  });

  application.register('logger:main', Logger);
  application.inject('route', 'logger', 'logger:main');
}

export default {
  name: 'logger',
  initialize: initialize
};
```

这种注入的结果是，所有的`route` 工厂将被注入属性`logger` .
`logger` 的值将来自 `logger:main`工厂.

此示例应用程序中的路由现在可以访问注入的logger：

```app/routes/index.js
import Route from '@ember/routing/route';

export default Route.extend({
  activate() {
    // The logger property is injected into all routes
    this.get('logger').log('Entered the index route!');
  }
});
```

注射也可以通过使用其全键在特定工厂进行：

```js
application.inject('route:index', 'logger', 'logger:main');
```

在这个例子中，logger只会被注入到indx路由中。

注入可以添加到任何需要实例化的类。这包括Ember的所有框架类，如  components, helpers, routes, 和  router.

### Ad Hoc Injections(Ad Hoc 注入)

依赖注入也可以直接使用 `inject`在Ember类中声明。
目前， `inject` 支持注入控制器 (通过 `import { inject } from '@ember/controller';`)
和服务 (通过 `import { inject } from '@ember/service';`).

下面的代码注入 `shopping-cart` 服务到 `cart-contents` 组件中作为 `cart`属性:

```app/components/cart-contents.js
import Component from '@ember/component';
import { inject as service } from '@ember/service';

export default Component.extend({
  cart: service('shopping-cart')
});
```

如果你希望注入和属性同名的服务，只需要使用service作为服务的名字（名字的dasherized版本将被使用）

If you'd like to inject a service with the same name as the property,
simply leave off the service name (the dasherized version of the name will be used):

```app/components/cart-contents.js
import Component from '@ember/component';
import { inject as service } from '@ember/service';

export default Component.extend({
  shoppingCart: service()
});
```

## Factory Instance Lookups(工厂实例查找)

要从正在运行的应用程序中获取工厂的实例，您可以调用
[`lookup`](https://emberjs.com/api/ember/release/classes/ApplicationInstance/methods/lookup?anchor=lookup) 方法在应用程序实例上。 该方法采用字符串来标识工厂并返回适当的对象。

```javascript
applicationInstance.lookup('factory-type:factory-name');
```

The application instance is passed to Ember's instance initializer hooks and it
is added as the "owner" of each object that was instantiated by the application
instance.

### Using an Application Instance Within an Instance Initializer

Instance initializers receive an application instance as an argument, providing
an opportunity to look up an instance of a registered factory.

```app/instance-initializers/logger.js
export function initialize(applicationInstance) {
  let logger = applicationInstance.lookup('logger:main');

  logger.log('Hello from the instance initializer!');
}

export default {
  name: 'logger',
  initialize: initialize
};
```

### Getting an Application Instance from a Factory Instance

[`Ember.getOwner`](https://emberjs.com/api/ember/release/classes/@ember%2Fapplication/methods/getOwner?anchor=getOwner) will retrieve the application instance that "owns" an
object. This means that framework objects like components, helpers, and routes
can use [`Ember.getOwner`](https://emberjs.com/api/ember/release/classes/@ember%2Fapplication/methods/getOwner?anchor=getOwner) to perform lookups through their application
instance at runtime.

For example, this component plays songs with different audio services based
on a song's `audioType`.

```app/components/play-audio.js
import Component from '@ember/component';
import { computed } from '@ember/object';
import { getOwner } from '@ember/application';

// Usage:
//
// {{play-audio song=song}}
//
export default Component.extend({
  audioService: computed('song.audioType', function() {
    let applicationInstance = getOwner(this);
    let audioType = this.get('song.audioType');
    return applicationInstance.lookup(`service:audio-${audioType}`);
  }),

  click() {
    let player = this.get('audioService');
    player.play(this.get('song.file'));
  }
});
```
