Ember的内部和大部分代码将在运行循环中进行。运行循环用于批处理，并以最有效和高效的方式进行排序（或重新排序）。

它通过调度特定队列的工作来实现.这些队列具有优先级，并按优先级顺序处理完成。

对于基本的Ember应用程序开发方案，您不需要了解运行循环或直接使用它。所有常用方式已经为你友好的提供，不需要直接使用运行循环。
All common paths are paved nicely for you and don't require working with the run loop directly.

使用运行循环的最常见情况是与包含某种异步回调的非Ember API集成。例如：

- DOM 更新和事件回调
- `setTimeout` 和 `setInterval` 回调
- `postMessage` 和 `messageChannel` 事件回调
- AJAX 回调
- Websocket 回调

## Why is the run loop useful?(为什么运行循环这么有用？)

很多时候，分批处理类似的工作是有益的。Web浏览器可以完成相似的操作，通过批量更改DOM。Very often, batching similar work has benefits.
Web browsers do something quite similar by batching changes to the DOM.

考虑下面的HTML片段：

```html
<div id="foo"></div>
<div id="bar"></div>
<div id="baz"></div>
```

并执行以下代码：

```javascript
foo.style.height = '500px' // write
foo.offsetHeight // read (recalculate style, layout, expensive!)

bar.style.height = '400px' // write
bar.offsetHeight // read (recalculate style, layout, expensive!)

baz.style.height = '200px' // write
baz.offsetHeight // read (recalculate style, layout, expensive!)
```

在这个例子中，代码序列迫使浏览器重新计算风格，并在每一步之后重新布局。但是，如果我们能够将相似的作业分批处理，那么浏览器只需重新​​计算一次样式和布局即可。

```javascript
foo.style.height = '500px' // write
bar.style.height = '400px' // write
baz.style.height = '200px' // write

foo.offsetHeight // read (recalculate style, layout, expensive!)
bar.offsetHeight // read (fast since style and layout are already known)
baz.offsetHeight // read (fast since style and layout are already known)
```

有趣的是，这种模式适用于许多其他类型的工作。从本质上讲，批处理类似的工作可以实现更好的流水线和进一步的优化。

我们来看一个在Ember中优化的类似示例， 从一个 `User` 对象开始：

```javascript
import EmberObject, {
  computed
} from '@ember/object';

let User = EmberObject.extend({
  firstName: null,
  lastName: null,

  fullName: computed('firstName', 'lastName', function() {
    return `${this.get('firstName')} ${this.get('lastName')}`;
  })
});
```

和一个模板来显示其属性：

```handlebars
{{firstName}}
{{fullName}}
```

如果我们在没有运行循环的情况下执行以下代码：

```javascript
let user = User.create({ firstName: 'Tom', lastName: 'Huda' });
user.set('firstName', 'Yehuda');
// {{firstName}} and {{fullName}} are updated

user.set('lastName', 'Katz');
// {{lastName}} and {{fullName}} are updated
```

我们看到浏览器会将模板重新渲染两次。

但是，如果我们在上面的代码中有循环，浏览器只会在属性全部设置完成后重新渲染模板。

```javascript
let user = User.create({ firstName: 'Tom', lastName: 'Huda' });
user.set('firstName', 'Yehuda');
user.set('lastName', 'Katz');
user.set('firstName', 'Tom');
user.set('lastName', 'Huda');
```

在上面的运行循环示例中，由于用户属性的结果与执行前相同，所以模板甚至不会重新渲染！

当然，可以根据具体情况优化这些方案，但免费获得这些方案会更好。使用运行循环，我们不仅可以将这些类的优化应用于每个场景，还可以应用于整个应用程序。

## How does the Run Loop work in Ember?(运行循环如何在Ember中工作？)

如前所述，我们在队列上安排工作（以函数调用的形式），并按优先级顺序处理这些队列。

队列是什么，他们的优先顺序是什么？

```javascript
Ember.run.queues
// => ["sync", "actions", "routerTransitions", "render", "afterRender", "destroy"]
```

由于优先级是从高到低的，"sync"队列比"render" 或 "destroy"队列有更高的优先级。

## What happens in these queues?(在这些队列中会发生什么？)

* `sync` 队列包含同步作业。
* `actions` 队列是一般的工作队列，并通常会含有预定任务，例如 promises.
* `routerTransitions` 队列包含在路由器过渡工作.
* `render` 队列包含的工作意味着渲染，这些通常会更新DOM。
* `afterRender` 队列包含在所有先前计划的渲染任务完成之后运行的作业。这通常对于第三方DOM操作库很有用，只应在整个DOM树更新后才能运行。
* `destroy` 队列包含完成其他计划销毁的对象的拆卸的作业。

## In what order are jobs executed on the queues?(作业在队列上执行的顺序是什么？)
该算法以这种方式工作：

1. 让具有挂起作业的最高优先级队列为：`CURRENT_QUEUE`,如果没有挂起作业的队列，则运行循环完成
2. 让一个新的临时队列被定义为 `WORK_QUEUE`
3. 将作业从 `CURRENT_QUEUE` 移入 `WORK_QUEUE`
4. 按顺序处理所有作业在 `WORK_QUEUE`中
5. 返回到步骤1

## An example of the internals(内部的一个例子)

我们没有编写内部调用各种运行循环调度功能的高级应用程序代码，而是剥离了封面，并显示了原始运行循环交互。

Rather than writing the higher level app code that internally invokes the various run loop scheduling functions,
we have stripped away the covers, and shown the raw run-loop interactions.

直接使用此API在大多数Ember应用程序中并不常见，但理解此示例将帮助您了解运行循环算法，这将使您成为更好的Ember开发人员。

<iframe src="https://s3.amazonaws.com/emberjs.com/run-loop-guide/index.html" width="678" height="410" style="border:1px solid rgb(170, 170, 170);margin-bottom:1.5em;"></iframe>

## How do I tell Ember to start a run loop?(我如何告诉Ember开始运行循环？)

当回调触发时，您应该开始运行循环。

You should begin a run loop when the callback fires.

`Ember.run` 方法可用于创建运行循环。
在这个例子中， jQuery 和 `Ember.run` 被用来处理一个点击事件和运行一些Ember代码。

本示例使用 `=>` 函数语法， 该语法是一种 [提供回调函数的新ES2015语法]
(https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
它提供了一个词法 `this`.
如果这个语法是新的，那么把它看作是一个函数， `this` 作为上下文被定义.

```javascript
$('a').click(() => {
  Ember.run(() => {  // begin loop
    // Code that results in jobs being scheduled goes here
  }); // end loop, jobs are flushed and executed
});
```

## What happens if I forget to start a run loop in an async handler?(如果我忘记在异步处理程序中启动运行循环会发生什么？)

如上所述，你应该把任何非Ember的异步调用放在 `Ember.run`中.如果你不这样做，Ember将为你尝试使用开始和结束几乎一致的方式（译者注：应该说的是同步的方法）。
考虑以下回调：

```javascript
$('a').click(() => {
  console.log('Doing things...');

  Ember.run.schedule('actions', () => {
    // Do more things
  });
});
```

运行中的循环API调用 _schedule_ 工作, 例如 [`run.schedule`](https://www.emberjs.com/api/ember/release/classes/@ember%2Frunloop/methods/schedule?anchor=schedule),
[`run.scheduleOnce`](https://www.emberjs.com/api/ember/release/classes/@ember%2Frunloop/methods/scheduleOnce?anchor=scheduleOnce),
[`run.once`](https://www.emberjs.com/api/ember/release/classes/@ember%2Frunloop/methods/once?anchor=once) 如果内容不是已经存在，它们拥有近似于循环运行的特性（原文：have the property that they will approximate a run loop for you if one does not already exist.）
这些自动创建的运行循环称为 _autoruns_.

下面是使用上述示例描述发生的一些伪代码：

```javascript
$('a').click(() => {
  // 1. autoruns do not change the execution of arbitrary code in a callback.
  //    This code is still run when this callback is executed and will not be
  //    scheduled on an autorun.
  console.log('Doing things...');

  Ember.run.schedule('actions', () => {
    // 2. schedule notices that there is no currently available run loop so it
    //    creates one. It schedules it to close and flush queues on the next
    //    turn of the JS event loop.
    if (! Ember.run.hasOpenRunLoop()) {
      Ember.run.begin();
      nextTick(() => {
        Ember.run.end()
      }, 0);
    }

    // 3. There is now a run loop available so schedule adds its item to the
    //    given queue
    Ember.run.schedule('actions', () => {
      // Do more things
    });

  });

  // 4. This schedule sees the autorun created by schedule above as an available
  //    run loop and adds its item to the given queue.
  Ember.run.schedule('afterRender', () => {
    // Do yet more things
  });
});
```

尽管自动调整很方便，但它们并不理想。当前的JS框架允许在运行循环刷新之前结束运行，这有时意味着浏览器将有机会做其他事情，比如垃圾收集。在数据更改和DOM重新渲染之间运行的GC可能会导致视觉滞后，应尽量减少。

依靠autoruns不是使用运行循环的严格或有效的方式。手动封装事件处理程序是首选。

## How is run loop behaviour different when testing?(测试时运行循环行为有何不同？)

当您的应用程序处于 _testing mode_ ，如果您尝试在没有可用运行循环的情况下安排工作，Ember将会抛出错误。

Autoruns在测试中被禁用有几个原因：

1. Autoruns是Embers的一种方式，如果您在计划回调之前忘记打开运行循环，则不会在生产中惩罚您。虽然这在生产中很有用，但这些仍然应该在测试中显示，以帮助您找到并修复它们。
2. Ember的一些测试助手承诺在解析之前等待运行循环清空。如果您的应用程序有运行在循环之 _外_ 的代码。这些代码会过早解决，并导致错误的测试失败，这些失败很难找到。
禁用autoruns可帮助您识别这些场景并帮助您的测试和应用程序！

## Where can I find more information?(我在哪里可以找到更多信息？)

查看 [Ember.run](https://www.emberjs.com/api/ember/release/classes/@ember%2Frunloop) 文档以及支持运行循环的 [Backburner  库](https://github.com/ebryn/backburner.js/)。
