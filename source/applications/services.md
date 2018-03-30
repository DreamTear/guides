 [`Service`](https://www.emberjs.com/api/ember/release/modules/@ember%2Fservice)是一个Ember对象，它在应用程序的整个过程中一直存在，并且可以在应用程序的不同部分提供。

服务对于需要共享状态或持久连接的功能很有用。服务的示例使用可能包括：

* 用户/会话认证。
* 地理位置。
* WebSockets.
* 服务器发送的事件或通知。
* 服务器支持的API调用可能不适合Ember数据。
* 第三方 API.
* 日志记录。

### Defining Services(定义服务)

服务可以使用 Ember CLI的 `service` 生成器生成。
例如，以下命令将创建该 `ShoppingCart` 服务：

```bash
ember generate service shopping-cart
```

服务必须扩展 [`Service`](https://www.emberjs.com/api/ember/release/modules/@ember%2Fservice) 基类：

```app/services/shopping-cart.js
import Service from '@ember/service';

export default Service.extend({
});
```

像任何Ember对象一样，一个服务被初始化并且可以拥有它自己的属性和方法。下面例子中，购物车服务管理代表购物车中当前物品的物品数组。

```app/services/shopping-cart.js
import Service from '@ember/service';

export default Service.extend({
  items: null,

  init() {
    this._super(...arguments);
    this.set('items', []);
  },

  add(item) {
    this.get('items').pushObject(item);
  },

  remove(item) {
    this.get('items').removeObject(item);
  },

  empty() {
    this.get('items').clear();
  }
});
```

### Accessing Services(访问服务)

要访问服务，可以使用来自`@ember/service`模块中的`inject`方法将服务注入任何container-resolved对象，例如：组件或其它服务。
有两种方法可以使用这个功能。您可以不带任何参数地调用它，也可以传递它使用服务已经注册的名称。当没有参数传递时，服务将根据变量键的名称加载。您可以像下面代码一样在没有任何参数的情况下加载购物车服务。

```app/components/cart-contents.js
import Component from '@ember/component';
import { inject as service } from '@ember/service';

export default Component.extend({
  //will load the service in file /app/services/shopping-cart.js
  shoppingCart: service()
});
```

注入服务的另一种方法是提供服务的名称作为参数。

```app/components/cart-contents.js
import Component from '@ember/component';
import { inject as service } from '@ember/service';

export default Component.extend({
  //will load the service in file /app/services/shopping-cart.js
  cart: service('shopping-cart')
});
```

这会将购物车服务注入到组件中，并将其作为 `cart` 属性提供。

有时服务可能存在也可能不存在，例如初始化程序有条件的注册服务时。
由于正常注入会在服务不存在时抛出错误，因此您必须使用Ember  [`getOwner`](https://emberjs.com/api/ember/release/classes/@ember%2Fapplication/methods/getOwner?anchor=getOwner) 来查找服务。

```app/components/cart-contents.js
import Component from '@ember/component';
import { computed } from '@ember/object';
import { getOwner } from '@ember/application';

export default Component.extend({
  //will load the service in file /app/services/shopping-cart.js
  cart: computed(function() {
    return getOwner(this).lookup('service:shopping-cart');
  })
});
```

注入的属性被延迟加载; 这意味着该服务将不会被实例化，直到该属性被明确调用。
因此，您需要使用该 `get` 功能访问组件中的服务，否则您可能会遇到未定义的问题。

加载后，服务将一直存在，直到应用程序退出。

下面我们向 `cart-contents` 组件添加一个删除操作。请注意，下面我们通过调用`this.get`来获得`cart`服务。

```app/components/cart-contents.js
import Component from '@ember/component';
import { inject as service } from '@ember/service';

export default Component.extend({
  cart: service('shopping-cart'),

  actions: {
    remove(item) {
      this.get('cart').remove(item);
    }
  }
});
```
一旦注入组件，服务也可以在模板中使用。
注意 `cart`将使用来自购物车中的数据。

```app/templates/components/cart-contents.hbs
<ul>
  {{#each cart.items as |item|}}
    <li>
      {{item.name}}
      <button {{action "remove" item}}>Remove</button>
    </li>
  {{/each}}
</ul>
```
