# You Don't Know JS Yet: Objects & Classes <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 1: Object Foundations

<br>

## Objects As Containers

Keys are often referred to as "property names", with the pairing of a property name and a value often called a "property". This book will use those terms distinctly in that manner.

- Regular JS objects are typically declared with literal syntax, like this:

```js
myObj = {
  // ..
};
```

<br>

## Defining Properties

Inside the object literal curly braces, you define properties (name and value) with
`propertyName: propertyValue` pairs, like this:

```js
myObj = {
  favoriteNumber: 42,
  isDeveloper: true,
  firstName: "Kyle",
};
```

The values you assign to the properties can be literals, as shown, or can be computed by expression.

<br>

### Symbols As Property Names

ES6 added a new primitive value type of `Symbol` , which is often used as a special property name for storing and retrieving property values. They're created via the `Symbol(..)` function call (without the `new` keyword), which accepts an optional description string used only for friendlier debugging purposes

<br>

### Concise Properties

In this situation, where the property name and value expression identifier are identical, you can omit the property-name portion of the property definition, as a so-called "concise property" definition:

```js
coolFact = "the first person convicted of speeding was going 8 mph";
anotherObj = {
  coolFact, // <-- concise property short-hand
};
```

<br>

### Concise Methods

Another similar shorthand is defining functions/methods in an object literal using a more concise form:

```js
anotherObj = {
  // standard function property
  greet: function () {
    console.log("Hello!");
  },

  // concise function/method property
  greet2() {
    console.log("Hello, friend!");
  },
};
```

While we're on the topic of concise method properties, we can also define generator functions (another ES6 feature):

```js
anotherObj = {
  // instead of:
  // greet3: function*() { yield "Hello, everyone!"; }

  // concise generator method
  *greet3() {
    yield "Hello, everyone!";
  },
};
```

And though it's not particularly common, concise methods/generators can even have quoted or computed names:

```js
anotherObj = {
  "greet-4"() {
    console.log("Hello, audience!");
  },

  // concise computed name
  ["gr" + "eet 5"]() {
    console.log("Hello, audience!");
  },

  // concise computed generator name
  *["ok, greet 6".toUpperCase()]() {
    yield "Hello, audience!";
  },
};
```

<br>

### Object Spread

```js
anotherObj = {
  favoriteNumber: 12,
  ...myObj, // object spread, shallow copies `myObj`
  greeting: "Hello!",
};
```

The spreading of `myObj`'s properties is shallow, in that it only copies the top-level properties from `myObj`; any values those properties hold are simply assigned over. If any of those values are references to other objects, the references themselves are assigned (by copy), but the underlying object values are not duplicated -- so you end up with multiple shared references to the same object(s).

You can think of object spreading like a for loop that runs through the properties one at a time and does an `=` style assignment from the source object ( `myObj` ) to the target object ( `anotherObj` ).

A common way `...` object spread is used is for performing shallow object duplication:

```js
myObjShallowCopy = { ...myObj };
```

the `...` object spread syntax can only appear inside the `{ .. }` object literal, which is creating a new object value. To perform a similar shallow object copy

<br>

### Deep Object Copy

Also, since `...` doesn't do full, deep object duplication, the object spread is generally only suitable for duplicating objects that hold simple, primitive values only, not references to other objects.

For deep object duplication, the standard approaches have been:

1. Use a library utility that declares a specific opinion on how the duplication behaviors/nuances should be handled.

2. Use the JSON.parse(JSON.stringify(..)) round-trip trick -- this only "works" correctly if there are no circular references, and if there are no values in the object that cannot be properly serialized with JSON (such as functions).

Recently, though, a third option has landed. This is not a JS feature, but rather a companion API provided to JS by environments like the web platform. Objects can be deep copied now using `structuredClone(..)`.

<br>

## Accessing Properties

### Object Entries

You can get a listing of the properties in an object, as an array of tuples (two-element subarrays) holding the property name and value:

```js
myObj = {
  favoriteNumber: 42,
  isDeveloper: true,
  firstName: "Kyle",
};
Object.entries(myObj);
// [ ["favoriteNumber",42], ["isDeveloper",true], ["firstName","Kyle"] ]
```

It's also possible to create a new object from a list of entries, using `Object.fromEntries(..)`

```js
myObjShallowCopy = Object.fromEntries(Object.entries(myObj));
```

<br>

### Destructuring

```js
myObj = {
  favoriteNumber: 42,
  isDeveloper: true,
  firstName: "Kyle",
};

const { favoriteNumber = 12 } = myObj;

const {
  isDeveloper: isDev,
  firstName: firstName,
  lastName: lname = "--missing--",
} = myObj;

favoriteNumber; // 42
isDev; // true
firstName; // "Kyle"
lname; // "--missing--"
```

The above snippet combines object destructuring with variable declarations -- in this example, `const` is used, but `var` and `let` work as well -- but it's not inherently a declaration mechanism. Destructuring is about access and assignment (source to target), so it can operate against existing targets rather than declaring new ones:

surrounding `( )` are required syntax here, when a declarator is not used.

```js
let fave;

({ favoriteNumber: fave } = myObj);

fave; // 42
```

<br>

### Conditional Property Access

**A?.B**

This operator will check the left-hand side reference ( `A` ) to see if it's null'ish ( `null` or `undefined` ). If so, the rest of the property access expression is short-circuited (skipped), and `undefined` is returned as the result (even if it was `null` that was actually encountered!). Otherwise, `?.` will access the property just as a normal `.` operator would.

Another form of the "optional chaining" operator is `?.[` , which is used when the property access you want to make conditional/safe requires a `[ .. ]` bracket.

```js
myObj["2 nicknames"]?.[0]; // "getify"
```

<br>

// prettier-ignore
| WARNING: |
| :------- |
| There's a third form of this feature, named "optional call", which uses `?.(` as the operator. It's used for performing a non-null'ish check on a property before executing the function value in the property. For example, instead of m`yObj.someFunc(42)` , you an do `myObj.someFunc?.(42)` . The `?.(` checks to make sure `myObj.someFunc` is non-null'ish before invoking it (with the `(42)` part). While that may sound like a useful feature, I think this is dangerous enough to warrant complete avoidance of this form/construct.
My concern is that `?.(` makes it seem as if we're ensuring that the function is "callable" before calling it, when in fact we're only checking if it's non-null'ish. Unlike `?.` which can allow a "safe" `.` access against a non-null'ish value that's also not an object, the `?.(` non-null'ish check isn't similarly "safe". If the property in question has any non-null'ish, non-function value in it, like `true` or `"Hello"` , the `(42)` call part will be invoked and yet throw a JS exception. |
