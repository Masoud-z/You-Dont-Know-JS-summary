# You Don't Know JS : Types & Grammar <ins>**_Summary_**</ins> - 1st Edition

<br>

# Chapter 1: Types

<br><br>

The ECMAScript language types are **Undefined**, **Null**, **Boolean**, **String**, **Number**, and **Object**.

<br>

## A Type by Any Other Name…

Beyond academic definition disagreements, why does it matter if JavaScript has types or not?

Having a proper understanding of each type and its intrinsic behavior is absolutely essential to understanding how to properly and accurately convert values to different types (see Chapter 4). Nearly every JS program ever written will need to handle value coercion in some shape or form, so it’s important you do so responsibly and
with confidence.

<br><br>

## Built-in Types

JavaScript defines seven built-in types:

1. null
2. undefined
3. boolean
4. number
5. string
6. object
7. symbol—added in ES6!

- All of these types except object are called “**primitives**”.

The `typeof` operator inspects the type of the given value, and always returns one of seven string values.

```js
typeof undefined === "undefined"; // true
typeof true === "boolean"; // true
typeof 42 === "number"; // true
typeof "42" === "string"; // true
typeof { life: 42 } === "object"; // true

// added in ES6!
typeof Symbol() === "symbol"; // true

typeof function a() {
  /* .. */
} === "function"; // true
```

- `null` is special—special in the sense that it’s buggy when combined with the `typeof` operator:

```js
var a = null;
!a && typeof a === "object"; // true
```

It would have been nice (and correct!) if it returned "`null`", but this original bug in JS has persisted for nearly two decades, and will likely never be fixed because there’s so much existing web content that relies on its buggy behavior that “fixing” the bug would create more “bugs” and break a lot of web software

likely never be fixed because there’s so much existing web content that relies on its buggy behavior that “fixing” the bug would create more “bugs” and break a lot of web software

```js
var a = null;
!a && typeof a === "object"; // true
```

`null` is the only primitive value that is “falsy” (aka false-like; see Chapter 4) but which also returns "object" from the typeof check.

<br>

So what’s the seventh string value that typeof can return?

```js
typeof function a() {
  /* .. */
} === "function"; // true
```

It’s easy to think that function would be a top-level built-in type in JS, especially given this behavior of the `typeof` operator. However, if you read the spec, you’ll see **it’s actually somewhat of a “subtype” of object. Specifically, a function is referred to as a “callable object”— an object that has an internal `[[Call]]` property that allows it to be invoked**. The fact that functions are actually objects is quite useful. **Most importantly, they can have properties**.

<br>

What about arrays? They’re native to JS, so are they a special type?

```js
typeof [1, 2, 3] === "object"; // true
```

It’s most appropriate to think of Arrays also as a “subtype” of object

<br><br>

## Values as Types

In JavaScript, **variables don’t have types** — **values have types**. **Variables can hold any value, at any time**.

JS doesn’t have “type enforcement,” in that the engine doesn’t insist that a _variable_ always holds values of the _same initial type_ that it starts out with.

If you use `typeof` against a variable, it’s not asking “What’s the type of the variable?” as it may seem, since JS variables have no types. Instead, it’s asking “What’s the type of the value in the variable?”

<br>

### undefined Versus “undeclared”

Variables that have no value currently actually have the undefined value. Calling `typeof` against such variables will return "`undefined`":

```js
var a;
typeof a; // "undefined"

var b = 42;
var c;
// later
b = c;
typeof b; // "undefined"
typeof c; // "undefined"
```

An “`undefined`” variable is one that has been declared in the accessible scope, but at the moment has no other value in it. By contrast, an “**_undeclared_**” variable is one that has not been formally declared in the accessible scope.

Consider:

```js
var a;
a; // undefined
b; // ReferenceError: b is not defined
```

- “`undefined`” and “**_is not defined_**” are very different things.

There’s also a special behavior associated with `typeof` as it relates to undeclared variables that even further reinforces the confusion. Consider:

```js
var a;
typeof a; // "undefined"
typeof b; // "undefined"
```

The `typeof` operator returns "`undefined`" even for “**_undeclared_**” (or “**_not defined”_**) variables. Notice that there was no error thrown when we executed `typeof b`, even though `b` is an undeclared variable. This is a special safety guard in the behavior of typeof.

<br>

### typeof Undeclared

Nevertheless, this safety guard is a useful feature when dealing with JavaScript in the browser, where multiple script files can load variables into the shared global namespace.

As a simple example, imagine having a “debug mode” in your program that is controlled by a global variable (flag) called `DEBUG`. You’d want to check if that variable was declared before performing a debug task like logging a message to the console. A top-level global `var DEBUG = true` declaration would only be included in a “debug.js” file, which you only load into the browser when you’re in development/testing, but not in production.

However, you have to take care in how you check for the global `DEBUG` variable in the rest of your application code, so that you don’t throw a `ReferenceError`. The safety guard on `typeof` is our friend in this case:

```js
// oops, this would throw an error!
if (DEBUG) {
  console.log("Debugging is starting");
}
// this is a safe existence check
if (typeof DEBUG !== "undefined") {
  console.log("Debugging is starting");
}
```

This sort of check is useful even if you’re not dealing with user defined variables (like `DEBUG`). If you are doing a feature check for a built-in API, you may also find it helpful to check without throwing an error:

```js
if (typeof atob === "undefined") {
  atob = function () {
    /*..*/
  };
}
```

Unlike referencing undeclared variables, there is no ReferenceError thrown if you try to access an object property (even on the global window object) that doesn’t exist.

<br><br>

## Review

JavaScript has seven built-in types: `null`, `undefined`, `boolean`, `number`, `string`, `object`, and `symbol`. They can be identified by the `typeof` operator.

**Variables don’t have types**, **_but the values in them do_**. These types define the intrinsic behavior of the values.

Many developers will assume “undefined” and “undeclared” are roughly the same thing, but in JavaScript, they’re quite different.
`undefined` is a value that a declared variable can hold. “Undeclared” means a variable has never been declared.

JavaScript unfortunately kind of conflates these two terms, not only in its error messages (“ReferenceError: a is not defined”) but also in the return values of `typeof`, which is "`undefined`" for both cases
