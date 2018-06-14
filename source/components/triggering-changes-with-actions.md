您可以将组件视为UI功能的黑盒子。到目前为止，您已经学习了父组件如何将属性传递给子组件，以及该组件如何使用JavaScript及其模板中的这些属性。

但是相反的方向呢？数据如何从组件流回父级？在Ember中，组件使用 **actions** 来传达事件和更改。

我们来看一个组件如何使用一个动作与其父代进行通信的简单示例。

想象一下，我们正在构建一个用户可以拥有账户的应用程序。我们需要为用户构建用户界面以删除其帐户。由于我们不希望用户意外删除其帐户，因此我们将构建一个按钮，要求用户确认才能触发某些操作。

一旦我们创建了“确认按钮”组件，我们希望能够在我们的应用程序中重用它。

## Creating the Component（创建组件）

我们来创建我们的组件 `button-with-confirmation`. 我们可以通过键入以下命令:

```shell
ember generate component button-with-confirmation
```

我们打算在模板中使用这个组件，如下所示:

```app/templates/components/user-profile.hbs
{{button-with-confirmation
  text="Click OK to delete your account."
}}
```

我们也想在其他地方使用该组件，可能是这样的:

```app/templates/components/send-message.hbs
{{button-with-confirmation
  text="Click OK to send your message."
}}
```

## Designing the Action（设计动作）

在对组件外部处理的组件执行操作时，需要将其分解为两个步骤:

1. 在父组件中，决定如何对动作做出反应。在这里，我们希望在一个地方使用该用户的帐户时删除该用户的帐户，并在另一个地方使用时发送消息。
2. 在组件中，确定什么时候发生了什么，以及何时告诉外部世界。 在这里，我们想要在用户点击按钮并确认后触发外部操作（删除账户或发送消息）。

让我们一步一步来。

## Implementing the Action(实施行动)

在父组件中，我们首先定义当用户单击按钮并确认后我们想要发生的事情。 在第一种情况下，我们会找到用户的帐户并将其删除。

在Ember中，每个组件都可以拥有一个名为 `actions`的属性, 您可以在其中放置
[可由用户与组件本身交互](../../templates/actions/), 或由子组件与其进行交互的函数.

我们来看看父组件的JavaScript文件。在这个例子中，假设我们有一个父组件 `user-profile` ，它向用户显示了用户的配置文件。

我们在父组件中提供一个`userDidDeleteAccount()`方法，当它被调用时，假设从  `login`
[服务](../../applications/services/) 中调用`deleteUser()` 方法。

```app/components/user-profile.js
import Component from '@ember/component';
import { inject as service } from '@ember/service';

export default Component.extend({
  login: service(),

  actions: {
    userDidDeleteAccount() {
      this.get('login').deleteUser();
    }
  }
});
```

现在我们已经实施了我们的行动，但是我们还没有告诉Ember我们想让它什么时候被触发，这是下一步要做的。

## Designing the Child Component(设计子组件)

接下来，在子组件中，我们将执行逻辑以确认用户想要通过单击按钮来执行他们指示的操作:

```app/components/button-with-confirmation.js
import Component from '@ember/component';

export default Component.extend({
  actions: {
    launchConfirmDialog() {
      this.set('confirmShown', true);
    },

    submitConfirm() {
      // trigger action on parent component
      this.set('confirmShown', false);
    },

    cancelConfirm() {
      this.set('confirmShown', false);
    }
  }
});
```

组件模板将有一个按钮和一个显示基于值`confirmShown`的确认对话框。

```app/templates/components/button-with-confirmation.hbs
<button {{action "launchConfirmDialog"}}>{{text}}</button>
{{#if confirmShown}}
  <div class="confirm-dialog">
    <button class="confirm-submit" {{action "submitConfirm"}}>OK</button>
    <button class="confirm-cancel" {{action "cancelConfirm"}}>Cancel</button>
  </div>
{{/if}}
```

## Passing the Action to the Component(将操作传递给组件)

现在我们需要做到这一点，在父组件`user-profile`中定义的action`userDidDeleteAccount()`可以被`button-with-confirmation`触发。
我们将通过以与传递其他属性完全相同的方式将action传递给子组件来完成此操作。这是可能的，因为动作只是函数，就像组件上的其他方法一样，因此它们可以像下面这样从一个组件传递到另一个组件：

```app/templates/components/user-profile.hbs
{{button-with-confirmation
  text="Click here to delete your account."
  onConfirm=(action "userDidDeleteAccount")
}}
```

这段代码的意思是 "从父组件中获取action `userDidDeleteAccount` 并将它作为子组件的`onConfirm`属性传递给子组件."
请注意这里使用的 `action` helper,它用于给组件传递名字为`"userDidDeleteAccount"`的方法。

我们可以为我们的 `send-message` 组件做类似的事情:

```app/templates/components/send-message.hbs
{{button-with-confirmation
  text="Click to send your message."
  onConfirm=(action "sendMessage")
}}
```

现在，我们可以在子组件中使用 `onConfirm` 来调用父组件的action:

```app/components/button-with-confirmation.js
import Component from '@ember/component';

export default Component.extend({
  actions: {
    launchConfirmDialog() {
      this.set('confirmShown', true);
    },

    submitConfirm() {
      //call the onConfirm property to invoke the passed in action
      this.get('onConfirm')();
    },

    cancelConfirm() {
      this.set('confirmShown', false);
    }
  }
});
```

`this.get('onConfirm')` 将返回父组件中传递的 `onConfirm`的函数, 接下来的 `()` 将调用这个方法.

与普通属性一样，动作可以是组件上的属性; 唯一的区别是该属性设置为知道如何触发行为的函数( the
only difference is that the property is set to a function that knows how
to trigger behavior)。

这使得记住如何向组件添加操作变得很容易。 这就像传递一个属性，但你使用 `action` helper 来传递一个函数。

组件中的操作允许您将发生的事件从其处理方式中分离出来，从而形成模块化，更可重用的组件。
Actions in components allow you to decouple an event happening from how it's handled, leading to modular,
more reusable components.

## Handling Action Completion(处理行动完成)

通常动作执行异步任务，例如向服务器发出ajax请求。由于动作是可以由父组件传入的函数，因此它们可以在调用时返回值。最常见的情况是操作返回promise，以便组件可以处理操作的完成。

在我们的用户 `button-with-confirmation` 组件中，我们希望保留确认模式，直到我们知道操作已成功完成。这是通过期望从`onConfirm`返回的promise来实现的。
在解决promise后，我们设置了一个用于指示确认模式可见性的属性。

```app/components/button-with-confirmation.js
import Component from '@ember/component';

export default Component.extend({
  actions: {
    launchConfirmDialog() {
      this.set('confirmShown', true);
    },

    submitConfirm() {
      // call onConfirm with the value of the input field as an argument
      let promise = this.get('onConfirm')();
      promise.then(() => {
        this.set('confirmShown', false);
      });
    },

    cancelConfirm() {
      this.set('confirmShown', false);
    }
  }
});
```

## Passing Arguments(传递参数)

有时，父组件调用的action需要一些上下文但这些上下文子组件没有(Sometimes the parent component invoking an action has some context needed for the action that the child component
doesn't)。例如，考虑一下`button-with-confirmation`组件被我们定义为需要使用`send-message`的情况。我们传递给子组件的`sendMessage`action可能需要提供一个消息类型参数作为参数:

```app/components/send-message.js
import Component from '@ember/component';

export default Component.extend({
  actions: {
    sendMessage(messageType) {
      //send message here and return a promise
    }
  }
});
```

但是，调用动作的组件`button-with-confirmation`不知道或关心它正在收集什么类型的消息。
在这种情况下，父级模板可以在将操作传递给子级时提供所需的参数。例如，如果我们想使用按钮
发送类型为 `"info"`:

```app/templates/components/send-message.hbs
{{button-with-confirmation
  text="Click to send your message."
  onConfirm=(action "sendMessage" "info")
}}
```

在 `button-with-confirmation`内部, `submitConfirm` action 的代码不用改变.
它仍然会调用 `onConfirm` 不用明确的参数:

```app/components/button-with-confirmation.js
const promise = this.get('onConfirm')();
```
然而 `(action "sendMessage" "info")`将动作传递给组件的表达式会创建一个闭包,
即一个绑定我们提供给指定函数的参数的对象。所以现在，当调用动作时，该参数将自动作为其参数
传递，有效地调用`sendMessage("info")`,尽管参数不会出现在调用代码中。

到目前为止，在我们的例子中，我们传递给`button-with-confirmation`的action是一个接受一
个参数`messageType`的方法。假设我们想通过允许`sendMessage`传递用户发送
的消息的实际文本作为第二个参数来扩展它：

```app/components/send-message.js
import Component from '@ember/component';

export default Component.extend({
  actions: {
    sendMessage(messageType, messageText) {
      //send message here and return a promise
    }
  }
});
```

我们想在`button-with-confirmation`内部使用两个参数调用这个action。我们已经知道如果我们
提供一个`messageType`给 `action` helper当我们将`button-with-confirmation`插入到它的父组件`send-message`模板中时，
这个值将被传递给`sendMessage` action作为它的第一个参数当调用`onConfirm`时。如果我们随后传递一个额外的参数给`onConfirm`，那么这个参数将作为`sendMessage`的第二个参数传递给这个函数（这种为函数提供参数的功能被称为[currying](https://en.wikipedia.org/wiki/Currying)）。

In our case, the explicit argument that we pass to `onConfirm` will be the required `messageText`.
However, remember that internally our `button-with-confirmation` component does not know or care that it is being used in a messaging application.
Therefore within the component's javascript file,
we will use a property `confirmValue` to represent that argument and pass it to `onConfirm` as shown here:

```app/components/button-with-confirmation.js
import Component from '@ember/component';

export default Component.extend({
  actions: {
    //...
    submitConfirm() {
      // call onConfirm with a second argument
      let promise = this.get('onConfirm')(this.get('confirmValue'));
      promise.then(() => {
        this.set('confirmShown', false);
      });
    },
    //...
  }
});
```

In order for `confirmValue` to take on the value of the message text,
we'll bind the property to the value of a user input field that will appear when the button is clicked.
To accomplish this,
we'll first modify the component so that it can be used in block form and we will [yield](../wrapping-content-in-a-component/) `confirmValue` to the block within the `"confirmDialog"` element:

```app/templates/components/button-with-confirmation.hbs
<button {{action "launchConfirmDialog"}}>{{text}}</button>
{{#if confirmShown}}
  <div class="confirm-dialog">
    {{yield confirmValue}}
    <button class="confirm-submit" {{action "submitConfirm"}}>OK</button>
    <button class="confirm-cancel" {{action "cancelConfirm"}}>Cancel</button>
  </div>
{{/if}}
```

With this modification,
we can now use the component in `send-message` to wrap a text input element whose `value` attribute is set to `confirmValue`:

```app/templates/components/send-message.hbs
{{#button-with-confirmation
    text="Click to send your message."
    onConfirm=(action "sendMessage" "info")
    as |confirmValue|}}
  {{input value=confirmValue}}
{{/button-with-confirmation}}
```

When the user enters their message into the input field,
the message text will now be available to the component as `confirmValue`.
Then, once they click the "OK" button, the `submitConfirm` action will be triggered, calling `onConfirm` with the provided `confirmValue`,
thus invoking the `sendMessage` action in `send-message` with both the `messageType` and `messageText` arguments.

## Invoking Actions Directly on Component Collaborators

Actions can be invoked on objects other than the component directly from the template.  For example, in our
`send-message` component we might include a service that processes the `sendMessage` logic.

```app/components/send-message.js
import Component from '@ember/component';
import { inject as service } from '@ember/service';

export default Component.extend({
  messaging: service(),

  // component implementation
});
```

We can tell the action to invoke the `sendMessage` action directly on the messaging service with the `target` attribute.

```app/templates/components/send-message.hbs
{{#button-with-confirmation
    text="Click to send your message."
    onConfirm=(action "sendMessage" "info" target=messaging)
    as |confirmValue| }}
  {{input value=confirmValue}}
{{/button-with-confirmation}}
```

By supplying the `target` attribute, the action helper will look to invoke the `sendMessage` action directly on the messaging
service, saving us from writing code on the component that just passes the action along to the service.

```app/services/messaging.js
import Service from '@ember/service';

export default Ember.Service.extend({
  actions: {
    sendMessage(messageType, text) {
      //handle message send and return a promise
    }
  }
});
```

## Destructuring Objects Passed as Action Arguments

A component will often not know what information a parent needs to process an action, and will just pass all the
information it has.
For example, our `user-profile` component is going to notify its parent, `system-preferences-editor`, that a
user's account was deleted, and passes along with it the full user profile object.


```app/components/user-profile.js
import Component from '@ember/component';
import { inject as service } from '@ember/service';

export default Component.extend({
  login: service(),

  actions: {
    userDidDeleteAccount() {
      this.get('login').deleteUser();
      this.get('didDelete')(this.get('login.currentUserObj'));
    }
  }
});
```

All our `system-preferences-editor` component really needs to process a user deletion is an account ID.
For this case, the action helper provides the `value` attribute to allow a parent component to dig into the passed
object to pull out only what it needs.

```app/templates/components/system-preferences-editor.hbs
{{user-profile didDelete=(action "userDeleted" value="account.id")}}
```

Now when the `system-preferences-editor` handles the delete action, it receives only the user's account `id` string.

```app/components/system-preferences-editor.js
import Component from '@ember/component';

export default Component.extend({
  actions: {
    userDeleted(idStr) {
      //respond to deletion
    }
  }
});
```

## Calling Actions Up Multiple Component Layers

When your components go multiple template layers deep, it is common to need to handle an action several layers up the tree.
Using the action helper, parent components can pass actions to child components through templates alone without adding JavaScript code to those child components.

For example, say we want to move account deletion from the `user-profile` component to its parent `system-preferences-editor`.

First we would move the `deleteUser` action from `user-profile.js` to the actions object on `system-preferences-editor`.

```app/components/system-preferences-editor.js
import Component from '@ember/component';
import { inject as service } from '@ember/service';

export default Component.extend({
  login: service(),
  actions: {
    deleteUser(idStr) {
      return this.get('login').deleteUserAccount(idStr);
    }
  }
});
```

Then our `system-preferences-editor` template passes its local `deleteUser` action into the `user-profile` as that
component's `deleteCurrentUser` property.

```app/templates/components/system-preferences-editor.hbs
{{user-profile
  deleteCurrentUser=(action 'deleteUser' login.currentUser.id)
}}
```

The action `deleteUser` is in quotes, since `system-preferences-editor` is where the action is defined now. Quotes indicate that the action should be looked for in `actions` local to that component, rather than in those that have been passed from a parent.

In our `user-profile.hbs` template we change our action to call `deleteCurrentUser` as passed above.

```app/templates/components/user-profile.hbs
{{button-with-confirmation
  onConfirm=(action deleteCurrentUser)
  text="Click OK to delete your account."
}}
```

Note that `deleteCurrentUser` is no longer in quotes here as opposed to [previously](#toc_passing-the-action-to-the-component). Quotes are used to initially pass the action down the component tree, but at every subsequent level you are instead passing the actual function reference (without quotes) in the action helper.

Now when you confirm deletion, the action goes straight to the `system-preferences-editor` to be handled in its local context.
