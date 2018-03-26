每个Ember 应用都是继承自[`Application`](https://emberjs.com/api/ember/release/classes/Application)类的子类的实例。
该类用于声明和配置组成应用的许多对象。

随着应用程序的启动，它会创建一个 [`ApplicationInstance`](https://emberjs.com/api/ember/release/classes/ApplicationInstance) 用于管理其有状态方面的内容。此实例充当为您的应用程序实例化的对象的“所有者”。

实质上，  `Application` *定义你的应用程序*
而 `ApplicationInstance` *管理它的状态*.

这种关注的分离不仅可以阐明应用程序的体系结构，还可以提高其效率。当在你的程序由于测试或者服务端渲染需要重启是尤其明显
(例如 通过 [FastBoot](https://github.com/tildeio/ember-cli-fastboot)).
 `Application`可以配置一次，然后在多个状态的  `ApplicationInstance` 实例间共享。
这些实例可以在不再需要时丢弃(例如 当测试运行或FastBoot请求完成时).
