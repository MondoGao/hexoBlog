---
title: functional js basis
lang: en
date: 2017-11-30 20:48:43
categories: code
tags:
- functional
- fe
---
## Why Be Functional？

```javascript
// Plain JS
let incompleteTasks = tasks.filter(function(task) {
    return !task.complete;
});

// Functional
let isCompleteTask = R.whereEq({complete: false}); //=> Fun: Object -> Bool
let giveMeIncompleteTasks = R.filter(isCompleteTask); //=> Fun: Object -> Object
```

- Self-document
- Testable
- Short
- Compositional

## Basis

### Meeting Particial Apply & Curry

**Particial Apply** - Specifies some arguments up front, producing a function that's waiting for all the rest arguments.

```javascript
let greet = (salutation, title, firstName, lastName) =>
  salutation + ', ' + title + ' ' + firstName + ' ' + lastName + '!';

let sayHello = partial(greet, ['Hello']);
let sayHelloToMs = partial(sayHello, ['Ms.']);

sayHelloToMs('Jane', 'Jones'); //=> 'Hello, Ms. Jane Jones!'
```

---

Curry - Call a function with incomplete arguments, it returns **a function** which takes the remaning arguments

```javascript
let add = (x, y) => x + y;
let curriedAdd = x => y => x + y;
```

But what if I have lots of parameters?

```javascript
let crazyFun = (a, b, c, d, e) => ...;
/* curried way?
crazyFun(1)(2)(3)(4)(5);
*/
```

We can do:

```javascript
crazyFun(1, 2, 3, 4, 5);
// of
crazyFun(1, 2)(3, 4, 5);
// or
crazyFun(__, 2)(1)(3, __, 5)(4);
// or
crazyFun(__, 2)(1, __, 4)(3, 5)
```

[Example](http://ramdajs.com/repl/#?let%20crazyFun%20%3D%20%28a%2C%20b%2C%20c%29%20%3D%3E%20%7B%0A%20%20console.log%28a%29%3B%0A%20%20console.log%28b%29%3B%0A%20%20console.log%28c%29%3B%0A%7D%0A%0AR.curry%28crazyFun%29%281%2C%20R.__%29%28R.__%2C%203%29%282%29)

###Function Factory - Compose

**Compose** - The reverse of pipe

```javascript
let toUpperCase = function(x) {
  return x.toUpperCase();
};
let exclaim = function(x) {
  return x + '!';
};
let shout = compose(exclaim, toUpperCase);

shout("I love mondo"); //=> "I LOVE MONDO!"
```

![](http://www.rabbitonweb.com/wp-content/uploads/2017/03/Learning-FP-Concepts-for-the-Greater-Good.jpg)

You can implement a `compose` by many ways:

```javascript
function composeTwo(f, g) {
    return (...args) => f(g(...args));
}

function compose(...fns) {
  return function composed(result) {
    let list = fns.slice();
    
    while (list.length > 0) {
        result = list.pop()(result);
    }
    
    return result;
  }
}
```

### Pure Function

**Pure function** - A function that, given the same input, will always return the same output and does not have any observable side effect.

Think about `slice` and `splice`: the latter change the origin array, it's impure.

```javascript
var xs = [1, 2, 3, 4, 5];

// pure
xs.slice(0, 3);
//=> [1, 2, 3]

xs.slice(0, 3);
//=> [1, 2, 3]

xs.slice(0, 3);
//=> [1, 2, 3]


// impure
xs.splice(0, 3);
//=> [1, 2, 3]

xs.splice(0, 3);
//=> [4, 5]

xs.splice(0, 3);
//=> []
```

Side effects may include, but are not limited to

- changing the file system
- inserting a record into a database
- making an http call
- mutations
- printing to the screen / logging
- obtaining user input
- querying the DOM
- **accessing system state** - free variables, etc.

A pure function can't produce any observable effect.

### Immutability

- Don't ever make a change to value.
- Always create a new copy.
- Helpful for reducing side effects.

We can use lenses to "make change":

```javascript
let xLens = R.lens(R.prop('x'), R.assoc('x'));
//----------------- ↑getter ------ ↑setter ---

R.view(xLens, {x: 1, y: 2});            //=> 1
R.set(xLens, 4, {x: 1, y: 2});          //=> {x: 4, y: 2}
R.over(xLens, R.negate, {x: 1, y: 2});  //=> {x: -1, y: 2}
```

## Reference

[«Mostly Adequate Guide»](http://randycoulman.com/blog/categories/thinking-in-ramda/)

[Why Ramda](http://fr.umio.us/why-ramda/)

[Favoring Curry](http://fr.umio.us/favoring-curry/)

[«Thinking in Ramda»](http://randycoulman.com/blog/categories/thinking-in-ramda/)

[«Functional Light JS»](http://randycoulman.com/blog/categories/thinking-in-ramda/)

[Ramda.js](http://ramdajs.com/)
