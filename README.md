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

## Project layout

指南内容采用Markdown文件的形式（就像大多数自述文件一样）。
指南内容在 `source` 文件夹中。左侧的导航栏是通过
`data/pages.yml`中产生的. `lib` 包含中间件插件,`spec` 包含这些插件的测试文件。

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

## 在本地通过Ruby和中间件运行

The Docker method described above is recommended over installing dependencies
separately. However, if necessary, these are the manual steps. The Guides are built
with Middleman, which runs on Ruby 1.9.3 or newer.

Mac users should install Ruby using rbenv to avoid changing their OS dependencies:

```
brew install rbenv
```

Follow the [rbenv installation instructions](https://github.com/rbenv/rbenv) to install the Ruby version specified [here](.ruby-version), then go through the init steps, set a global version, and restart the terminal. If `gem env home` shows rbenv in the path, your installation was successful. You should not have to sudo install any gems.

Once you have installed Ruby, you will need bundler and Middleman:

```
gem install bundler middleman
```

During build, Middleman will require Aspell to look for misspellings. On Macs, it can be installed via Homebrew:

``` sh
brew install aspell --with-lang-en
```

On Windows, you can download an [installer](http://aspell.net/win32/), but unfortunately it is unmaintained. On Linux, you can install with your distribution's package manager. On all platforms, you can also [build the most recent version from source](http://aspell.net/man-html/Installing.html).

Some Mac users may also need to install openSSL, which will be indicated in an error during the bundle command. See [Troubleshooting.md](TROUBLESHOOTING.md).

To get started:

#### Local Dev
``` sh
git clone git://github.com/emberjs/guides.git
cd guides
bundle
bundle exec middleman
```

#### Viewing

Then visit [http://localhost:4567/](http://localhost:4567/).

If you run into problems, check [Troubleshooting.md](TROUBLESHOOTING.md).

### Spellchecking

If you have a false hit during spellchecking, you can add the word to `/data/spelling-exceptions.txt`.
Words are line separated and case insensitive.

# Maintainers

See [MAINTAINERS.md](MAINTAINERS.md).

# Releasing

See https://github.com/emberjs/guides.emberjs.com.
