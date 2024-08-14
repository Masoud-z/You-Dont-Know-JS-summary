# You Don't Know JS Yet: Get Started **_Summary_** - 2nd Edition

<br>

# Chapter 1: What _Is_ JavaScript?

<br>
<br>

# Chapter 2: Surveying JS

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
