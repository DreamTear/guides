在开发您的Ember应用程序时，您可能会遇到Ember本身无法解决的常见情况，例如身份验证或使用SASS处理样式表。
Ember CLI提供了一种名为 [Ember Addons](#toc_addons) 的通用格式，用于分发可重用的库来解决这些问题。
此外，您可能希望利用不特定于Ember应用的前端依赖项，如CSS框架或JavaScript日期选择器。

## Addons(插件)

Ember插件可以使用 [Ember CLI](http://ember-cli.com/extending/#developing-addons-and-blueprints)
(例如 `ember install ember-cli-sass`)进行安装。
插件可能会通过 `bower.json`自动修改项目文件引入其他依赖项。

你可以在 [Ember Observer](http://emberobserver.com)上找到插件列表。

## Other assets(其他资产)

第三方JavaScript不可用作插件或Bower软件包应放置在 `vendor/` 项目的文件夹中。

你自己的资产 (比如 `robots.txt`, `favicon`, 定制字体,等) 应放置在在 `public/` 文件夹中的项目。

## Compiling Assets(编译资产)

当您使用未包含在插件中的依赖项时，您必须指示Ember CLI将您的资产包含在构建中。
编译使用 `ember-cli-build.js`中的配置来进行。您应该只尝试导入位于 `bower_components` 和 `vendor` 文件夹中的资产。

### Globals provided by JavaScript assets(全球资源由JavaScript资源提供)

某些全局变量 (例如下面例子中使用到的 `moment`) 可以在您的应用程序中使用
，而不需要 `import` 它们。提供资产路径作为第一个也是唯一的参数。

```ember-cli-build.js
app.import('bower_components/moment/moment.js');
```

你需要添加 `"moment"` 到 `.eslintrc.js` 的 `globals` 部分，以防止ESLint出现使用未定义变量的错误。

### AMD JavaScript modules(AMD JavaScript模块)

提供资产路径作为第一个参数，并将模块和导出列表作为第二个参数。

```ember-cli-build.js
app.import('bower_components/ic-ajax/dist/named-amd/main.js', {
  exports: {
    'ic-ajax': [
      'default',
      'defineFixture',
      'lookupFixture',
      'raw',
      'request'
    ]
  }
});
```

你现在可以在你的app中 `import` 他们。 (例如 `import { raw as icAjaxRaw } from 'ic-ajax';`)

### Environment-Specific Assets(环境特定资产)

如果您需要在不同的环境中使用不同的资产，请将对象指定为第一个参数。
对象的 key 应该是环境名称, 而且值应该是环境中使用的资源。

```ember-cli-build.js
app.import({
  development: 'bower_components/ember/ember.js',
  production:  'bower_components/ember/ember.prod.js'
});
```

如果您只需要在一个环境中导入资产，则可以将 `app.import` 放在要给 `if` 语句中。
对于在测试期间需要使用的资源，你可以通过 `{type: 'test'}` 选项来确保它们在测试模式下可用。

```ember-cli-build.js
if (app.env === 'development') {
  // Only import when in development mode
  app.import('vendor/ember-renderspeed/ember-renderspeed.js');
}
if (app.env === 'test') {
  // Only import in test mode and place in test-support.js
  app.import(app.bowerDirectory + '/sinonjs/sinon.js', { type: 'test' });
  app.import(app.bowerDirectory + '/sinon-qunit/lib/sinon-qunit.js', { type: 'test' });
}
```

### CSS

提供资产路径作为第一个参数：

```ember-cli-build.js
app.import('bower_components/foundation/css/foundation.css');
```

以这种方式添加的所有样式将被输出到 `/assets/vendor.css`.

### Other Assets(其他资产)

所有位于 `public/` 目录下的资源将被原样复制到 `dist/`目录下。

例如  `favicon` 路径为 `public/images/favicon.ico` 将被复制到 `dist/images/favicon.ico`.

所有第三方资源，包括手动添加到 `vendor/` 目录下，或者通过像Bower这样的包管理器添加的资源，必须通过 `import()`添加。

没有通过 `import()` 添加的第三方资源,最后编译时将不会被添加在最终版本中。

默认情况下，被 `import`的资源，将被按原来的目录结构复制到 `dist/` 目录下。

```ember-cli-build.js
app.import('bower_components/font-awesome/fonts/fontawesome-webfont.ttf');
```

这个例子会在中创建字体文件 `dist/font-awesome/fonts/fontawesome-webfont.ttf`.

您还可以选择配置 `import()` 将文件放置在不同的路径中。
以下示例将将该文件复制到 `dist/assets/fontawesome-webfont.ttf`.

```ember-cli-build.js
app.import('bower_components/font-awesome/fonts/fontawesome-webfont.ttf', {
  destDir: 'assets'
});
```

如果你需要在别人之前加载某些依赖关系，你可以在 `import()`的第二个参数上设置`prepend`为`true`。
这将在默认行为前添加依赖关系到vendor文件。

```ember-cli-build.js
app.import('bower_components/es5-shim/es5-shim.js', {
  type: 'vendor',
  prepend: true
});
```
