---
layout: post
title:  "Understanding `this` in JavaScript"
date:   2016-12-21
categories: javascript
---

What better topic to start the blog with. Let's look at how `this` works in JavaScript.

## Level 0: The Basics

At the root scope, `this` refers to the global object or `undefined` if in strict mode. In browsers, the global object is `window`.

```javascript
console.log(this);
// logs the `window` object

(function(){
  'use strict';
  console.log(this);
})()
// logs `undefined`
```


> Level 0 🔑
>
> Within a function, what `this` refers to depends on how the function is invoked, NOT how it's defined.

You should also keep in mind that you can't directly set the value of `this` from within a function.

```javascript
function foo(){
  this = 'trying to set this!';
}
foo(); // Uncaught ReferenceError: Invalid left-hand side in assignment
```

We'll be working with the following function for the remainder of this article.

```javascript
function printThis(){
  console.log(this);
}
```

By mere inspection, I couldn't tell you what this will log. I need to know  _how_ the function will be invoked.

And this leads us to ...

## Level 1: The Four Invocation Patterns

There are four ways to execute a function in JavaScript

### The function call
This is the regular way of calling a function

```javascript
printThis()
```

As mentioned above, the output of our function depends on whether we're in strict mode or not. If not in strict mode, `this` will refer to the global object, `window` in browsers. If we are in strict mode, then `this` will be undefined.


### The method call
Suppose we instead had an object like this:

```javascript
var obj = {
  print: printThis
}
```

And we invoked the function like this `obj.print()`.

Our output will be the object `obj`. In this case, `this` refers to the object the method is attached to. This rule is often summarized as "that which is to the left of the dot".

Note that the following is _not_ the same thing:

```javascript
var printB = obj.print;
printB();
```

Remember Level 0, `this` depends on _how_ the function is called. The above code puts us back into the regular function call case, which will then log the global object or undefined.

### The constructor

Suppose we had the following:

```javascript
function Obj(){
  this.x = 'foo';
}

var object = new Obj();
```

We've invoked the `Obj` constructor function using the `new` keyword. In this case, `this` now refers to the newly created object. This object's prototype is equal to `Obj.prototype`.

### Using `apply/call`

We can change the value of `this` to anything we like using `call` or `apply`. The first argument to either method is the value that `this` will take within the function.

A useful mnemonic is that `apply` accepts an array as it's second argument (they both start with _a_).

`call` accepts each argument separately.

```javascript
var obj = {prop: 'a property'});
printThis.call(obj)
// logs `obj`
```

This really drives home the point that `this` can be anything since the caller can use `call` or `apply` to change it to anything they desire.

Many libraries tend to set the value of `this` to something useful when you pass a callback to a library function. For example, in jQuery, `this` will often be set to the DOM element where a bound event is taking place:

```javascript
$('#foo').click(function(){
  console.log(this);
  // logs the `#foo` element
})
```

### Using `bind`

We can use the native `bind` method to "hard bind" the value of `this`. No matter how this bound function is invoked, the value of `this` will stay the same.

```javascript
var obj {
  foo: function(){console.log(this)}
}

var method = obj.foo;
var boundMethod = obj.foo.bind(obj);

obj.foo();
// logs `obj`

method();
// logs `window` object

boundMethod();
// logs `obj`
```

One caveat is that when using a bound function as a constructor, the bound value of `this` is ignored.

```javascript
function MyObj {
  this.x = 'foo';
}

var o = {y: 'hello'}

var boundMyObj = MyObj.bind(o);
boundMyObj();
console.log(o);
// logs {x: 'foo', y: 'hello'}

var bar = new boundMyObj();
console.log(bar);
// logs {x: 'foo'}
```

## Level 2: Arrow Functions

In addition to being a more concise syntax, arrow functions also differ in how `this` is bound. The value of `this` within an arrow function body is equal to whatever `this` was in the surrounding scope ("lexical `this`").

```javascript
function foo() {
  this.x = 'hello';
  [1,2,3].forEach(()=>{
    console.log(this.x);
  });
}

new foo();
```

The above snippet logs `hello` 3 times. If we were to use a non-arrow function, we get `undefined` logged 3 times. This is because the value of `this` within the `forEach` callback would refer to `window` rather than our newly created `foo` object.

## Level 3: Advanced

Levels 0 through 2 will probably account for 99% of the cases you'll encounter in real life. But there are some esoteric edge cases that aren't covered.

This section is based on the material found in [this fantastic blog post](http://perfectionkills.com/know-thy-reference/).

Consider the following lines, what would you expect the value of `this` to be?

```javascript
var foo = {bar: function(){console.log(this)}};

(1, foo.bar)();
(g = foo.bar)();
(foo.bar)();
```

The first two lines log the `window` object but the last line logs `foo`. It's a bit surprising isn't it?

To understand what's happening, we have to understand _references_.

A reference is essentially an internal data structure that the JavaScript compiler uses to "reference" a function. It can be thought of as an abstract object that looks like this:

```javascript
var Reference = {
  base: EnvironmentRecord, // determines the value that `this` will take
  name: "foo", // the function name
  strict: false // whether we're in strict mode
};
```
So a reference isn't an actual function.

The comma operator and simple assignment _don't_ create references. This means that the first two examples above are more like doing `(function(){console.log(this)})()`.

The grouping operator _does_ create a reference. That's why we still log `foo` in the last example.

If you want to learn more about references, check out [this blog post](http://perfectionkills.com/know-thy-reference/).

