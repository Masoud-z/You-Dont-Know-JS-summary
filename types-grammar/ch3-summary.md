# You Don't Know JS : Types & Grammar <ins>**_Summary_**</ins> - 1st Edition

<br>

# Chapter 2: Natives

<br><br>

Here’s a list of the most commonly used natives:

- `String()`
- `Number()`
- `Boolean()`
- `Array()`
- `Object()`
- `Function()`
- `RegExp()`
- `Date()`
- `Error()`
- `Symbol()`—added in ES6!

As you can see, these natives are actually built-in functions

It is true that each of these natives can be used as a native constructor. But what’s being constructed may be different than you think:

```js
var a = new String("abc");
typeof a; // "object" ... not "String"
a instanceof String; // true
Object.prototype.toString.call(a); // "[object String]"
```

The result of the constructor form of value creation (`new String("abc")`) is an object wrapper around the primitive ("abc") value.

<br><br>

## Internal `[[Class]]`

Values that are `typeof` of "object" (such as an array) are additionally tagged with an internal `[[Class]]` property (think of this more as an internal classification rather than related to classes from traditional class-oriented coding). This property cannot be accessed directly, but can generally can be revealed indirectly by borrowing
the default `Object.prototype.toString(..)` method called against the value.

```js
Object.prototype.toString.call([1, 2, 3]);
// "[object Array]"
Object.prototype.toString.call(/regex-literal/i);
// "[object RegExp]

Object.prototype.toString.call(null);
// "[object Null]"
Object.prototype.toString.call(undefined);
// "[object Undefined]"
```

You’ll note that there are no `Null()` or `Undefined()` native constructors, but nevertheless "`Null`" and "`Undefined`" are the internal `[[Class]]` values exposed.

But for the other simple primitives like string, number, and boolean, another behavior actually kicks in, which is usually called “boxing”.

```js
Object.prototype.toString.call("abc");
// "[object String]"
Object.prototype.toString.call(42);
// "[object Number]"
Object.prototype.toString.call(true);
// "[object Boolean]"
```

In this snippet, each of the simple primitives are automatically boxed by their respective object wrappers, which is why "`String`", "`Number`", and "`Boolean`" are revealed as the respective internal `[[Class]]` values.

<br><br>

## Boxing Wrappers

Primitive values don’t have properties or methods, so to access `.length` or `.toString()` you need an object wrapper around the value. Thankfully, **JS will automatically box (aka wrap) the primitive value to fulfill such accesses**.

<br>

### Object Wrapper Gotchas

There are even gotchas with using the object wrappers directly that you should be aware of if you do choose to ever use them.
For example, consider `Boolean` wrapped values:

```js
var a = new Boolean(false);
if (!a) {
  console.log("Oops"); // never runs
}
```

The problem is that you’ve created an object wrapper around the `false` value, but objects themselves are “truthy”, so using the object behaves oppositely to using the underlying `false` value itself, which is quite contrary to normal expectation.

<br><br>

## Unboxing

If you have an object wrapper and you want to get the underlying primitive value out, you can use the valueOf() method:

```js
var a = new String("abc");
var b = new Number(42);
var c = new Boolean(true);
a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```

- Unboxing can also happen implicitly, when using an object wrapper value in a way that requires the primitive value. This process is called **coercion**.

```js
var a = new String("abc");
var b = a + ""; // `b` has the unboxed primitive value "abc"
typeof a; // "object"
typeof b; // "string"
```

<br><br>

## Natives as Constructors

For `array`, `object`, `function`, and regular-expression values, it’s almost universally preferred that you use the literal form for creating the values, but the literal form creates the same sort of object as the constructor form does (that is, there is no nonwrapped value).

<br><br>

### Array(..)

```js
var a = new Array(1, 2, 3);
a; // [1, 2, 3]
var b = [1, 2, 3];
b; // [1, 2, 3]
```

The `Array(..)` constructor does not require the new keyword in front of it. If you omit it, it will behave as if you have used it anyway. So `Array(1,2,3)` is the same outcome as `new Array(1,2,3)`.

The Array constructor has a special form where if only one number argument is passed, instead of providing that value as contents of the array, it’s taken as a length to “presize the array”

- An array with at least one “empty slot” in it is often called a “sparse array.”

```js
var a = new Array(3);
a.length; // 3
a;
```

```js
var a = new Array(3);
var b = [undefined, undefined, undefined];
var c = [];
c.length = 3;
a; // [empty × 3]
b; // [undefined, undefined, undefined]
c; // [empty × 3]
```

As you can see with `c` in this example, empty slots in an array can happen after creation of the array. When changing the length of an array to go beyond its number of actually defined slot values, you implicitly introduce empty slots. In fact, you could even call `delete b[1]` in the above snippet, and it would introduce an empty slot into the middle of `b`
