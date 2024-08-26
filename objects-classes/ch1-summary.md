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

<!-- TODO: Delete  // prettier-ignore from github itself when you finished working on the book-->

// prettier-ignore
| WARNING: |
| :------- |
| There's a third form of this feature, named "optional call", which uses `?.(` as the operator. It's used for performing a non-null'ish check on a property before executing the function value in the property. For example, instead of m`yObj.someFunc(42)` , you an do `myObj.someFunc?.(42)` . The `?.(` checks to make sure `myObj.someFunc` is non-null'ish before invoking it (with the `(42)` part). While that may sound like a useful feature, I think this is dangerous enough to warrant complete avoidance of this form/construct. My concern is that `?.(` makes it seem as if we're ensuring that the function is "callable" before calling it, when in fact we're only checking if it's non-null'ish. Unlike `?.` which can allow a "safe" `.` access against a non-null'ish value that's also not an object, the `?.(` non-null'ish check isn't similarly "safe". If the property in question has any non-null'ish, non-function value in it, like `true` or `"Hello"` , the `(42)` call part will be invoked and yet throw a JS exception. |

<br>

### Accessing Properties On Non-Objects

You can generally access properties/methods from values that aren't themselves objects.

If you perform a property access ( `.` or `[ .. ]` ) against a non-object, non-null'ish value, JS will by default (temporarily!) coerce the value into an object-wrapped representation, allowing the property access against that implicitly instantiated object.

<br>

**_boxing_**:
This process is typically called "**boxing**", as in putting a value inside a "box" (object container).

Note that `null` and `undefined` can be object-ified, by calling `Object(null)` / `Object(undefined)`. However, JS does not automatically box these null'ish values, so property access against them will fail.

<br>

## Assigning Properties

It's also possible to assign one or more properties at once -- assuming the source properties (name and value pairs) are in another object -- using the `Object.assign(..)` (added in ES6) method:
`Object.assign(..)` takes the first object as target, and the second (and optionally subsequent) object(s) as source(s).

```js
// shallow copy all (owned and enumerable) properties
// from `myObj` into `anotherObj`
Object.assign(anotherObj, myObj);

Object.assign(
  /*target=*/ anotherObj,
  /*source1=*/ {
    someProp: "some value",
    anotherProp: 1001,
  },

  /*source2=*/ {
    yetAnotherProp: false,
  }
);
```

<br>

## Deleting Properties

Once a property is defined on an object, the only way to remove it is with the `delete` operator:

```js
anotherObj = {
  counter: 123,
};

anotherObj.counter; // 123

delete anotherObj.counter;

anotherObj.counter; // undefined
```

- Calling `delete` on anything other than an object property is a misuse of the `delete` operator, and will either fail silently (in non-strict mode) or throw an exception (in strict mode).

- Deleting a property from an object is distinct from assigning it a value like `undefined` or `null` . A property assigned `undefined` , either initially or later, is still present on the object, and might still be revealed when enumerating the contents.

<br>

## Determining Container Contents

There is an important difference between how the `in` operator and the `hasOwnProperty(..)` method behave. The `in` operator will check not only the target object specified, but if not found there, it will also consult the object's `[[Prototype]]` chain. By contrast, `hasOwnProperty(..)` only consults the target object.

```js
myObj = {
  favoriteNumber: 42,
  coolFact: "the first person convicted",
  beardLength: undefined,
  nicknames: ["getify", "ydkjs"],
};

"favoriteNumber" in myObj; // true

myObj.hasOwnProperty("coolFact"); // true
myObj.hasOwnProperty("beardLength"); // true

myObj.nicknames = undefined;

myObj.hasOwnProperty("nicknames"); // true

delete myObj.nicknames;

myObj.hasOwnProperty("nicknames"); // false
```

<br>

`Object.hasOwn(..)` does essentially the same thing as `hasOwnProperty(..)`, but it's invoked as a static helper external to the object value instead of via the object's `[[Prototype]]` , making it safer and more consistent in usage:

```js
// we should now prefer:
Object.hasOwn(myObj, "favoriteNumber");
```

Even though (at time of writing) this feature is just now emerging in JS, there are polyfills that make this API available in your programs even when running in a previous JS environment that doesn't yet have the feature defined.

```js
// simple polyfill sketch for `Object.hasOwn(..)`
If (!Object.hasOwn) {
          Object.hasOwn = function hasOwn(obj,propName) {
            return Object.prototype.hasOwnProperty.call(obj,propName);
          };
}
```

<br>

### Listing All Container Contents

- `Object.entries(..)` API tells us what properties an object has (as long as they're enumerable).
- `Object.keys(..)` gives us list of the enumerable property names (aka, keys) in an object.
- `Object.values(..)` instead gives us list of all values held in enumerable properties.

<br>

If we wanted to get all the keys in an object (enumerable or not)? `Object.getOwnPropertyNames(..)` seems to do what we want, in that it's like `Object.keys(..)` but also returns non-enumerable property names.

<br>

- `Object.getOwnPropertySymbols(..)` returns all of an object's Symbol properties.

<br>

- object can also "inherit" contents from its `[[Prototype]]` chain. These are not considered owned contents, so they won't show up in any of these lists.

- Recall that the `in` operator will potentially traverse the entire chain looking for the existence of a property. Similarly, a `for..in` loop will traverse the chain and list any enumerable (owned or inherited) properties.

- there's no built-in API that will traverse the whole chain and return a list of the combined set of both owned and inherited contents.

<br>

## Temporary Containers

Using a container to hold multiple values is sometimes just a temporary transport mechanism, such as when you want to pass multiple values to a function via a single argument, or when you want a function to return multiple values.

```js
function formatValues({ one, two, three }) {
  // the actual object passed in as an
  // argument is not accessible, since
  // we destructured it into three
  // separate variables

  one = one.toUpperCase();
  two = `--${two}--`;
  three = three.substring(0, 5);

  // this object is only to transport
  // all three values in a single
  // return statement
  return { one, two, three };
}

// destructuring the return value from
// the function, because that returned
// object is just a temporary container
// to transport us multiple values

const { one, two, three } =
  // this object argument is a temporary
  // transport for multiple input values
  formatValues({
    one: "Kyle",
    two: "Simpson",
    three: "getify",
  });

one; // "KYLE"
two; // "--Simpson--"
three; // "getif"
```

<br>

## Containers Are Collections Of Properties

The most common usage of objects is as containers for multiple values.
