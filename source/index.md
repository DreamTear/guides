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

While we try to make the Guides as beginner-friendly as we can,
we must establish a baseline so that the guides can keep focused on Ember.js functionality.
We will try to link to appropriate documentation whenever a concept is introduced.

To make the most out of the guides, you should have a working knowledge of:

* **HTML, CSS, JavaScript** - the building blocks of web pages. You can find documentation of each of these technologies at the [Mozilla Developer Network][mdn].
* **Promises** - the native way to deal with asynchrony in your JavaScript code. See the relevant [Mozilla Developer Network][promises] section.
* **ES2015 modules** - you will better understand [Ember CLI's][ember-cli] project structure and import paths if you are comfortable with [JavaScript Modules][js-modules].
* **ES2015 syntax** - Ember CLI comes with Babel.js by default so you can
take advantage of newer language features such as arrow functions, template
strings, destructuring, and more. You can check the
[Babel.js documentation][babeljs] or read [Understanding ECMAScript 6][es6]
online.

## A Note on Mobile Performance

Ember will do a lot to help you write fast apps, but it can't prevent you from
writing a slow one. This is especially true on mobile devices. To deliver a great
experience, it's important to measure performance early and often, and with a diverse
set of devices.

Make sure you are testing performance on real devices. Simulated mobile
environments on a desktop computer give an optimistic-at-best representation of
what your real world performance will be like. The more operating systems and
hardware configurations you test, the more confident you can be.

Due to their limited network connectivity and CPU power, great performance on
mobile devices rarely comes for free. You should integrate performance testing
into your development workflow from the beginning. This will help you avoid
making costly architectural mistakes that are much harder to fix if you only
notice them once your app is nearly complete.

In short:

1. Always test on real, representative mobile devices.
2. Measure performance from the beginning, and keep testing as your app
   develops.

These tips will help you identify problems early so they can be addressed systematically, rather than
in a last-minute scramble.

## Reporting a problem

Typos, missing words, and code samples with errors are all considered
documentation bugs. If you spot one of them, or want to otherwise improve
the existing guides, we are happy to help you help us!

Some of the more common ways to report a problem with the guides are:

* Using the pencil icon on the top-right of each guide page
* Opening an issue or pull request to [the GitHub repository][gh-guides]

Clicking the pencil icon will bring you to GitHub's editor for that
guide so you can edit right away, using the Markdown markup language.
This is the fastest way to correct a typo, a missing word, or an error in
a code sample.

If you wish to make a more significant contribution be sure to check our
[issue tracker][gh-guides-issues] to see if your issue is already being
addressed. If you don't find an active issue, open a new one.

If you have any questions about styling or the contributing process, you
can check out our [contributing guide][gh-guides-contributing]. If your
question persists, reach us at `#-team-learning` on the [Slack
group][slackin].

Good luck!

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
