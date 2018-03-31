默认情况下，每个组件都将返回一个 `<div>` 元素. 如果您要在开发人员工具中查看渲染组件，则会看到如下所示的DOM表示形式：

```html
<div id="ember180" class="ember-view">
  <h1>My Component</h1>
</div>
```

您可以通过`Component` 在您的JavaScript中创建一个子类来定制Ember为您的组件生成什么类型​​的元素，包括它的属性和类名称。

### Customizing the Element(自定义元素)

要使用除 `div`以外的标签, 创建 `Component` 的子类，并为其分配`tagName` 属性. 此属性可以是任何有效的HTML5标记名称的字符串。

```app/components/navigation-bar.js
import Component from '@ember/component';

export default Component.extend({
  tagName: 'nav'
});
```

```app/templates/components/navigation-bar.hbs
<ul>
  <li>{{#link-to "home"}}Home{{/link-to}}</li>
  <li>{{#link-to "about"}}About{{/link-to}}</li>
</ul>
```

### Customizing the Element's Class(自定义元素的类)

您可以在调用时指定组件元素的类，方法与常规HTML元素相同：

```hbs
{{navigation-bar class="primary"}}
```

您还可以通过将其 `classNames` 属性设置为字符串数组来指定将哪些类名应用于组件的元素：

```app/components/navigation-bar.js
import Component from '@ember/component';

export default Component.extend({
  classNames: ['primary']
});
```

如果你想让类名由组件的属性决定，你可以使用类名绑定。 如果绑定到布尔属性，则将根据值添加或删除类名称：

```app/components/todo-item.js
import Component from '@ember/component';

export default Component.extend({
  classNameBindings: ['isUrgent'],
  isUrgent: true
});
```

该组件将呈现以下内容：

```html
<div class="ember-view is-urgent"></div>
```

如果 `isUrgent` 更改为 `false`, 则 `is-urgent` 类名将被删除。

默认情况下，布尔属性的名称是dasherized。您可以通过用冒号分隔它来自定义类名称：

```app/components/todo-item.js
import Component from '@ember/component';

export default Component.extend({
  classNameBindings: ['isUrgent:urgent'],
  isUrgent: true
});
```

这将呈现这个HTML：

```html
<div class="ember-view urgent">
```

除了值的自定义类名称之外 `true`, 还可以指定在值为时使用的类名称 `false`:

```app/components/todo-item.js
import Component from '@ember/component';

export default Component.extend({
  classNameBindings: ['isEnabled:enabled:disabled'],
  isEnabled: false
});
```

这将呈现这个HTML：

```html
<div class="ember-view disabled">
```

您也可以指定一个只能在属性为`false`时才添加的类，通过声明 `classNameBindings` 像这样:

```app/components/todo-item.js
import Component from '@ember/component';

export default Component.extend({
  classNameBindings: ['isEnabled::disabled'],
  isEnabled: false
});
```

这将呈现这个HTML：

```html
<div class="ember-view disabled">
```

如果该 `isEnabled` 属性设置为 `true`, 则不添加类名称：

```html
<div class="ember-view">
```

如果绑定属性的值是一个字符串，则该值将作为类名添加，而不进行修改：

```app/components/todo-item.js
import Component from '@ember/component';

export default Component.extend({
  classNameBindings: ['priority'],
  priority: 'highestPriority'
});
```

这将呈现这个HTML：

```html
<div class="ember-view highestPriority">
```

### Customizing Attributes(定制属性)

您可以使用`attributeBindings`属性将属性绑定到表示组件的DOM元素:

```app/components/link-item.js
import Component from '@ember/component';

export default Component.extend({
  tagName: 'a',
  attributeBindings: ['href'],

  href: 'http://emberjs.com'
});
```

您还可以将这些属性绑定到不同名称的属性：

```app/components/link-item.js
import Component from '@ember/component';

export default Component.extend({
  tagName: 'a',
  attributeBindings: ['customHref:href'],

  customHref: 'http://emberjs.com'
});
```

如果该属性为空，则不会渲染：

```app/components/link-item.js
import Component from '@ember/component';

export default Component.extend({
  tagName: 'span',
  attributeBindings: ['title'],

  title: null,
});
```
当没有标题传递给组件时，这会呈现这个HTML：

```html
<span class="ember-view">
```

...以及将“Ember JS”的标题传递给组件时的HTML：

```html
<span class="ember-view" title="Ember JS">
```
