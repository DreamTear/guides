要定义组件，请运行:

```shell
ember generate component my-component-name
```

Ember组件用于将标记和样式封装为可重用内容。组件由两部分组成：一个用于定义行为的JavaScript组件文件，以及为组件的UI定义标记的随附Handlebars模板。

组件的名称中必须至少有一个破折号。 因此 `blog-post` 是一个可以接受的名字， `audio-player-controls`也是, 但是 `post` 不是. 这可以防止与当前或未来的HTML元素名称发生冲突， 将Ember组件与W3C [Custom
Elements](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/custom/index.html)
规范对齐， 并确保Ember自动检测组件。

示例组件模板可能如下所示：

```app/templates/components/blog-post.hbs
<article class="blog-post">
  <h1>{{title}}</h1>
  <p>{{yield}}</p>
  <p>Edit title: {{input type="text" value=title}}</p>
</article>
```

鉴于上述模板，您现在可以使用该 `{{blog-post}}` 组件：

```app/templates/index.hbs
{{#each model as |post|}}
  {{#blog-post title=post.title}}
    {{post.body}}
  {{/blog-post}}
{{/each}}
```

它的模型 `model`在路由处理程序的钩子中填充：

```app/routes/index.js
import Route from '@ember/routing/route';

export default Route.extend({
  model() {
    return this.get('store').findAll('post');
  }
});
```

每个组件返回一个元素（原文：Each component is backed by an element under the hood. ）默认情况下，Ember将使用一个 `<div>` 元素来包含组件的模板。
要了解如何更改Ember用于组件的元素， 请参阅
[自定义组件的元素](../customizing-a-components-element).


## Defining a Component Subclass(定义组件子类)

很多时候， 你的组件会封装你发现自己反复使用的特定的Handlebars模板片段。在这些情况下，你根本不需要编写任何JavaScript。 如上所述定义Handlebars模板并使用创建的组件。

如果您需要自定义组件的行为，你需要定义 [`Component`](https://www.emberjs.com/api/ember/release/classes/Component)的子类. 例如，如果您想要更改组件的元素，响应组件的模板中的操作，或者使用JavaScript手动更改组件的元素，则需要自定义子类。

Ember通过文件名知道哪个类控制哪个组件. 例如，如果您有一个名为 `blog-post`的组件, 你将创建一个 `app/components/blog-post.js`文件. 如果你的组件名字叫
`audio-player-controls`, 对应的文件名应该叫
`app/components/audio-player-controls.js`.

## Dynamically rendering a component(动态渲染组件)

[`{{component}}`](https://www.emberjs.com/api/ember/release/classes/Ember.Templates.helpers/methods/component?anchor=component) 助手可以用来推迟组件的运行时间.  `{{my-component}}` 语法总是呈现相同的部件，
当使用 `{{component}}` 助手，允许动态的选择一个组件去渲染. 这在您想根据数据与不同的外部库进行交互的情况下很有用。使用 `{{component}}` 助手可以让你保持不同的逻辑分开。

帮助器的第一个参数是要呈现的组件的名称的字符串. 因此 `{{component 'blog-post'}}` 和使用 `{{blog-post}}`一样.

 [`{{component}}`](https://www.emberjs.com/api/ember/release/classes/Ember.Templates.helpers/methods/component?anchor=component) 真正的价值来自于能够动态地选择将要渲染的组件。以下是使用助手作为选择不同组件以显示不同类型帖子的手段的示例：

```app/templates/components/foo-component.hbs
<h3>Hello from foo!</h3>
<p>{{post.body}}</p>
```

```app/templates/components/bar-component.hbs
<h3>Hello from bar!</h3>
<div>{{post.author}}</div>
```

```app/routes/index.js
import Route from '@ember/routing/route';

export default Route.extend({
  model() {
    return this.get('store').findAll('post');
  }
});
```

```app/templates/index.hbs
{{#each model as |post|}}
  {{!-- either foo-component or bar-component --}}
  {{component post.componentName post=post}}
{{/each}}
```

当传递的参数 `{{component}}` 等于 `null` 或者 `undefined`,
帮助器不渲染任何内容。当参数改变时，当前渲染的组件被销毁并且新组件被创建并被引入。

根据数据选择不同的组件进行渲染，可以让您针对每种情况制定不同的模板和行为. `{{component}}` 助手是提高代码的模块化的强大工具。
