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

