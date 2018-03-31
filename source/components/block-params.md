组件不仅可以有传入的属性([Passing Properties to a Component](../passing-properties-to-a-component/)),
而且可以返回在块表达式中使用的输出。
### Return values from a component with `yield`(返回组件的值 yield)

```app/templates/index.hbs
{{blog-post post=model}}
```

```app/templates/components/blog-post.hbs
{{yield post.title post.body post.author}}
```

这里整个博客文章模型作为单个组件属性传递给组件。另一方面，组件正在使用`yield`返回值.在这种情况下，所传递的值将从传入的帖子中提取，而且组件已经获取的任何内容都可以被获取，例如内部属性或服务中的某些内容。

### Consuming yielded values with block params(使用块参数产生值)

块表达式可以使用块参数将名称绑定到块中使用的任何已产生的值。(原文：The block expression can then use block params to bind names to any yielded values for use in the block.)
这允许在使用组件时进行模板定制，其中标记由消费模板提供，但保留组件中实现的任何事件处理行为，例如 `click()` 处理程序。

```app/templates/index.hbs
{{#blog-post post=model as |title body author|}}
  <h2>{{title}}</h2>
  <p class="author">by {{author}}</p>
  <p class="post-body">{{body}}</p>
{{/blog-post}}
```

这些名称按照它们传递给 `yield` 组件模板的顺序进行绑定。

### Supporting both block and non-block component usage in one template(在一个模板中支持块和非块组件的使用)

可以使用 `has-block` 助手从单个组件模板支持组件的块和非块使用。

```app/templates/components/blog-post.hbs
{{#if (has-block)}}
  {{yield post.title post.body post.author}}  
{{else}}
  <h1>{{post.title}}</h1>
  <p class="author">Authored by {{post.author}}</p>
  <p>{{post.body}}</p>
{{/if}}
```

这在使用非块表单中的组件时提供默认模板，但在使用块表达式时提供用于块参数的已赋值。
