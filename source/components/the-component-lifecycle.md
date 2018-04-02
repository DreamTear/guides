让组件如此有用的部分原因是它们让你完全控制组件的DOM。这允许你直接操作DOM，监听和响应浏览器事件或者在你的Ember应用中使用第三方Javascript库。

随着组件被渲染，重新渲染并最终被移除，Ember提供了允许在组件生命周期的特定时间运行代码的 _lifecycle hooks_ .

为了充分利用组件，了解这些生命周期方法非常重要。

## Order of Lifecycle Hooks Called(调用生命周期钩子的顺序)
下面列出了根据渲染场景执行的组件生命周期 [hooks](../../getting-started/core-concepts/#toc_hooks).

### On Initial Render(在初始渲染阶段)

1. `init`
2. [`didReceiveAttrs`](#toc_formatting-component-attributes-with-code-didreceiveattrs-code)
3. `willRender`
4. [`didInsertElement`](#toc_integrating-with-third-party-libraries-with-code-didinsertelement-code)
5. [`didRender`](#toc_making-updates-to-the-rendered-dom-with-code-didrender-code)

### On Re-Render(在重新渲染阶段)

1. [`didUpdateAttrs`](#toc_resetting-presentation-state-on-attribute-change-with-code-didupdateattrs-code)
2. [`didReceiveAttrs`](#toc_formatting-component-attributes-with-code-didreceiveattrs-code)
3. `willUpdate`
4. `willRender`
5. `didUpdate`
6. [`didRender`](#toc_making-updates-to-the-rendered-dom-with-code-didrender-code)

### On Component Destroy(在组件销毁阶段)

1. [`willDestroyElement`](#toc_detaching-and-tearing-down-component-elements-with-code-willdestroyelement-code)
2. `willClearRender`
3. `didDestroyElement`

## Lifecycle Hook Examples(生命周期钩子示例)

以下是在组件中使用生命周期钩子的一些示例。

### Resetting Presentation State on Attribute Change with `didUpdateAttrs`(当属性改变时使用`didUpdateAttrs`来重置显示状态)

`didUpdateAttrs` 运行时组件的属性发生了变化，而不是当组件由于`component.rerender`、`component.set`或者应用与模板的model或服务变化引起的重新渲染时。

由于 `didUpdateAttrs` 在重新渲染之前调用，因此可以使用此钩子在特定属性发生更改时执行代码。这个钩子可以成为观察者的有效替代品，因为它将在属性发生变化之后，重新渲染之前运行。

这个场景的一个例子是配置文件编辑器组件。An example of this scenario in action is a profile editor component.  在您编辑一个用户时，用户属性发生更改时，你可以使用`didUpdateAttrs`去清除，在编辑之前，用户所产生的任何错误状态.

```app/templates/components/profile-editor.hbs
<ul class="errors">
  {{#each errors as |error|}}
    <li>{{error.message}}</li>
  {{/each}}
</ul>
<fieldset>
  {{input name="user.name" value=name change=(action "required")}}
  {{input name="user.department" value=department change=(action "required")}}
  {{input name="user.email" value=email change=(action "required")}}
</fieldset>
```

```/app/components/profile-editor.js
import Component from '@ember/component';

export default Component.extend({
  init() {
    this._super(...arguments);
    this.errors = [];
  },

  didUpdateAttrs() {
    this._super(...arguments);
    this.set('errors', []);
  },

  actions: {
    required(event) {
      if (!event.target.value) {
        this.get('errors').pushObject({ message: `${event.target.name} is required`});
      }
    }
  }
});
```

### Formatting Component Attributes with `didReceiveAttrs`(使用`didReceiveAttrs`格式化组件属性)

`didReceiveAttrs` 运行在 `init`之后, 它也在后续的重新渲染上运行，这对于所有渲染中相同的逻辑非常有用。重新渲染在内部启动时不会运行。
It does not run when the re-render has been initiated internally.

Since the `didReceiveAttrs` hook is called every time a component's attributes are updated whether on render or re-render,
you can use the hook to effectively act as an observer, ensuring code is executed every time an attribute changes.

For example, if you have a component that renders based on a json configuration, but you want to provide your component with the option of taking the config as a string,
you can leverage `didReceiveAttrs` to ensure the incoming config is always parsed.

```app/components/profile-editor.js
import Component from '@ember/component';

export default Component.extend({
  didReceiveAttrs() {
    this._super(...arguments);
    const profile = this.get('data');
    if (typeof profile === 'string') {
      this.set('profile', JSON.parse(profile));
    } else {
      this.set('profile', profile);
    }
  }
});
```

### Integrating with Third-Party Libraries with `didInsertElement`

Suppose you want to integrate your favorite date picker library into an Ember project.
Typically, 3rd party JS/jQuery libraries require a DOM element to bind to.
So, where is the best place to initialize and attach the library?

After a component successfully renders its backing HTML element into the DOM, it will trigger its [`didInsertElement()`][did-insert-element] hook.

Ember guarantees that, by the time `didInsertElement()` is called:

1. The component's element has been both created and inserted into the
   DOM.
2. The component's element is accessible via the component's
   [`$()`][dollar]
   method.

A component's [`$()`][dollar] method allows you to access the component's DOM element by returning a JQuery element.
For example, you can set an attribute using jQuery's `attr()` method:

```app/components/profile-editor.js
import Component from '@ember/component';

export default Component.extend({
  didInsertElement() {
    this._super(...arguments);
    this.$().attr('contenteditable', true);
  }
});
```

[`$()`][dollar] will, by default, return a jQuery object for the component's root element, but you can also target child elements within the component's template by passing a selector:

```app/components/profile-editor.js
import Component from '@ember/component';

export default Component.extend({
  didInsertElement() {
    this._super(...arguments);
    this.$('div p button').addClass('enabled');
  }
});
```

Let's initialize our date picker by overriding the [`didInsertElement()`][did-insert-element] method.

Date picker libraries usually attach to an `<input>` element, so we will use jQuery to find an appropriate input within our component's template.

```app/components/profile-editor.js
import Component from '@ember/component';

export default Component.extend({
  didInsertElement() {
    this._super(...arguments);
    this.$('input.date').myDatePickerLib();
  }
});
```

[`didInsertElement()`][did-insert-element] is also a good place to
attach event listeners. This is particularly useful for custom events or
other browser events which do not have a [built-in event
handler][event-names].

For example, perhaps you have some custom CSS animations trigger when the component
is rendered and you want to handle some cleanup when it ends:

```app/components/profile-editor.js
import Component from '@ember/component';

export default Component.extend({
  didInsertElement() {
    this._super(...arguments);
    this.$().on('animationend', () => {
      $(this).removeClass('sliding-anim');
    });
  }
});
```

There are a few things to note about the `didInsertElement()` hook:

- It is only triggered once when the component element is first rendered.
- In cases where you have components nested inside other components, the child component will always receive the `didInsertElement()` call before its parent does.
- Setting properties on the component in [`didInsertElement()`][did-insert-element] triggers a re-render, and for performance reasons,
  is not allowed.
- While [`didInsertElement()`][did-insert-element] is technically an event that can be listened for using `on()`, it is encouraged to override the default method itself,
  particularly when order of execution is important.

[did-insert-element]: https://www.emberjs.com/api/ember/release/classes/Component/events/didInsertElement?anchor=didInsertElement
[dollar]: https://www.emberjs.com/api/ember/release/classes/Component/methods/$?anchor=%24
[event-names]: http://guides.emberjs.com/v2.1.0/components/handling-events/#toc_event-names

### Making Updates to the Rendered DOM with `didRender`

The `didRender` hook is called during both render and re-render after the template has rendered and the DOM updated.
You can leverage this hook to perform post-processing on the DOM of a component after it's been updated.

In this example, there is a list component that needs to scroll to a selected item when rendered.
Since scrolling to a specific spot is based on positions within the DOM, we need to ensure that the list has been rendered before scrolling.
We can first render this list, and then set the scroll.

The component below takes a list of items and displays them on the screen.
Additionally, it takes an object representing which item is selected and will select and set the scroll top to that item.

```app/templates/application.hbs
{{selected-item-list items=items selectedItem=selection}}
```

When rendered the component will iterate through the given list and apply a class to the one that is selected.


```app/templates/components/selected-item-list.hbs
{{#each items as |item|}}
  <div class="list-item {{if item.isSelected 'selected-item'}}">{{item.label}}</div>
{{/each}}
```

The scroll happens on `didRender`, where it will scroll the component's container to the element with the selected class name.

```/app/components/selected-item-list.js
import Component from '@ember/component';

export default Component.extend({
  classNames: ['item-list'],

  didReceiveAttrs() {
    this._super(...arguments);
    this.set('items', this.get('items').map((item) => {
      if (item.id === this.get('selectedItem.id')) {
        item.isSelected = true;
      }
      return item;
    }));
  },

  didRender() {
    this._super(...arguments);
    this.$('.item-list').scrollTop(this.$('.selected-item').position.top);
  }
});
```

### Detaching and Tearing Down Component Elements with `willDestroyElement`

When a component detects that it is time to remove itself from the DOM, Ember will trigger the [`willDestroyElement()`](https://www.emberjs.com/api/ember/release/classes/Component/events/willDestroyElement?anchor=willDestroyElement) method,
allowing for any teardown logic to be performed.

Component teardown can be triggered by a number of different conditions.
For instance, the user may navigate to a different route, or a conditional Handlebars block surrounding your component may change:

```app/templates/application.hbs
{{#if falseBool}}
  {{my-component}}
{{/if}}
```

Let's use this hook to cleanup our date picker and event listener from above:

```app/components/profile-editor.js
import Component from '@ember/component';

export default Component.extend({
  willDestroyElement() {
    this.$().off('animationend');
    this.$('input.date').myDatepickerLib().destroy();
    this._super(...arguments);
  }
});
```
