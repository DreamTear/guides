您可以响应组件上的用户事件，如双击、鼠标悬停、和键盘事件。只需在组件上实现要作为响应的事件的名称即可。

例如，假设我们有一个这样的模板：

```hbs
{{#double-clickable}}
  This is a double clickable area!
{{/double-clickable}}
```

当它被点击时，我们让`double-clickable` 显示一个警告：

```app/components/double-clickable.js
import Component from '@ember/component';

export default Component.extend({
  doubleClick() {
    alert('DoubleClickableComponent was clicked!');
  }
});
```

浏览器的事件冒泡可能会将事件传递到父组件中。 在组件事件处理函数中 `return true;` 来使事件可以冒泡.

```app/components/double-clickable.js
import Component from '@ember/component';
import Ember from 'ember';

export default Component.extend({
  doubleClick() {
    console.info('DoubleClickableComponent was clicked!');
    return true;
  }
});
```

请参阅本页末尾的事件名称列表。任何事件都可以在组件中定义事件处理程序。

## Sending Actions(发送操作)

在某些情况下，你的组件需要定义事件处理程序，可能是为了支持各种可拖动的行为。 例如，组件可能需要在收到放置事件时传递 `id`：

```hbs
{{drop-target action=(action "didDrop")}}
```

您可以定义组件的事件处理程序来管理放置事件。如果你需要，你也可以通过使用`return false;`来阻止事件冒泡.

```app/components/drop-target.js
import Component from '@ember/component';

export default Component.extend({
  attributeBindings: ['draggable'],
  draggable: 'true',

  dragOver() {
    return false;
  },

  drop(event) {
    let id = event.dataTransfer.getData('text/data');
    this.get('action')(id);
  }
});
```

在上面的组件中， `didDrop` 是 `action` 传入的。 这个动作是从 `drop` 事件处理程序调用的，并将一个参数传递给动作 -   通过`drop` 事件对象找到`id`的值.

另一种保存本地事件行为并使用动作的方法是分配一个行内的行为。考虑下面的模板，其中包含元素`button`的  `onclick`处理函数:

```hbs
<button onclick={{action "signUp"}}>Sign Up</button>
```

`signUp`只是组件散列`actions`中的一个。由于该操作被分配给内联处理函数，函数定义可以将事件对象定义为其第一个参数。

```js
actions: {
  signUp(event){
  	// Only when assigning the action to an inline handler, the event object
    // is passed to the action as the first parameter.
  }
}
```

定义在`actions`中的方法，正常情况下是不会接收到浏览器事件作为参数. 所以，该动作的函数定义不能定义一个事件参数。 以下示例演示了使用操作的默认行为。

```hbs
<button {{action "signUp"}}>Sign Up</button>
```

```js
actions: {
  signUp() {
    // No event object is passed to the action.
  }
}
```

要将 `event` 对象用作函数参数：

- 在组件中定义事件处理函数(用于接收浏览器事件对象).
- 或者，为模板中的内联事件处理程序分配一个操作（创建一个闭包操作并将事件对象作为参数接收）。


## Event Names(事件名称)

上面描述的事件处理示例响应一组事件。下面列出了内置事件的名称。自定义事件可以使用 [Application.customEvents](https://www.emberjs.com/api/ember/release/classes/Application/properties/customEvents?anchor=customEvents)进行注册。

触摸事件：

* `touchStart`
* `touchMove`
* `touchEnd`
* `touchCancel`

键盘事件

* `keyDown`
* `keyUp`
* `keyPress`

鼠标事件

* `mouseDown`
* `mouseUp`
* `contextMenu`
* `click`
* `doubleClick`
* `mouseMove`
* `focusIn`
* `focusOut`
* `mouseEnter`
* `mouseLeave`

表单事件:

* `submit`
* `change`
* `focusIn`
* `focusOut`
* `input`

HTML5拖放事件:

* `dragStart`
* `drag`
* `dragEnter`
* `dragLeave`
* `dragOver`
* `dragEnd`
* `drop`
