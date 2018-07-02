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

## Sharing Component Data with its Wrapped Content

There is also a way to share data within your blog post component with the content it is wrapping.
In our blog post component we want to provide a way for the user to configure what type of style they want to write their post in.
We will give them the option to specify either `markdown-style` or `html-style`.

```app/templates/index.hbs
{{#blog-post editStyle="markdown-style"}}
  <p class="author">by {{author}}</p>
  {{body}}
{{/blog-post}}
```

Supporting different editing styles will require different body components to provide special validation and highlighting.
To load a different body component based on editing style,
you can yield the component using the [`component helper`](https://www.emberjs.com/api/ember/release/classes/Ember.Templates.helpers/methods/component?anchor=component) and [`hash helper`](https://www.emberjs.com/api/ember/release/classes/Ember.Templates.helpers/methods/hash?anchor=hash).
Here, the appropriate component is assigned to a hash using nested helpers and yielded to the template.
Notice `editStyle` being used as an argument to the component helper.

```app/templates/components/blog-post.hbs
<h2>{{title}}</h2>
<div class="body">{{yield (hash body=(component editStyle))}}</div>
```

Once yielded, the data can be accessed by the wrapped content by referencing the `post` variable.
Now a component called `markdown-style` will be rendered in `{{post.body}}`.

```app/templates/index.hbs
{{#blog-post editStyle="markdown-style" postData=myText as |post|}}
  <p class="author">by {{author}}</p>
  {{post.body}}
{{/blog-post}}
```

Finally, we need to share `myText` with the body in order to have it display.
To pass the blog text to the body component, we'll add a `postData` argument to the component helper.

```app/templates/components/blog-post.hbs
<h2>{{title}}</h2>
<div class="body">
  {{yield (hash
    body=(component editStyle postData=postData)
  )}}
</div>
```

At this point, our block content has access to everything it needs to render,
via the wrapping `blog-post` component's template helpers.

Additionally, since the component isn't instantiated until the block content is rendered,
we can add arguments within the block.
In this case we'll add a text style option which will dictate the style of the body text we want in our post.
When `{{post.body}}` is instantiated, it will have both the `editStyle` and `postData` given by its wrapping component,
as well as the `bodyStyle` declared in the template.

```app/templates/index.hbs
{{#blog-post editStyle="markdown-style" postData=myText as |post|}}
  <p class="author">by {{author}}</p>
  {{post.body bodyStyle="compact-style"}}
{{/blog-post}}
```

Components built this way are commonly referred to as "Contextual Components",
allowing inner components to be wrapped within the context of outer components without breaking encapsulation.
