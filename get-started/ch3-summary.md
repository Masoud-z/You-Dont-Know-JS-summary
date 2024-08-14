# You Don't Know JS Yet: Get Started **_Summary_** - 2nd Edition

<br>

# Chapter 3: Digging to the Roots of JS

## What is spread ?

To spread an iterator, you have to have something to spread it into. There are two possibilities in JS: an **array** or **an argument list for a function call**.

<br>
An array spread:

```js
// spread an iterator into an array,
// with each iterated value occupying
// an array element position.
var vals = [...it];
```

<br>

A function call spread:

```js
// spread an iterator into a function,
// call with each iterated value
// occupying an argument position.
doSomethingUseful(...it);
```

In both cases, the iterator-spread form of ... follows the iterator-consumption protocol (the same as the for..of loop) to retrieve all available values from an iterator and place (aka, spread) them into the receiving context (array, argument list)

- For the most part, all built-in iterables in JS have three iterator forms available: **keys-only (`keys()`)**, **values-only (`values()`)**, and **entries (`entries()`)**.

<br>

## What is closure?

Closure is when a function remembers and continues to access variables from outside its scope, even when the function is executed in a different scope.

- It’s not necessary that the outer scope be a function—it usually is, but not always—just that there be at least one variable in an outer scope accessed from an inner function:

- when a function is defined, it is attached to its enclosing scope via closure. Scope is the set of rules that controls how references to variables are resolved.

<br>

## this Keyword

functions also have another characteristic besides their scope that influences what they can access. This characteristic is best described as an **execution context**, and it’s exposed to the function via its `this` keyword.

Scope is static and contains a fixed set of variables available at the moment and location you define a function, but a **function’s execution context is dynamic, entirely dependent on how it is called** (regardless of where it is defined or even called from).

`this` is not a fixed characteristic of a function based on the function’s definition, but rather a dynamic characteristic that’s determined each time the function is called.

When a function does reference `this`, makes it a **this-aware function**. In other words, **it’s a function that is dependent on its execution context**.

```js
function classroom(teacher) {
  return function study() {
    console.log(`${teacher} says to study ${this.topic}`);
  };
}
var assignment = classroom("Kyle");

assignment();
// Kyle says to study undefined -- Oops :(
```

In this snippet, we call `assignment()` as a plain, normal function, without providing it any execution context.

Since this program is not in strict mode, context-aware functions that are called without any context specified default the context to the global object (`window` in the browser). As there is no global variable named topic (and thus no such property on the global object), `this.topic` resolves to `undefined`.

2:

```js
var homework = {
  topic: "JS",
  assignment: assignment,
};
homework.assignment();
// Kyle says to study JS
```

A copy of the `assignment` function reference is set as a property on the homework object, and then it’s called as `homework.assignment()`. That means the `this` for that function call will be the `homework` object. Hence, t`his.topic` resolves to `"JS"`.

3:

```js
var otherHomework = {
  topic: "Math",
};
assignment.call(otherHomework);
// Kyle says to study Math
```

A third way to invoke a function is with the `call(..)` method, which takes an object (`otherHomework` here) to use for setting the `this` reference for the function call. The property reference `this.topic` resolves to `"Math"`.

- The benefit of `this`-aware functions —and their dynamic context— is the ability to more flexibly re-use a single function with data from different objects. A function that closes over a scope can never reference a different scope or set of variables. But a function that has dynamic `this` context awareness can be quite helpful for certain tasks.

<br>

## Prototypes

a prototype is a characteristic of an object, and specifically resolution of a property access.
Think about a prototype as a linkage between two objects; the linkage is hidden behind the scenes

This prototype linkage occurs when
an object is created; it’s linked to another object that already exists.

A series of objects linked together via prototypes is called the “**prototype chain**”.

## Object Linkage

To define an object prototype linkage, you can create the object using the `Object.create(..)` utility:

```js
var homework = {
  topic: "JS",
};
var otherHomework = Object.create(homework);
otherHomework.topic; // "JS"
```

- `Object.create(null)` creates an object that is not prototype linked anywhere, so it’s purely just a standalone object; in some circumstances, that may be preferable.

<br>

## `this` Revisited

```js
var homework = {
  study() {
    console.log(`Please study ${this.topic}`);
  },
};
var jsHomework = Object.create(homework);
jsHomework.topic = "JS";
jsHomework.study();
// Please study JS
var mathHomework = Object.create(homework);
mathHomework.topic = "Math";
mathHomework.study();
// Please study Math
```
