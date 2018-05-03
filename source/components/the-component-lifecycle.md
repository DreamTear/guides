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

由于每次在渲染或重新渲染时更新组件的属性都会调用`didReceiveAttrs`钩子，因此可以使用钩子来有效地充当观察者，确保每次属性更改时都执行代码。

例如，如果您有一个基于json配置呈现的组件，但您希望为组件提供将配置作为字符串的选项，则可以利用`didReceiveAttrs`来确保始终分析传入配置。

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

### Integrating with Third-Party Libraries with `didInsertElement`(使用`didInsertElement`与第三方库集成在一起)

假设你想将你最喜欢的日期选择器库集成到Ember项目中。通常，第三方JS / jQuery库需要一个DOM元素。那么，初始化和附加库的最佳位置在哪里呢？

组件成功将其HTML元素渲染到DOM后，将触发[`didInsertElement()`][did-insert-element]钩子。

Ember在`didInsertElement()`被调用时保证：

1. 组件的元素被创建并且插入到DOM中.
2. 组件的元素可以通过组件的[`$()`][dollar]方法访问 。

组件的 [`$()`][dollar] 方法允许您通过返回JQuery元素来访问组件的DOM元素。
例如，您可以使用jQuery的 `attr()` 方法设置一个属性:

```app/components/profile-editor.js
import Component from '@ember/component';

export default Component.extend({
  didInsertElement() {
    this._super(...arguments);
    this.$().attr('contenteditable', true);
  }
});
```

[`$()`][dollar] 默认情况下，会为组件的根元素返回一个jQuery对象，但是您还可以通过传递一个选择器来定位组件模板中的子元素:

```app/components/profile-editor.js
import Component from '@ember/component';

export default Component.extend({
  didInsertElement() {
    this._super(...arguments);
    this.$('div p button').addClass('enabled');
  }
});
```

我们通过重写 [`didInsertElement()`][did-insert-element] 方法来初始化我们的日期选择器。

日期选择器库通常附加到一个 `<input>` 元素, 所以我们将使用jQuery在我们的组件模板中找到合适的输入。

```app/components/profile-editor.js
import Component from '@ember/component';

export default Component.extend({
  didInsertElement() {
    this._super(...arguments);
    this.$('input.date').myDatePickerLib();
  }
});
```

[`didInsertElement()`][did-insert-element] 也是附加事件监听器的好地方。这对于没有 [内置事件处理程序][event-names]的自定义事件或其他浏览器事件特别有用。

例如，也许你有一些自定义的CSS动画在组件被渲染时触发，在结束时做一些清理:

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

有关 `didInsertElement()` 钩子的一些事情需要注意:

- 只有在组件元素第一次呈现时才会触发一次。
- 在组件嵌套在其他组件内的情况下，子组件将始终在它的父组件之前调用 `didInsertElement()` 。
- 在 [`didInsertElement()`][did-insert-element] 中设置组件中的属性会触发重新渲染，并且出于性能方面的考虑，这是不允许的。
- 虽然 [`didInsertElement()`][did-insert-element] 从技术上讲是可以使用`on()`监听，但鼓励重写默认方法本身，特别是在执行顺序很重要时。

[did-insert-element]: https://www.emberjs.com/api/ember/release/classes/Component/events/didInsertElement?anchor=didInsertElement
[dollar]: https://www.emberjs.com/api/ember/release/classes/Component/methods/$?anchor=%24
[event-names]: http://guides.emberjs.com/v2.1.0/components/handling-events/#toc_event-names

### Making Updates to the Rendered DOM with `didRender`(更新已经渲染的DOM使用`didRender`)

 `didRender` 在模板渲染和DOM更新后，在渲染和重新渲染过程中调用该钩子。您可以利用此挂钩在组件的DOM更新后对其执行后处理。

在这个例子中，有一个列表组件需要在渲染时滚动到选定的项目。由于滚动到特定位置是基于DOM内的位置，因此我们需要确保列表在滚动前已经被渲染。我们可以先渲染这个列表，然后设置滚动条。

下面的组件获取项目列表并将它们显示在屏幕上。此外，它需要一个对象来表示选择哪个项目，然后选中并将滚动条滚动到这个列表项。

```app/templates/application.hbs
{{selected-item-list items=items selectedItem=selection}}
```

呈现时，组件将遍历给定列表并将class添加到选定的项目上。


```app/templates/components/selected-item-list.hbs
{{#each items as |item|}}
  <div class="list-item {{if item.isSelected 'selected-item'}}">{{item.label}}</div>
{{/each}}
```

当`didRender`时，组件的容器将滚动到所选class的元素。

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

### Detaching and Tearing Down Component Elements with `willDestroyElement`(分离和拆卸组件元素在`willDestroyElement`)

当组件检测到是时候从DOM中移除它时，Ember将触发该[`willDestroyElement()`](https://www.emberjs.com/api/ember/release/classes/Component/events/willDestroyElement?anchor=willDestroyElement)方法，从而允许执行任何拆卸逻辑。

部件拆卸可以由许多不同的条件触发。例如，用户可能导航到不同的路线，或者围绕您的组件的条件Handlebars块可能会改变：

```app/templates/application.hbs
{{#if falseBool}}
  {{my-component}}
{{/if}}
```

让我们使用这个钩子从上面清理我们的日期选择器和事件监听器：

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
