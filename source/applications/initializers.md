初始化器提供了一个在应用程序引导时配置应用程序的机会.

有两种类型的初始化程序：应用程序初始化程序和应用程序实例初始化程序。

应用程序初始化程序在应用程序启动时运行，并提供在应用程序中配置 [依赖注入](../dependency-injection) 的主要方法.

应用程序实例初始化程序在加载应用程序实例时运行。它们提供了一种配置应用程序初始状态的方法，以及设置应用程序实例本地的依赖注入（例如A / B测试配置）。

在初始化程序中执行的操作应尽可能轻量化，以尽量减少加载应用程序的延迟。尽管在应用程序初始化中存在异步操作的高级技术，(例如 `deferReadiness` 和 `advanceReadiness`), 但通常应避免使用这些技术。任何异步加载条件（例如用户授权）在您的应用程序路由的钩子中几乎总是处理得更好，这允许在等待条件解决的同时进行DOM交互。

## Application Initializers(应用初始化程序)

应用程序初始化器可以使用 Ember CLI的 `initializer` 生成器创建：

```bash
ember generate initializer shopping-cart
```

让我们定制 `shopping-cart` 初始化程序，将 `cart` 属性注入到应用程序中的所有路由中：

```app/initializers/shopping-cart.js
export function initialize(application) {
  application.inject('route', 'cart', 'service:shopping-cart');
};

export default {
  initialize
};
```

## Application Instance Initializers(应用程序实例初始化程序)

应用程序实例初始化器可以使用 Ember CLI的 `instance-initializer` 生成器创建：

```bash
ember generate instance-initializer logger
```

让我们添加一些简单的日志记录来指示实例已启动：

```app/instance-initializers/logger.js
export function initialize(applicationInstance) {
  let logger = applicationInstance.lookup('logger:main');
  logger.log('Hello from the instance initializer!');
}

export default {
  initialize
};
```

## Specifying Initializer Order(指定初始化程序顺序)

如果您想控制初始化程序的运行顺序，可以使用 `before` 或 `after` 选项：

```app/initializers/config-reader.js
export function initialize(application) {
  // ... your code ...
};

export default {
  before: 'websocket-init',
  initialize
};
```

```app/initializers/websocket-init.js
export function initialize(application) {
  // ... your code ...
};

export default {
  after: 'config-reader',
  initialize
};
```

```app/initializers/asset-init.js
export function initialize(application) {
  // ... your code ...
};

export default {
  after: ['config-reader', 'websocket-init'],
  initialize
};
```

请注意，排序仅适用于相同类型的初始化程序（即应用程序或应用程序实例）。应用程序初始化器总是在应用程序实例初始化器之前运行

## Customizing Initializer Names(自定义初始化程序名称)

默认情况下，初始化程序名称是从它们的模块名称派生的. 这个初始化器将被命名为 `logger`:

```app/instance-initializers/logger.js
export function initialize(applicationInstance) {
  let logger = applicationInstance.lookup('logger:main');
  logger.log('Hello from the instance initializer!');
}

export default { initialize };
```

如果您想更改名称，只需重命名该文件即可，但如果需要，还可以明确指定名称：

```app/instance-initializers/logger.js
export function initialize(applicationInstance) {
  let logger = applicationInstance.lookup('logger:main');
  logger.log('Hello from the instance initializer!');
}

export default {
  name: 'my-logger',
  initialize
};
```

此初始化程序现在将具有该名称 `my-logger`.
