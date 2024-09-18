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

- But if you try to `JSON.stringify(..)` an object with circular reference(s) in it, an error will be thrown.

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

- An optional second argument can be passed to `JSON.stringify(..)` that is called **replacer**.
  This argument can either be an array or a function. It’s used to customize the recursive serialization of an object by providing a filtering mechanism for which properties should and should not be included,
