组件与周围环境隔离，因此组件所需的任何数据都必须传入。

例如，假设您有一个 `blog-post` 组件用于显示博客文章:

```app/templates/components/blog-post.hbs
<article class="blog-post">
  <h1>{{title}}</h1>
  <p>{{body}}</p>
</article>
```

现在想象我们有以下模板和路由：

```app/routes/index.js
import Route from '@ember/routing/route';

export default Route.extend({
  model() {
    return this.get('store').findAll('post');
  }
});
```

如果我们试图这样使用组件：

```app/templates/index.hbs
{{#each model as |post|}}
  {{blog-post}}
{{/each}}
```

将呈现以下HTML：

```html
<article class="blog-post">
  <h1></h1>
  <p></p>
</article>
```

为了使组件可用，您必须像这样传递它：

```app/templates/index.hbs
{{#each model as |post|}}
  {{blog-post title=post.title body=post.body}}
{{/each}}
```

重要的是要注意这些属性保持同步（技术上称为“绑定”）。 也就是说，如果 `componentProperty`的值在组件中改变, `outerProperty` 将被更新以反映那个变化. 反过来也是如此。

## Positional Params(位置参数)

除了按名称传递参数之外，还可以按位置传递参数。换句话说，您可以像这样调用上述组件示例：

```app/templates/index.hbs
{{#each model as |post|}}
  {{blog-post post.title post.body}}
{{/each}}
```

要将组件设置为以这种方式接收参数， 您需要在组件中设置 [`positionalParams`](https://www.emberjs.com/api/ember/release/classes/Component/properties/positionalParams?anchor=positionalParams) 属性.

```app/components/blog-post.js
import Component from '@ember/component';

export default Component.extend({}).reopenClass({
  positionalParams: ['title', 'body']
});
```

然后你可以像使用 `{{blog-post title=post.title body=post.body}}`时一样使用组件中的属性.

请注意， `positionalParams` 属性作为静态变量通过`reopenClass`添加到类中。 位置参数总是在组件类上声明，并且在应用程序运行时不能更改。

或者，您可以通过设置 `positionalParams` 为字符串来接受任意数量的参数， 例如 `positionalParams: 'params'`. 这将允许你接收这些参数作为一个数组，像下面一样:

```app/components/blog-post.js
import Component from '@ember/component';
import { computed } from '@ember/object';

export default Component.extend({
  title: computed('params.[]', function(){
    return this.get('params')[0];
  }),
  body: computed('params.[]', function(){
    return this.get('params')[1];
  })
}).reopenClass({
  positionalParams: 'params'
});
```
