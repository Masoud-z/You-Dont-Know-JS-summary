# You Don't Know JS : Types & Grammar <ins>**_Summary_**</ins> - 1st Edition

<br>

# Chapter 2: Values

<br><br>

## Arrays

As compared to other type-enforced languages, JavaScript arrays are just containers for any type of value, from string to number to object to even another array (which is how you get multidimensional arrays):

```js
var a = [1, "2", [3]];
a.length; // 3
a[0] === 1; // true
a[2][0] === 3; // true
```

You don’t need to presize your arrays, you can just declare them and add values as you see fit:

```js
var a = [];
a.length; // 0
a[0] = 1;
a[1] = "2";
a[2] = [3];
a.length; // 3
```

- Using `delete` on an `array` value will remove that slot from the `array`, but even if you remove the final element, it does not update the length property, so be careful!

Be careful about creating “sparse” arrays (leaving or creating empty/missing slots):

```js
var a = [];
a[0] = 1;
// no `a[1]` slot set here
a[2] = [3];
a[1]; // undefined
a.length; // 3
```

While that works, it can lead to some confusing behavior with the “empty slots” you leave in between. While the slot appears to have the `undefined` value in it, it will not behave the same as if the slot is explicitly set (`a[1] = undefined`).

`arrays` are numerically indexed (as you’d expect), but the tricky thing is that they also are `objects` that can have string keys/properties added to them (but which don’t count toward the length of the array):

```js
var a = [];
a[0] = 1;
a["foobar"] = 2;
a.length; // 1
a["foobar"]; // 2
a.foobar; // 2
```

However, a gotcha to be aware of is that if a `string` value intended as a key can be coerced to a standard base-10 number, then it is assumed that you wanted to use it as a `number` index rather than as a `string` key!

```js
var a = [];
a["13"] = 42;
a.length; // 14
```

<br>

### Array-Likes

There will be occasions where you need to convert an array-like value (a numerically indexed collection of values) into a true array, usually so you can call array utilities (like `indexOf(..)`, `concat(..)`, `forEach(..)`, etc.) against the collection of values.

For example, various DOM query operations return lists of DOM elements that are not true arrays but are array-like enough for our conversion purposes.

Another common example is when functions expose the arguments (array-like) object (as of ES6, deprecated) to access the arguments as a list.

One very common way to make such a conversion is to borrow the `slice(..)` utility against the value:

```js
function foo() {
  var arr = Array.prototype.slice.call(arguments);
  arr.push("bam");
  console.log(arr);
}
foo("bar", "baz"); // ["bar","baz","bam"]
```

As of ES6, there’s also a built-in utility called **`Array.from(..)`** that can do the same task:

```js
var arr = Array.from(arguments);
```

<br><br>

## Strings

It’s important to realize that JavaScript strings are really not the same as arrays of characters.

Strings do have a shallow resemblance to arrays—they are arraylikes. For instance, both of them have a `length` property, an `indexOf(..) `method (array version only as of ES5), and a `concat(..)` method.

- JavaScript strings are immutable, while arrays are quite mutable.

A further consequence of immutable strings is that none of the string methods that alter its contents can modify in-place, but rather must create and return new strings. By contrast, many of the array methods that change array contents actually _do_ modify inplace.

Also, many of the array methods that could be helpful when dealing with strings are not actually available for them, but we can “borrow” nonmutation array methods against our string:

```js
a.join; // undefined
a.map; // undefined

var c = Array.prototype.join.call(a, "-");
var d = Array.prototype.map
  .call(a, function (v) {
    return v.toUpperCase() + ".";
  })
  .join("");
c; // "f-o-o"
d; // "F.O.O."
```

- Unfortunately, this “borrowing” doesn’t work with array mutators, because strings are immutable and thus can’t be modified in place

- Another workaround (aka hack) is to convert the string into an array, perform the desired operation, then convert it back to a string

- Be careful! This approach doesn’t work for strings with complex (unicode) characters in them

<br><br>

## Numbers

### Numeric Syntax

the trailing portion (the fractional) of a decimal value after the ., if 0, is optional:

```js
var a = 42.0;
var b = 42;
```

Very large or very small `numbers` will by default be outputted in exponent form, the same as the output of the `toExponential()` method, like:

```js
var a = 5e10;
a; // 50000000000
a.toExponential(); // "5e+10"
var b = a * a;
b; // 2.5e+21
var c = 1 / a;
c; // 2e-11
```

Because `number` values can be boxed with the Number object wrapper, number values can access methods that are built into the `Number.prototype`.

you can access these methods directly on number literals. But you have to be careful with the `.` operator. Since `.` is a valid numeric character, it will first be interpreted as part of the number literal, if possible, instead of being interpreted as a property accessor:

```js
// invalid syntax:
42.toFixed( 3 ); // SyntaxError
// these are all valid:
(42).toFixed( 3 ); // "42.000"
0.42.toFixed( 3 ); // "0.420"
42..toFixed( 3 ); // "42.000"
```

`42..toFixed(3)` works because the first `.` is part of the number and the second `.` is the property operator.

This is also technically valid (notice the space):

```js
// prettier-ignore
42 .toFixed(3); // "42.000"
```

numbers can also be specified in exponent form, which is common when representing larger numbers, such as:

```js
var onethousand = 1e3; // means 1 * 10^3
var onemilliononehundredthousand = 1.1e6; // means 1.1 * 10^6
```

number literals can also be expressed in other bases, like binary, octal, and hexadecimal.

<br>

### Small Decimal Values

The most (in)famous side effect of using binary floating-point numbers (which, remember, is true of all languages that use IEEE 754— not just JavaScript as many assume/pretend) is:

```js
0.1 + 0.2 === 0.3; // false
```

Simply put, the representations for 0.1 and 0.2 in binary floating point are not exact, so when they are added, the result is not exactly 0.3. It’s really close, 0.30000000000000004, but if your comparison fails, “close” is irrelevant.

As of ES6, `Number.EPSILON` is predefined with this tolerance value, so you’d want to use it, but you can safely polyfill the definition for pre-ES6:

```js
if (!Number.EPSILON) {
  Number.EPSILON = Math.pow(2, -52);
}
```

We can use this Number.EPSILON to compare two numbers for “equality” (within the rounding error tolerance):

```js
function numbersCloseEnoughToEqual(n1, n2) {
  return Math.abs(n1 - n2) < Number.EPSILON;
}
var a = 0.1 + 0.2;
var b = 0.3;

numbersCloseEnoughToEqual(a, b); // true
numbersCloseEnoughToEqual(0.0000001, 0.0000002); // false
```

The maximum floating-point value that can be represented is roughly 1.798e+308 (which is really, really, really huge!), predefined for you as `Number.MAX_VALUE`. On the small end, `Number.MIN_VALUE` is roughly 5e-324, which isn’t negative but is really close to zero!

<br>

### Safe Integer Ranges

The maximum integer that can “safely” be represented (that is, there’s a guarantee that the requested value is actually representable unambiguously) is 2^53 - 1, which is 9007199254740991. If you insert your commas, you’ll see that this is just over 9 quadrillion. So that’s pretty darn big for numbers to range up to.

This value is actually automatically predefined in ES6, as `Number.MAX_SAFE_INTEGER`. Unsurprisingly, there’s a minimum value, -9007199254740991, and it’s defined in ES6 as `Number.MIN_SAFE_INTEGER`.

<br>

### Testing for Integers

To test if a value is an integer, you can use the ES6-specified `Number.isInteger(..)`:

```js
Number.isInteger(42); // true
Number.isInteger(42.0); // true
Number.isInteger(42.3); // false
```

To test if a value is a safe integer, use the ES6-specified `Number.isSafeInteger(..)`:

```js
Number.isSafeInteger(Number.MAX_SAFE_INTEGER); // true
Number.isSafeInteger(Math.pow(2, 53)); // false
Number.isSafeInteger(Math.pow(2, 53) - 1); // true
```

<br>

### 32-Bit (Signed) Integers

While integers can range up to roughly 9 quadrillion safely (53 bits), there are some numeric operations (like the bitwise operators) that are only defined for 32-bit numbers, so the “safe range” for numbers used in that way must be much smaller.

The range then is Math.pow(-2,31) (-2147483648, about –2.1 billion) up to Math.pow(2,31)-1 (2147483647, about +2.1 billion).

<br><br>

## Special Values

### The Nonvalue Values

For the `undefined` type, there is one and only one value: `undefined`. For the `null` type, there is one and only one value: `null`. So for both of them, the label is both its type and its value

Regardless of how you choose to “define” and use these two values, `null` is a special keyword, not an identifier, and thus you cannot treat it as a variable to assign to (why would you!?). However, `undefined` is (unfortunately) an identifier.

<br>

### Undefined

In non-strict mode, it’s actually possible (though incredibly illadvised!) to assign a value to the globally provided undefined identifier:

```js
function foo() {
  undefined = 2; // really bad idea!
}
foo();
function foo() {
  "use strict";
  undefined = 2; // TypeError!
}
```

In both non-strict mode and strict mode, however, you can create a local variable of the name undefined. But again, this is a terrible idea!

```js
function foo() {
  "use strict";
  var undefined = 2;
  console.log(undefined); // 2
}
foo();
```

<br>

#### void operator

While `undefined` is a built-in identifier that holds (unless modified) the built-in `undefined` value, another way to get this value is the `void` operator.

The expression `void ___` “voids” out any value, so that the result of the expression is always the undefined value. It doesn’t modify the existing value; it just ensures that no value comes back from the operator expression:

```js
var a = 42;
console.log(void a, a); // undefined 42
```

<br>

### Special Numbers

#### The not number, number

Any mathematic operation you perform without both operands being numbers will result in the operation failing to produce a valid number, in which case you will get the `NaN` value.

**the type of not-a-number is number!**

```js
var a = 2 / "foo"; // NaN
typeof a === "number"; // true
```

`NaN` is a very special value in that it’s never equal to another `NaN` value (i.e., it’s never equal to itself).

```js
var a = 2 / "foo";
a == NaN; // false
a === NaN; // false
```

We use the built-in global utility called `isNaN(..)` and it tells us if the value is `NaN` or not.

The `isNaN(..)` utility has a fatal flaw. It appears it tried to take the meaning of `NaN` (“Not a Number”) too literally—that its job is basically, “test if the thing passed in is either not a number or is a number.” But that’s not quite accurate:

```js
var a = 2 / "foo";
var b = "foo";
a; // NaN
b;
("foo");
window.isNaN(a); // true
window.isNaN(b); // true--ouch!
```

As of ES6, finally a replacement utility has been provided: `Number.isNaN(..)`. A simple polyfill for it so that you can safely check `NaN` values now even in pre-ES6 browsers is:

```js
if (!Number.isNaN) {
  Number.isNaN = function (n) {
    return n !== n;
  };
}
var a = 2 / "foo";
var b = "foo";
Number.isNaN(a); // true
Number.isNaN(b); // false--phew!
```

<br>

#### Infinities

However, in JS, this operation is well-defined and results in the value `Infinity` (aka `Number.POSITIVE_INFINITY`). Unsurprisingly:

```js
var a = 1 / 0; // Infinity
var b = -1 / 0; // -Infinity
```

JS uses finite numeric representations (IEEE 754 floating-point, which we covered earlier), so contrary to pure mathematics, it seems it is possible to overflow even with an operation like addition or subtraction, in which case you’d get `Infinity` or `-Infinity`. For example:

```js
var a = Number.MAX_VALUE; // 1.7976931348623157e+308
a + a; // Infinity
a + Math.pow(2, 970); // Infinity
a + Math.pow(2, 969); // 1.7976931348623157e+308
```

`Infinity / Infinity` is not a defined operation. In JS, this results in `NaN`.

But what about any positive finite number divided by `Infinity`? That’s easy! 0. And what about a negative finite number divided by `Infinity`? Keep reading!

<br>

#### Zeros

While it may confuse the mathematics-minded reader, JavaScript has both a normal zero `0` (otherwise known as a positive zero `+0`) and a negative zero `-0`.

```js
var a = 0 / -3; // -0
var b = 0 * -3; // -0
```

However, if you try to stringify a negative zero value, it will always be reported as "0", according to the spec:

```js
var a = 0 / -3;
// (some browser) consoles at least get it right
a; // -0
// but the spec insists on lying to you!
a.toString(); // "0"
a + ""; // "0"
String(a); // "0"
// strangely, even JSON gets in on the deception
JSON.stringify(a); // "0"
```

Interestingly, the reverse operations (going from string to number) don’t lie:

```js
+"-0"; // -0
Number("-0"); // -0
JSON.parse("-0"); // -0
```

In addition to stringification of negative zero being deceptive to hide its true value, the comparison operators are also (intentionally) configured to lie:

<br><br>

### Special Equality

As of ES6, there’s a new utility that can be used to test two values for absolute equality, without any of these exceptions. It’s called `Object.is(..)`

```js
var a = 2 / "foo";
var b = -3 * 0;
Object.is(a, NaN); // true
Object.is(b, -0); // true
Object.is(b, 0); // false
```

There’s a pretty simple poly fill for Object.is(..) for pre-ES6 environments:

```js
if (!Object.is) {
  Object.is = function (v1, v2) {
    // test for `-0`
    if (v1 === 0 && v2 === 0) {
      return 1 / v1 === 1 / v2;
    }

    // test for `NaN`
    if (v1 !== v1) {
      return v2 !== v2;
    }

    // everything else
    return v1 === v2;
  };
}
```

<br><br>

## Value Versus Reference
