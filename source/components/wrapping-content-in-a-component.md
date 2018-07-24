有时，您可能需要定义一个包装其他模板提供的内容的组件。

例如，假设我们正在构建一个`blog-post`组件，可以在我们的应用程序中使用它来显示博客帖子：

```app/templates/components/blog-post.hbs
<h1>{{title}}</h1>
<div class="body">{{body}}</div>
```

现在，我们可以使用 `{{blog-post}}` 组件并在另一个模板中传递它的属性：

```handlebars
{{blog-post title=title body=body}}
```

有关更多信息，请参阅[将属性传递给组件](../passing-properties-to-a-component/)。

在这种情况下，我们想要显示的内容来自模型。但是，如果我们希望使用我们的组件的开发人员能够提供自定义HTML内容呢？

除了您迄今为止学到的简单形式，组件还支持以 **块形式** 使用。在块形式中，组件可以传递Handlebars模板，该模板在`{{yield}}`表达式出现的任何位置呈现在组件的模板内。

要使用块形式，请在组件名称的开头添加一个`#`字符，然后确保添加结束标记。

有关更多信息，请参阅有关[块表达式](http://handlebarsjs.com/#block-expressions)的Handlebars文档。

在这种情况下，我们可以在`{{blog-post}}`组件中使用 **块形式** ，并告诉Ember块内容应该使用`{{yield}}`助手呈现在哪里。要更新上面的示例，我们将首先更改组件的模板：

```app/templates/components/blog-post.hbs
<h1>{{title}}</h1>
<div class="body">{{yield}}</div>
```

你可以看到我们已经用`{{yield}}`取代了 `{{body}}`.这告诉Ember使用该组件时将提供此内容。

接下来，我们将使用块形式更新组件模板：

```app/templates/index.hbs
{{#blog-post title=title}}
  <p class="author">by {{author}}</p>
  {{body}}
{{/blog-post}}
```

重要的是要注意，组件块内的模板范围与外部相同。如果组件外部的模板中有属性，则组件块中也可以使用该属性。

## Sharing Component Data with its Wrapped Content（使用包装内容共享组件数据）

还有一种方法可以将博客文章组件中的数据与其包装的内容进行共享。
在我们的博客文章组件中，我们希望为用户提供一种方式来配置他们想要写入帖子的方式。
我们将为他们提供指定的选项`markdown-style`或者`html-style`。

```app/templates/index.hbs
{{#blog-post editStyle="markdown-style"}}
  <p class="author">by {{author}}</p>
  {{body}}
{{/blog-post}}
```

支持不同的编辑样式将需要不同的主体组件来提供特殊的验证和高亮显示。
要根据编辑样式加载不同的主体组件，可以使用[`component helper`](https://www.emberjs.com/api/ember/release/classes/Ember.Templates.helpers/methods/component?anchor=component)和生成组件[`hash helper`](https://www.emberjs.com/api/ember/release/classes/Ember.Templates.helpers/methods/hash?anchor=hash)。
在这里，使用嵌套的帮助程序将适当的组件分配给哈希并将其输出到模板。
请注意，`editStyle`它被用作`component helper`的参数。

```app/templates/components/blog-post.hbs
<h2>{{title}}</h2>
<div class="body">{{yield (hash body=(component editStyle))}}</div>
```

一旦设置，通过引用`post`变量，`wrapped content`可以访问数据。
现在叫做`markdown-style`的组件将被渲染到 `{{post.body}}` 中。

```app/templates/index.hbs
{{#blog-post editStyle="markdown-style" postData=myText as |post|}}
  <p class="author">by {{author}}</p>
  {{post.body}}
{{/blog-post}}
```

最后，我们需要分享`myText`,这样他才能在`body`中展示它。要将博客文本传递给body组件，我们将向组件助手添加一个`postData`参数。

```app/templates/components/blog-post.hbs
<h2>{{title}}</h2>
<div class="body">
  {{yield (hash
    body=(component editStyle postData=postData)
  )}}
</div>
```

此时，我们的内容块已经通过`blog-post`组件获得他渲染所需要的所有内容。

此外，由于在呈现块内容之前不会实例化组件，因此我们可以在块中添加参数。在这种情况下，我们将添加一个文本样式选项，它将指示我们在帖子中想要的正文文本的样式。在 `{{post.body}}` 实例化时，它将包含由包装组件提供的`editStyle`和`postData`以及模板中声明的`bodyStyle`。

```app/templates/index.hbs
{{#blog-post editStyle="markdown-style" postData=myText as |post|}}
  <p class="author">by {{author}}</p>
  {{post.body bodyStyle="compact-style"}}
{{/blog-post}}
```

以这种方式构建的组件通常被称为“上下文组件”，允许内部组件在外部组件的上下文中包装而不破坏封装。
