# You Don't Know JS Yet: Get Started <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 2: Surveying JS

<br><br>

## What is interpolation?

```js
console.log(`My name is ${firstName}.`);
// My name is Kyle.
```

Assuming this program has already defined a variable `firstName` with the string value "Kyle", the `-delimited string then resolves the variable expression (indicated with ${ ..}) to its current value. This is called **interpolation**.

## What are primitive values?

In addition to `strings`, `numbers`, and `booleans`, two other primitive values in JS programs are `null` and `undefined`.
The final primitive value to be aware of is a **symbol**, which is a special-purpose value that behaves as a hidden unguessable value. Symbols are almost exclusively used as special keys on
objects:

```js
hitchhikersGuide[Symbol("meaning of life")];
// 42
```

- arrays are a special type of object that’s comprised of an ordered and numerically indexed list of data.

- Objects are more general: an unordered, keyed collection of any various values. In other words, you access the element by a string location name (aka “key” or “property”) rather than by its numeric position (as with arrays).

<br><br>

## What is coercion?

Converting from one value type to another, such as from string to number, is referred to in JS as “**coercion**.”

- In JS, all object values are held by reference

- If the value types being compared are different, the `==` differs from `===` in that it allows coercion before the comparison.

- Instead of “loose equality,” the `==` operator should be described as “**coercive equality**”.

<br><br>

## ES Modules

ES modules (ESM), introduced to the JS language in ES6:

- There’s no wrapping function to define a module. The wrapping context is a file. ESMs are always file-based; one file, one module.
- You don’t interact with a module’s “API” explicitly, but rather use the export keyword to add a variable or method to its public API definition.
- You don’t “instantiate” an ES module, you just import it to use its single instance.
