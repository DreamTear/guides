正如其他的框架一样Ember也包含了多种方式绑定实现，并且可以在任何一个对象上使用绑定。也就是说，绑定大多数情况都是使用在Ember框架本身，对于开发最好还是使用计算属性。


创建双向绑定最简单的方式是使用 [`computed.alias()`](https://www.emberjs.com/api/ember/release/classes/@ember%2Fobject%2Fcomputed/methods/alias?anchor=alias&show=inherited%2Cprotected%2Cprivate%2Cdeprecated),
指定另一个对象的路径。

```javascript
import EmberObject from '@ember/object';
import { alias } from '@ember/object/computed';

husband = EmberObject.create({
  pets: 0
});

Wife = EmberObject.extend({
  pets: alias('husband.pets')
});

wife = Wife.create({
  husband: husband
});

wife.get('pets'); // 0

// Someone gets a pet.
husband.set('pets', 1);
wife.get('pets'); // 1
```

Note that bindings don't update immediately. Ember waits until all of your
application code has finished running before synchronizing changes, so you can
change a bound property as many times as you'd like without worrying about the
overhead of syncing bindings when values are transient.

## One-Way Bindings

A one-way binding only propagates changes in one direction, using
[`computed.oneWay()`](https://www.emberjs.com/api/ember/release/classes/@ember%2Fobject%2Fcomputed/methods/alias?anchor=oneWay&show=inherited%2Cprotected%2Cprivate%2Cdeprecated). Often, one-way bindings are a performance
optimization and you can safely use a two-way binding (which are de facto one-way bindings if you only ever change one side).
Sometimes one-way bindings are useful to achieve specific behaviour such as a
default that is the same as another property but can be overridden (e.g. a
shipping address that starts the same as a billing address but can later be
changed)

```javascript
import EmberObject, { computed } from '@ember/object';
import Component from '@ember/component';
import { oneWay } from '@ember/object/computed';

user = EmberObject.create({
  fullName: 'Kara Gates'
});

UserComponent = Component.extend({
  userName: oneWay('user.fullName')
});

userComponent = UserComponent.create({
  user: user
});

// Changing the name of the user object changes
// the value on the view.
user.set('fullName', 'Krang Gates');
// userComponent.userName will become "Krang Gates"

// ...but changes to the view don't make it back to
// the object.
userComponent.set('userName', 'Truckasaurus Gates');
user.get('fullName'); // "Krang Gates"
```
