# You Don't Know JS : Types & Grammar <ins>**_Summary_**</ins> - 1st Edition

<br>

# Chapter 4: Coercion

<br><br>

## Converting Values

Converting a value from one type to another is often called “type casting” when done explicitly, and “coercion” when done implicitly (forced by the rules of how a value is used).

- It may not be obvious, but JavaScript coercions always result in one of the scalar primitive values, like `string`, `number`, or `boolean`. There is no coercion that results in a complex value like object or function.

However, in JavaScript, most people refer to all these types of conversions as coercion, so the way I prefer to distinguish is to say “implicit coercion” versus “explicit coercion.”

The difference should be obvious: “explicit coercion” is when it is obvious from looking at the code that a type conversion is intentionally occurring, whereas “implicit coercion” is when the type conversion will occur as a less obvious side effect of some other intentional operation.

```js
var a = 42;
var b = a + ""; // implicit coercion
var c = String(a); // explicit coercion
```

<br><br>

## Abstract Value Operations

<br>

### ToString

For regular objects, unless you specify your own, the default `toString()` (located in `Object.prototype.toString()`) will return the internal `[[Class]]` , like for instance "`[object Object]`".

Arrays have an overridden default `toString()` that stringifies as the (string) concatenation of all its values (each stringified themselves), with "," in between each value:

```js
var a = [1, 2, 3];
a.toString(); // "1,2,3"
```

<br>

#### JSON stringification

It may be easier to consider values that are not JSON-safe. Some examples are `undefined`, functions, (ES6+) `symbols`, and objects with circular references (where property references in an object structure create a never-ending cycle through each other). These are all illegal values for a standard JSON structure, mostly because they aren’t portable to other languages that consume JSON values

- The `JSON.stringify(..)` utility will automatically omit `undefined`, function, and symbol values when it comes across them.

If such a value is found in an array, that value is replaced by `null` (so that the array position information isn’t altered). If found as a property of an object, that property will simply be excluded.

```js
JSON.stringify(undefined); // undefined
JSON.stringify(function () {}); // undefined
JSON.stringify([1, undefined, function () {}, 4]); // "[1,null,null,4]"

JSON.stringify({ a: 2, b: function () {} }); // "{"a":2}"
```

<br>

- But if you try to `JSON.stringify(..)` an object with circular reference(s) in it, an error will be thrown.

  <br>

JSON stringification has the special behavior that if an object value has a `toJSON()` method defined, this method will be called first to get a value to use for serialization.

```js
var o = {};
var a = {
  b: 42,
  c: o,
  d: function () {},
};

// create a circular reference inside `a`
o.e = a;
// would throw an error on the circular reference
// JSON.stringify( a );

// define a custom JSON value serialization
a.toJSON = function () {
  // only include the `b` property for serialization
  return { b: this.b };
};
JSON.stringify(a); // "{"b":42}"
```

<br>

- An optional second argument can be passed to `JSON.stringify(..)` that is called **replacer**.
  This argument can either be an array or a function. It’s used to customize the recursive serialization of an object by providing a filtering mechanism for which properties should and should not be included,

If _replacer_ is an array, it should be an array of strings, each of which will specify a property name that is allowed to be included in the serialization of the object. If a property exists that isn’t in this list, it will be skipped.

If _replacer_ is a _function_, it will be called once for the object itself, and then once for each property in the object, and each time is passed two arguments, _key_ and _value_. To skip a _key_ in the serialization, return `undefined`. Otherwise, return the _value_ provided.

```js
var a = {
  b: 42,
  c: "42",
  d: [1, 2, 3],
};

JSON.stringify(a, ["b", "c"]); // "{"b":42,"c":"42"}"

JSON.stringify(a, function (k, v) {
  if (k !== "c") return v;
});
// "{"b":42,"d":[1,2,3]}"
```

In the function replacer case, the key argument `k` is `undefined` for the first call (where the a object itself is being passed in). The if statement filters out the property named "`c`". Stringification is recursive, so the `[1,2,3]` array has each of its values (1, 2, and 3) passed as `v` to replacer, with indexes (0, 1, and 2) as `k`.

<br>

- A third optional argument can also be passed to `JSON.stringify(..)`, called space, which is used as indentation for prettier human-friendly output.

space can be a positive integer to indicate how many `space` characters should be used at each indentation level. Or, space can be a string, in which case up to the first 10 characters of its value will be used for each indentation level:

```js
var a = {
  b: 42,
  c: "42",
  d: [1, 2, 3],
};

JSON.stringify(a, null, 3);
// "{
//    "b": 42,
//    "c": "42",
//    "d": [
//        1,
//        2,
//        3
//     ]
// }"

JSON.stringify(a, null, "-----");
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"
```

<br><br>

## ToNumber

If any non-number value is used in a way that requires it to be a number, such as a mathematical operation, the ES5 spec defines the `ToNumber` abstract operation.

For example, `true` becomes `1` and `false` becomes `0`. `undefined` becomes `NaN`, but (curiously) `null` becomes `0`.

`ToNumber` for a string value essentially works for the most part like the rules/syntax for numeric literals. If it fails, the result is `NaN` (instead of a syntax error as with number literals).

As of ES5, you can create such a noncoercible object—one without `valueOf()` and `toString()`—if it has a `null` value for its `[[Prototype]]`, typically created with `Object.create(null)`.

```js
var a = {
  valueOf: function () {
    return "42";
  },
};

Number(a); // 42

var b = {
  toString: function () {
    return "42";
  },
};

Number(b); // 42

var c = [4, 2];
c.toString = function () {
  return this.join(""); // "42"
};

Number(c); // 42
Number(""); // 0
Number([]); // 0
Number(["abc"]); // NaN
```

<br><br>

## ToBoolean

- You can coerce `1` to `true` (and vice versa) or `0` to `false` (and vice versa). **But they’re not the same**.

<br>

#### Falsy values

All of JavaScript’s values can be divided into two categories:

1. Values that will become `false` if coerced to `boolean`
2. Everything else (which will obviously become `true`)

<br>

The so-called “falsy” values list:

- `undefined`
- `null`
- `false`
- `+0`, `-0`, and `NaN`
- `""` (Empty string)

#### Falsy objects

There are certain cases where browsers have created their own sort of exotic values behavior, namely this idea of “falsy objects,” on top of regular JS semantics.

A “falsy object” is a value that looks and acts like a normal object (properties, etc.), but when you coerce it to a boolean, it coerces to a false value.

The most well-known case is `document.all`, an array-like (object) provided to your JS program by the DOM (not the JS engine itself), which exposes elements in your page to your JS program. It used to behave like a normal object—it would act truthy. **But not anymore**.

<br>

#### Truthy values

- The string values themselves are all truthy, because "" is the only string value on the falsy list.

```js
var a = []; // empty array--truthy
Boolean(a); //true

var b = {}; // empty object--truthy
Boolean(b); //true

var c = function () {}; // empty function--truthy
Boolean(c); //true
```

<br><br>

## Explicit Coercion

Explicit coercion refers to type conversions that are obvious and explicit.

<br>

### Explicitly: Strings <--> Numbers

To coerce between `strings` and `numbers`, we use the built-in `String(..)` and `Number(..)` functions (which we referred to as “native constructors”), but very importantly, we do not use the `new` keyword in front of them. As such, we’re not creating object wrappers.

`String(..)` coerces from any other value to a primitive `string` value, using the rules of the `ToString` operation discussed earlier. `Number(..)` coerces from any other value to a primitive `number` value, using the rules of the `ToNumber` operation discussed earlier.

Besides` String(..)` and `Number(..)`, there are other ways to “explicitly” convert these values between string and number:

```js
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

b; // "42"
d; // 3.14
```

The unary `-` operator also coerces like `+` does, but it **also flips the sign of the number**. However, you cannot put two (--) next to each other to unflip the sign, as **that’s parsed as the decrement operator**. Instead, you would need to do `- -"3.14"` **with a space in between**, and that would result in coercion to `3.14`.

- You should strongly consider avoiding unary `+` (or `-`) coercion when it’s immediately adjacent to other operators.

Another extremely confusing place for unary `+` to be used adjacent to another operator would be the `++` increment operator and `--` decrement operator. For example, consider `a +++b`, `a + ++b`, and `a + + +b`.

<br>

#### Date to number

Another common usage of the unary `+` operator is to coerce a `Date` object into a `number`, because the result is the Unix timestamp (milli‐seconds elapsed since 1 January 1970 00:00:00 UTC) representation of the date/time value:

```js
var d = new Date("Mon, 18 Aug 2014 08:53:06 CDT");
d; // Mon, 18 Aug 2014 08:53:06 CDT (Central Time)
+d; // 1408369986000
```

- The `()` set on a constructor call (a function called with `new`) is optional only if there are no arguments to pass. So you may run across the `var timestamp = +new Date`.

But coercion is not the only way to get the timestamp out of a Date object. A noncoercion approach is perhaps even preferable, as it’s even more explicit:

```js
var timestamp = new Date().getTime();
// var timestamp = (new Date()).getTime();
// var timestamp = (new Date).getTime();
```

But an even more preferable noncoercion option is to use the `Date.now()` static function added in ES5.

- I’d recommend skipping the coercion forms related to dates. Use `Date.now()` for current now timestamps, and `new Date( .. ).get Time()` for getting a timestamp of a specific non-now date/time that you need to specify.

<br>

#### The curious case of the ~

`~x` is roughly the same as `-(x+1)`. That’s weird, but slightly easier to reason about. So:

```js
~42; // -(42+1) ==> -43
```

<br>

#### Truncating bits

some developers use the double tilde `~~` to truncate the decimal part of a number (i.e., “coerce” it to a whole number integer).

```js
~~12.999999; // 12
```

However, `~~` needs some caution/clarification. First,` it only works reliably on 32-bit values`. But more importantly, `it doesn’t work the same on negative numbers`

```js
Math.floor(-49.6); // -50
~~-49.6; // -49
```

<br>

### Explicitly: Parsing Numeric Strings

```js
var a = "42";
var b = "42px";
Number(a); // 42
parseInt(a); // 42
Number(b); // NaN
parseInt(b); // 42
```

**Parsing a numeric value out of a string is tolerant of non-numeric characters—it just stops parsing left-to-right when encountered**— **whereas coercion is not tolerant and fails, resulting in the NaN value**.

- `parseInt(..)` has a twin, `parseFloat(..)`, which (as it sounds) pulls out a floating-point number from a string.

- Don’t forget that `parseInt(..)` operates on string values. -If you pass a non-string, the value you pass will automatically be coerced to a string first, which would clearly be a kind of hidden implicit coercion.

```js
parseInt("ASD12"); //NaN
```

Prior to ES5, another gotcha existed with `parseInt(..)`, which was the source of many JS programs’ bugs. If you didn’t pass a second argument to indicate which numeric base (aka radix) to use for interpreting the numeric string contents, `parseInt(..)` would look at the first character to make a guess.

As of ES5, `parseInt(..)` no longer guesses. Unless you say otherwise, it assumes base-10. That’s much nicer.

<br>

#### Parsing non-strings

```js
parseInt(1 / 0, 19); // 18
```

It’s essentially `parseInt( "Infinity", 19 )`. How does it parse? The first character is "I", which is value 18 in the silly base-19. The second character "n" is not in the valid set of numeric characters, and as such the parsing simply politely stops, just like when it ran across "p" in "42px". The result? 18. Exactly like it sensibly should be.

```js
parseInt(0.000008); // 0 ("0" from "0.000008")
parseInt(0.0000008); // 8 ("8" from "8e-7")
parseInt(false, 16); // 250 ("fa" from "false")
parseInt(parseInt, 16); // 15 ("f" from "function..")
parseInt("0x10"); // 16
parseInt("103", 2); // 2
```

In JavaScript, `parseInt("103", 2)` returns 2 because parseInt interprets the string "103" as a binary (base 2) number until it encounters an invalid digit, which in this case is "3"

<br><br>

### Explicitly: \* --> Boolean

Just like with `String(..)` and `Number(..)` above, `Boolean(..)` (without the `new`, of course!) is an explicit way of forcing the `ToBoolean` coercion:

```js
var a = "0";
var b = [];
var c = {};

Boolean(a); // true
Boolean(b); // true
Boolean(c); // true

var d = "";
var e = 0;
var f = null;
var g;

Boolean(d); // false
Boolean(e); // false
Boolean(f); // false
Boolean(g); // false

!!a; // true
!!b; // true
!!c; // true
!!d; // false
!!e; // false
!!f; // false
!!g; // false
```

the unary `!` negate operator explicitly coerces a value to a `boolean`. The problem is that it also flips the value from truthy to falsy or vice versa. So, the most common way JS developers explicitly coerce to `boolean` is to use the `!!` double-negate operator, because the second `!` will flip the parity back to the original.

<br>

```js
var a = 42;
var b = a ? true : false;
```

The `? :` ternary operator will test a for truthiness, and based on that
test will either assign `true` or `false` to `b`, accordingly

There’s a hidden implicit coercion, in that the a expression has to first be coerced to boolean to perform the truthiness test. I’d call this idiom “explicitly implicit.”

<br><br>

## Implicit Coercion

Implicit coercion refers to type conversions that are hidden, with nonobvious side effects that implicitly occur from other actions.

<br>

### Implicitly: Strings <--> Numbers

According to the ES5 spec, section 11.6.1, the `+` algorithm (when an object value is an operand) will concatenate if either operand is either already a string, or if the following steps produce a string representation. So, when `+` receives an object (including array) for either operand, it first calls the `ToPrimitive` abstract operation (section 9.1) on the value, which then calls the `[[DefaultValue]]` algorithm (section 8.12.8) with a context hint of number.

Consider:

```js
var a = [1, 2];
var b = [3, 4];
a + b; // "1,23,4"
```

The `array` will fail to produce a simple primitive, so it then falls to a `toString()` representation. The two arrays thus become "1,2" and "3,4", respectively. Now, `+` concatenates the two strings as you’d normally expect: "1,23,4".

If either operand to `+` is a string (or become one with the above steps!), the operation will be string concatenation. Otherwise, it’s always numeric addition.

Comparing this implicit coercion of `a + ""` to our earlier example of `String(a)` explicit coercion, there’s one additional quirk to be aware of. Because of how the `ToPrimitive` abstract operation works, `a + ""` invokes `valueOf()` on the a value, whose return value is then finally converted to a string via the internal `ToString` abstract operation. But `String(a)` just invokes `toString()` directly.

Both approaches ultimately result in a string, but if you’re using an object instead of a regular primitive number value, you may not necessarily get the same string value!
Consider:

```js
var a = {
  valueOf: function () {
    return 42;
  },
  toString: function () {
    return 4;
  },
};
a + ""; // "42"
String(a); // "4"
```

<br>

How can we implicitly coerce from `string` to `number`?

```js
var a = "3.14";
var b = a - 0;
b; // 3.14
```

The `-` operator is defined only for numeric subtraction, so `a - 0` forces a’s value to be coerced to a number.
While far less common, `a \* 1` or `a / 1` would accomplish the same result, as those operators are also only defined for numeric operations.

What about object values with the `-` operator? Similar story as for `+` above:

```js
var a = [3];
var b = [1];
a - b; // 2
```

Both array values have to become numbers, **but they end up first being coerced to strings** (using the expected `toString()` serialization), and then are coerced to numbers, for the `-` subtraction to perform on.

<br>

### Implicitly: Booleans --> Numbers

Consider:

```js
function onlyOne(a, b, c) {
  return !!((a && !b && !c) || (!a && b && !c) || (!a && !b && c));
}
var a = true;
var b = false;
onlyOne(a, b, b); // true
onlyOne(b, a, b); // true
onlyOne(a, b, a); // false
```

This `onlyOne(..)` utility should only return `true` if exactly one of the arguments is `true` / _truthy_.

But what if we needed that utility to be able to handle four, five, or twenty flags in the same way?
here’s where coercing the boolean values to numbers (0 or 1, obviously) can greatly help:

```js
function onlyOne() {
  var sum = 0;
  for (var i = 0; i < arguments.length; i++) {
    // skip falsy values. same as treating
    // them as 0's, but avoids NaN's.
    if (arguments[i]) {
      sum += arguments[i];
    }
  }
  return sum == 1;
}

var a = true;
var b = false;
onlyOne(b, a); // true
onlyOne(b, a, b, b, b); // true
onlyOne(b, b); // false
onlyOne(b, a, b, b, b, a); // false
```

We could of course do this with explicit coercion instead:

```js
function onlyOne() {
  var sum = 0;
  for (var i = 0; i < arguments.length; i++) {
    sum += Number(!!arguments[i]);
  }
  return sum === 1;
}
```

<br>

### Implicitly: \* --> Boolean

what sort of expression operations require/force (implicitly) a boolean coercion?

1. The test expression in an `if (..)` statement
2. The test expression (second clause) in a `for ( .. ; .. ; .. )` header
3. The test expression in `while (..)` and `do..while(..)` loops
4. The test expression (first clause) in `? :` ternary expressions
5. The lefthand operand (which serves as a test expression) to the `||` (“logical or”) and `&&` (“logical and”) operators

Any value used in these contexts that is not already a boolean will be implicitly coerced to a boolean using the rules of the ToBoolean abstract operation covered earlier in this chapter.

<br>

### Operators `||` and `&&`

The value produced by a `&&` or `||` operator is not necessarily of type Boolean. The value produced will always be the value of one of the two operand expressions.

```js
var a = 42;
var b = "abc";
var c = null;
a || b; // 42
a && b; // "abc"
c || b; // "abc"
c && b; // null
```

Another way of thinking about these operators:

```js
a || b;
// roughly equivalent to:
a ? a : b;

a && b;
// roughly equivalent to:
a ? b : a;
```

I call `a || b` “**roughly equivalent**” to `a ? a : b` because **the outcome is identical**, _but there’s a nuanced difference_. In `a ? a : b`, if `a` was a more complex expression (_like for instance one that might have side effects like calling a function_, etc.), **then the `a` expression would possibly be evaluated twice (if the first evaluation was truthy)**. By contrast, **for `a || b`, the `a` expression is evaluated only once**, and that value is used both for the coercive test as well as the result value (if appropriate). The same nuance applies to the `a && b` and `a ? b : a` expressions.

<br>

### Symbol Coercion

_explicit_ coercion of a `symbol` to a `string` is allowed, but _implicit_ coercion of the same is disallowed and throws an error.

```js
var s1 = Symbol("cool");
String(s1); // "Symbol(cool)"
var s2 = Symbol("not cool");
s2 + ""; // TypeError
```

**`symbol` values cannot coerce to `number`** at all (throws an error either way), but strangely they can both _explicitly_ and _implicitly_ coerce to boolean (**always `true`**).

<br><br>

## Loose Equals Versus Strict Equals

The correct description is: "`==` allows coercion in the equality comparison and `===` disallows coercion.”

<br>

### Equality Performance

Don’t fall into the trap, as many have, of thinking this has anything to do with performance, though, as if == is going to be slower than === in any relevant way. While it’s measurable that coercion does take a little bit of processing time, it’s mere microseconds (yes, that’s millionths of a second!).s

If you want coercion, use `==` loose equality, but if you don’t want coercion, use `===` strict equality

The implication here then is that both `==` and `===` check the types of their operands. The difference is in how they respond if the types don’t match.
<br>

### Abstract Equality

The `==` operator’s behavior is defined as “**The Abstract Equality Comparison Algorithm**” in section 11.9.3 of the ES5 spec.

- `NaN` is never equal to itself.
- `+0` and `-0` are equal to each other.

<br>

for `==` loose equality comparison with `object`s (including `functions` and `arrays`). Two such values are only equal if they are both references to the exact same value. No coercion occurs here.

The `===` strict equality comparison is defined identically to 11.9.3.1, including the provision about two object values. It’s a very little known fact that **`==` and `===` behave identically in the case where two `object`s are being compared**!

<br>

#### Comparing: `string`s to `number`s

**In loose Equality (`x == y`):**

1. If `Type(x)` is Number and `Type(y)` is String, return the result of the comparison `x == ToNumber(y)`.

2. If `Type(x)` is String and `Type(y)` is Number, return the result of the comparison `ToNumber(x) == y`.

<br>

#### Comparing: anything to `boolean`

**In loose Equality `(x == y)`:**

1. If `Type(x)` is Boolean, return the result of the comparison `ToNumber(x) == y`.

2. If `Type(y)` is Boolean, return the result of the comparison `x == ToNumber(y)`.

What is relevant is to understand how the `==` comparison algorithm behaves _with all the different type combinations_. As it regards a `boolean` value on either side of the `==`, **a `boolean` always coerces to a `number` first**.

If you avoid ever using `== true` or `== false` (aka loose equality with booleans) in your code, you’ll never have to worry about this truthiness/falsiness mental gotcha.

<br>

#### Comparing: `nulls` to `undefined`s

**In loose Equality (`x == y`):**

1. If `x` is `null` and `y` is `undefined`, return `true`.
2. If `x` is `undefined` and `y` is `null`, return `true`.

`null` and `undefined` can be treated as indistinguishable for comparison purposes, if you use the `==` loose equality operator to allow their mutual implicit coercion:

```js
var a = null;
var b;
a == b; // true
a == null; // true
b == null; // true

a == false; // false
b == false; // false
a == ""; // false
b == ""; // false
a == 0; // false
b == 0; // false
```

<br>

#### Comparing: objects to nonobjects

If an object/function/array is compared to a simple scalar primitive (`string`, `number`, or `boolean`),

1. If `Type(x)` is either String or Number and `Type(y)` is Object, return the result of the comparison `x == ToPrimitive(y)`.

2. If `Type(x)` is Object and `Type(y)` is either String or Number, return the result of the comparison `ToPrimitive(x) == y`.

- You may notice that these clauses only mention String and Number, **but not Boolean**. That’s because, as quoted earlier, clauses 11.9.3.6-7 **take care of coercing any Boolean operand presented to a Number first**.

<br>

All the quirks of the `ToPrimitive` abstract operation that we discussed earlier in this chapter (`toString()`, `valueOf()`) apply here as you’d expect.

<br>

**“unboxing**,” where an object wrapper around a primitive value (like from `new String("abc")`, for instance) is unwrapped, and the underlying primitive value ("`abc`") is returned. This behavior is related to the ToPrimitive coercion in the `==` algorithm:

```js
var a = "abc";
var b = Object(a); // same as `new String( a )`
a === b; // false
a == b; // true
```

`a == b` is `true` because `b` is coerced (aka “unboxed,” unwrapped) via `ToPrimitive` to its underlying "`abc`" simple scalar primitive value, which is the same as the value in `a`.

<br>

The `null` and `undefined` values cannot be boxed—they have no object wrapper equivalent—so `Object(null)` is just like `Object()` in that both just produce a normal object.

```js
var a = null;
var b = Object(a); // same as `Object()`
a == b; // false
var c = undefined;
var d = Object(c); // same as `Object()`
c == d; // false
var e = NaN;
var f = Object(e); // same as `new Number( e )`
e == f; // false
```

- `NaN` can be boxed to its Number object wrapper equivalent, but when `==` causes an unboxing, the `NaN == NaN` comparison fails because `NaN` is never equal to itself.

<br>

### Edge Cases
