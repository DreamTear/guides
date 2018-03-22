## Ember Guides 源代码

[![Build Status](https://travis-ci.org/emberjs/guides.svg?branch=master)](https://travis-ci.org/emberjs/guides)

这是 [Ember.js Guides](https://guides.emberjs.com)网站的源代码.

在寻找其它部分的存储库? 请查看
[website](https://github.com/emberjs/website),
[ember-api-docs](https://github.com/ember-learn/ember-api-docs),
[super-rentals tutorial](https://github.com/ember-learn/super-rentals),
[statusboard](https://github.com/ember-learn/statusboard),
和 [styleguide](https://github.com/ember-learn/ember-styleguide)

## 特约

欢迎并感谢您的帮助! 请参阅 [CONTRIBUTING.md](CONTRIBUTING.md)了解如何格式化您的工作并提交合并请求的详细说明。

## 项目结构

指南内容采用Markdown文件的形式（就像大多数自述文件一样）。
指南内容在 `source` 文件夹中。左侧的导航栏是通过
`data/pages.yml`中产生的. `lib` 包含Middleman插件,`spec` 包含这些插件的测试文件。

## 本地使用Docker运行（推荐）

这是推荐给新贡献者的方法。虽然指南是用Ruby构建的，但大部分工作都是在Markdown文件中完成的。
你不需要知道Ruby或者安装它的依赖。只需按照以下Docker容器说明进行本地安装和运行即可。

首先，安装 [Docker and Compose](https://store.docker.com/search?offering=community&type=edition) 并保持运行。

接下来，下面的命令将为Guides应用程序安装所有必需的依赖项并启动服务器。这需要一段时间才能运行，可能需要几分钟。依赖关系将安装在Docker容器内，并且不会影响您的正常开发人员环境。

```sh
git clone git://github.com/emberjs/guides.git
cd guides
docker-compose build
docker-compose up
```

你可以查看站点，通过 [http://localhost:4567](http://localhost:4567)

## 在本地通过Ruby和Middleman运行

上面描述的Docker方法推荐单独安装依赖关系。但是，如果有必要，下面是手动步骤。 这些指南是用Middleman构建的，它运行在Ruby 1.9.3或更新的版本上。

Mac用户应该使用rbenv安装Ruby，以避免更改他们的操作系统依赖关系：

```
brew install rbenv
```

按照 [rbenv 安装说明](https://github.com/rbenv/rbenv)安装指定的Ruby版本 [此处](.ruby-version), 然后执行init步骤，设置全局版本，然后重新启动终端。如果 `gem env home` 在路径中显示rbenv，则说明安装成功。你不应该sudo安装任何gems.

一旦你安装了Ruby，你将需要bundler和Middleman：

```
gem install bundler middleman
```

在构建期间，Middleman将要求Aspell寻找拼写错误。在Mac上，它可以通过Homebrew安装：

``` sh
brew install aspell --with-lang-en
```

在Windows上，您可以下载 [安装程序](http://aspell.net/win32/)，但不幸的是它没有维护。在Linux上，您可以使用发行版的软件包管理器进行安装。 在所有平台上，您还可以 [从源代码构建最新版本](http://aspell.net/man-html/Installing.html).

一些Mac用户可能还需要安装openSSL，这在bundle命令中会显示错误。请参阅 [Troubleshooting.md](TROUBLESHOOTING.md).

开始：

#### 本地开发
``` sh
git clone git://github.com/emberjs/guides.git
cd guides
bundle
bundle exec middleman
```

#### 查看

然后访问 [http://localhost:4567/](http://localhost:4567/).

如果遇到问题，请检查 [Troubleshooting.md](TROUBLESHOOTING.md).

### 拼写检查

如果您在拼写检查过程中遇到错误，则可以将错误单词添加到`/data/spelling-exceptions.txt`。
单词是行分隔的，不区分大小写。

# 维护者

请参阅 [MAINTAINERS.md](MAINTAINERS.md).

# 发版信息

请参阅 https://github.com/emberjs/guides.emberjs.com.
