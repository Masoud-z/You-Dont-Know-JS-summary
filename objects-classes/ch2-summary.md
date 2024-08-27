# You Don't Know JS Yet: Objects & Classes <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 1: How Objects Work

<br>

## Property Descriptors

Each property on an object is internally described by what's known as a "property descriptor". This is, itself, an object (aka, "metaobject") with several properties (aka "attributes") on it, dictating how the target property behaves.
We can retrieve a property descriptor for any existing property using
`Object.getOwnPropertyDescriptor(..)` (ES5):

```js
myObj = {
  favoriteNumber: 42,
  isDeveloper: true,
  firstName: "Kyle",
};
Object.getOwnPropertyDescriptor(myObj, "favoriteNumber");
// {
// value: 42,
// enumerable: true,
// writable: true,
// configurable: true
// }
```

We can even use such a descriptor to define a new property on an object, using
`Object.defineProperty(..)` (ES5):

```js
anotherObj = {};

Object.defineProperty(anotherObj, "fave", {
  value: 42,
  enumerable: true, // default if omitted
  writable: true, // default if omitted
  configurable: true, // default if omitted
});

anotherObj.fave; // 42
```

If an existing property has not already been marked as non-configurable (with `configurable: false` in its descriptor), it can always be re-defined/overwritten using `Object.defineProperty(..)`.

we can even define multiple properties at once, each with their own descriptor with `Object.defineProperties(…)`:

```js
anotherObj = {};
Object.defineProperties(anotherObj, {
  fave: {
    // a property descriptor
  },
  superFave: {
    // another property descriptor
  },
});
```

<br>

### Accessor Properties

A property descriptor usually defines a value property, as shown above. However, a special kind of property, known as an "accessor property" (aka, a getter/setter), can be defined. For these a property like this, its descriptor does not define a fixed value property,

```js
anotherObj = {};

Object.defineProperty(anotherObj, "fave", {
  get() {
    console.log("Getting 'fave' value!");
    return 123;
  },
  set(v) {
    console.log(`Ignoring ${v} assignment.`);
  },
});

anotherObj.fave;
// Getting 'fave' value!
// 123

anotherObj.fave = 42;
// Ignoring 42 assignment.

anotherObj.fave;
// Getting 'fave' value!
// 123
```

<br>

### Enumerable, Writable, Configurable

The `enumerable` attribute controls whether the property will appear in various enumerations of object properties, such as `Object.keys(..)` , `Object.entries(..)` , `for..in` loops, and the copying that occurs with the `...` object spread and `Object.assign(..)`.

The `writable` attribute controls whether a value assignment (via `=` ) is allowed.

However, as long as the property is still `configurable`, `Object.defineProperty(..)` can still change the value by setting value differently.

The `configurable` attribute controls whether a property's descriptor can be redefined/overwritten.

A property that's `configurable: false` is locked to its definition, and any further attempts to change it with `Object.defineProperty(..)` will fail. A non configurable property can still be assigned new values (via `=` ), as long as `writable: true` is still set on the property's descriptor.

<br>

## Object Sub-Types

Two most common sub-types of objects you'll interact with are **arrays** and **`function`** s.

<br>

### Arrays

Arrays are objects that are specifically intended to be numerically indexed, rather than using string named property locations.

<br>

**Empty Slots**:
If you assign an index of an array more than one position beyond the current end of the array, JS will leave the in between slots "empty" rather than auto-assigning them to undefined as you might expect.

<br>

There are APIs in JS, like array's `map(..)` , where empty slots are surprisingly skipped over.

<br>

### Functions

```js
function help(opt1, opt2, ...remainingOpts) {
  // ..
}
help.name; // "help"
help.length; // 2
```

The `length` of a function is the count of its explicitly defined parameters, up to but not including a parameter that either has a default value defined (e.g., `param = 42` ) or a "rest parameter" (e.g., `...remainingOpts` )

<br>

#### Avoid Setting Function-Object Properties

You should avoid assigning properties on function objects. If you're looking to store extra information associated with a function, use a separate `Map(..)` (or `WeakMap(..)` ) with the function object as the key, and the extra information as the value.

```js
extraInfo = new Map();
extraInfo.set(help, "this is some important information");
// later:
extraInfo.get(help); // "this is some important information"
```

<br>

## Object Characteristics

In addition to defining behaviors for specific properties, certain behaviors are configurable across the whole object:

- extensible
- sealed
- frozen

### Extensible

Extensibility refers to whether an object can have new properties defined/added to it.
you can change shut off extensibility for an object with `Object.preventExtensions(myObj)`;

```js
myObj = {
  favoriteNumber: 42,
};

myObj.firstName = "Kyle"; // works fine

Object.preventExtensions(myObj);

myObj.nicknames = ["getify", "ydkjs"]; // fails
myObj.favoriteNumber = 123; // works fine
```

<br>

### seal

`Object.seal(..)` creates a “sealed” object, which means it takes an existing object and essentially calls `Object.preventExtensions(..)` on it, but also marks all its existing properties as `configurable:false`. So, not only can you not add any more properties, but you also cannot reconfigure or delete any existing properties (**_though you can still modify their values_**).

<br>

### Frozen

`Object.freeze(..)` creates a frozen object, which means it takes an existing object and essentially calls `Object.seal(..)` on it, but it also marks all “data accessor” properties as `writable:false`, so that their values cannot be changed.

- It prevents any changes to the object or to any of its direct properties (**_though, as mentioned earlier, the contents of any referenced other objects are unaffected_**).

<br>

## Extending The MOP

Objects in JS behave according to a set of rules referred to as the **_Metaobject Protocol (MOP)_**.

// TODO unfinished!

<br>

## [[Prototype]] Chain

The `[[Prototype]]` is an internal linkage that an object gets by default when its created, pointing to another object.

By default, all objects are `[[Prototype]]` -linked to the built-in object named `Object.prototype`

The ability for `myObj.toString` to access the `toString` property even though it doesn't actually have it, is commonly referred to as "**inheritance**", or more specifically, "**prototypal inheritance**". The `toString` and `hasOwnProperty` properties, along with many others, are said to be "**inherited properties**" on `myObj` .

<br>

Some common "inherited" properties from `Object.prototype` include:

- `constructor`
- `__proto__`
- `toString()`
- `valueOf()`
- `hasOwnProperty(..)`
- `isPrototypeOf(..)`

<br>

- As of ES2022, JS has finally added the static version of this utility: `Object.hasOwn(..)`.

```js
myObj = {
  favoriteNumber: 42,
};

Object.hasOwn(myObj, "favoriteNumber"); // true
```

- This form is now considered the more preferable and robust option, and the instance method ( `hasOwnProperty(..)` ) form should now generally be avoided.

<br>

### Creating An Object With A Different [[Prototype]]

By default, any object you create in your programs will be `[[Prototype]]` -linked to that `Object.prototype` object.
But you can create an object with a different linkage like this:

```js
myObj = Object.create(differentObj);
```

The `Object.create(..)` method takes its first argument as the value to set for the newly created object's `[[Prototype]]` .

- If you want to create an object without any `[[Prototype]]`, you should pass `null` to `Object.create(..)`.

Alternately, but less preferably, you can use the `{ .. }` literal syntax along with a special (and strange looking!) property, `__proto__` :

```js
myObj = {
  __proto__: differentObj,
  // .. the rest of the object definition
};
```

Whether you use `Object.create(..)` or `__proto__` , the created object in question will **usually** be `[[Prototype]]` -linked to a different object than the default `Object.prototype` .

<br>

### Empty [[Prototype]] Linkage

If you want to create an object without any [[Prototype]], you should pass `null` to `Object.create(..)` or you can set null to `__proto__` property.

- These types of (useful!) objects are sometimes referred to in popular parlance as "**dictionary objects**".
