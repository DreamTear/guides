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

## An example of the internals

Rather than writing the higher level app code that internally invokes the various run loop scheduling functions,
we have stripped away the covers, and shown the raw run-loop interactions.

Working with this API directly is not common in most Ember apps,
but understanding this example will help you to understand the run-loops algorithm,
which will make you a better Ember developer.

<iframe src="https://s3.amazonaws.com/emberjs.com/run-loop-guide/index.html" width="678" height="410" style="border:1px solid rgb(170, 170, 170);margin-bottom:1.5em;"></iframe>

## How do I tell Ember to start a run loop?

You should begin a run loop when the callback fires.

The `Ember.run` method can be used to create a run loop.
In this example, jQuery and `Ember.run` are used to handle a click event and run some Ember code.

This example uses the `=>` function syntax, which is a [new ES2015 syntax for callback functions]
(https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
that provides a lexical `this`.
If this syntax is new,
think of it as a function that has the same `this` as the context it is defined in.

```javascript
$('a').click(() => {
  Ember.run(() => {  // begin loop
    // Code that results in jobs being scheduled goes here
  }); // end loop, jobs are flushed and executed
});
```

## What happens if I forget to start a run loop in an async handler?

As mentioned above, you should wrap any non-Ember async callbacks in `Ember.run`.
If you don't, Ember will try to approximate a beginning and end for you.
Consider the following callback:

```javascript
$('a').click(() => {
  console.log('Doing things...');

  Ember.run.schedule('actions', () => {
    // Do more things
  });
});
```

The run loop API calls that _schedule_ work, i.e. [`run.schedule`](https://www.emberjs.com/api/ember/release/classes/@ember%2Frunloop/methods/schedule?anchor=schedule),
[`run.scheduleOnce`](https://www.emberjs.com/api/ember/release/classes/@ember%2Frunloop/methods/scheduleOnce?anchor=scheduleOnce),
[`run.once`](https://www.emberjs.com/api/ember/release/classes/@ember%2Frunloop/methods/once?anchor=once) have the property that they will approximate a run loop for you if one does not already exist.
These automatically created run loops we call _autoruns_.

Here is some pseudocode to describe what happens using the example above:

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

Although autoruns are convenient, they are suboptimal.
The current JS frame is allowed to end before the run loop is flushed,
which sometimes means the browser will take the opportunity to do other things, like garbage collection.
GC running in between data changing and DOM rerendering can cause visual lag and should be minimized.

Relying on autoruns is not a rigorous or efficient way to use the run loop.
Wrapping event handlers manually are preferred.

## How is run loop behaviour different when testing?

When your application is in _testing mode_ then Ember will throw an error if you try to schedule work
without an available run loop.

Autoruns are disabled in testing for several reasons:

1. Autoruns are Embers way of not punishing you in production if you forget to open a run loop
before you schedule callbacks on it.
While this is useful in production, these are still situations that should be revealed in testing
to help you find and fix them.
2. Some of Ember's test helpers are promises that wait for the run loop to empty before resolving.
If your application has code that runs _outside_ a run loop,
these will resolve too early and give erroneous test failures which are difficult to find.
Disabling autoruns help you identify these scenarios and helps both your testing and your application!

## Where can I find more information?

Check out the [Ember.run](https://www.emberjs.com/api/ember/release/classes/@ember%2Frunloop) API documentation,
as well as the [Backburner library](https://github.com/ebryn/backburner.js/) that powers the run loop.
