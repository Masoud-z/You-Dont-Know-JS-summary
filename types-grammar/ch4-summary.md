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
