欢迎来到Ember.js指南！本文档将带您从初学者到Ember专家。

## 什么是 Ember?

Ember是一个JavaScript前端框架，旨在帮助您构建具有丰富和复杂用户交互的网站。
它通过为开发人员提供许多对于管理现代Web应用程序复杂性至关重要的功能以及可快速迭代的集成开发工具包来实现这一点。

这些指南中将介绍的一些功能包括：

* [Ember CLI](./configuring-ember/configuring-ember-cli/) - 一个强大的开发工具包，用于创建，开发和构建Ember应用程序。 当您在整个指南中看到一条 `$ ember <command>` 形式的命令时, 那就是Ember CLI!
* [Routing](./routing) - Ember应用程序的核心部分。使开发人员能够从URL驱动应用程序状态。
* [Templating engine](./templates/handlebars-basics/) -使用Handlebars语法来编写应用程序的模板
* [Data layer](./models/) - Ember Data提供了一种与外部API进行通信并管理应用程序状态的一致方式
* [Ember Inspector](./ember-inspector/) -  一个浏览器扩展或书签，用于现场检查您的应用程序。这对于在发现Ember应用程序很有用，尝试安装它并打开 [NASA website](https://www.nasa.gov/)!

## 组织

在每个指南页面的左侧是一个目录表，按照可以展开的章节进行展示，以显示他们所涵盖的主题。每节中的章节和主题都是从基本到高级的概念。

本指南旨在包含如何构建Ember应用程序的实际解释，重点介绍Ember.js最广泛使用的功能。有关每个Ember功能和API的全面文档，请参阅[Ember.js API 文档](http://emberjs.com/api/).

本指南首先解释如何开始使用Ember，然后介绍如何构建您的第一个Ember应用程序。
如果你刚开始使用Ember,我们建议您首先遵循本指南的前两部分。

## 假设

虽然我们尽可能使指南尽可能地适合初学者，但我们必须建立一个基准，以便指南能够专注于Ember.js功能。只要引入一个概念，我们就会尝试链接到适当的文档。

为了充分利用指南，您应该具备以下知识：

* **HTML, CSS, JavaScript** - 网页的构建块。您可以在 [Mozilla开发者网络上][mdn]找到每种技术的文档。.
* **Promises** - 原生的方式来处理JavaScript代码中的异步。请参阅相关的 [Mozilla开发者网络][promises] 部分。
* **ES2015 modules** - 如果您熟悉[JavaScript模块][js-modules]，您将更好地理解[Ember CLI][ember-cli]的项目结构和导入路径.
* **ES2015 syntax** -  Ember CLI默认带有Babel.js，因此您可以利用更新的语言功能，如箭头函数，模板字符串，解构等等。您可以检查[Babel.js 文档][babeljs] 或 在线阅读 [理解 ECMAScript 6][es6]。

## 关于移动性能的说明

Ember会帮你编写快速的应用程序，但它不能阻止你编写一个缓慢的应用程序。在移动设备上尤其如此。为了提供出色的体验，重要的是要及早并经常地使用各种设备来衡量性能。

确保你在真实设备上测试性能。在台式计算机上模拟移动环境可以为您的真实世界的性能提供乐观的最佳表现。您测试的操作系统和硬件配置越多，您就越有信心。

由于其有限的网络连接和CPU功能，移动设备上的卓越性能很少能够免费获得。您应该从一开始就将性能测试集成到您的开发工作流程中。这将帮助您避免构建昂贵的体系结构错误，如果您在应用几乎完成时才注意到这些错误，那么这些错误将更难修复。

简而言之：

1. 始终在真实，有代表性的移动设备上进行测试
2. 从一开始就衡量性能，并随着应用程序的开发继续测试。

这些提示将帮助您及早发现问题，以便系统地解决问题，而不是在最后一刻的争夺中解决问题。

## 报告问题

错别字，遗漏词和错误代码示例均被视为文档错误。如果您发现其中一个，或者想改善现有指南，
我们很乐意帮助您解决问题！

用指南报告问题的一些更常见的方法是：

* 使用每个指南页面右上角的铅笔图标
* 打开问题或向 [GitHub存储库][gh-guides]提出请求

点击铅笔图标会将您带到GitHub的该指南的编辑器中，以便您可以使用Markdown标记语言立即进行编辑。
这是纠正代码示例中的拼写错误，遗漏词或错误的最快方法。

如果您希望做出更重要的贡献，请务必查看我们的[问题跟踪器][gh-guides-issues] ，
看看您的问题是否已经得到解决。如果您没有找到有效的问题，请打开一个新的问题。

如果您对造型或参与过程有任何疑问，可以查看我们的[参与指南][gh-guides-contributing]。如果您的问题仍然存在，`#-team-learning` 请联系 [Slack团队][slackin].

祝你好运！

[ember-cli]: https://ember-cli.com/

[mdn]: https://developer.mozilla.org/en-US/docs/Web
[promises]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[js-modules]: http://jsmodules.io/
[babeljs]: https://babeljs.io/docs/learn-es2015/
[es6]: https://leanpub.com/understandinges6/read

[gh-guides]: https://github.com/emberjs/guides/
[gh-guides-issues]: https://github.com/emberjs/guides/issues
[gh-guides-contributing]: https://github.com/emberjs/guides/blob/master/CONTRIBUTING.md

[slackin]: https://ember-community-slackin.herokuapp.com/
