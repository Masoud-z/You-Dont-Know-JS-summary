# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 3: The Scope Chain

<br>

## What is scope chain?

The connections between scopes that are nested within other scopes is called the scope chain, which determines the path along which variables can be accessed.

## Shadowing

- A single scope cannot have two or more variables with the same name;
- if you need to maintain two or more variables of the same name, you must use separate (often nested) scopes.

- When you choose to shadow a variable from an outer scope, one direct impact is that from that scope inward/downward (through any nested scopes) it’s now impossible for any marble to be colored as the shadowed variable

### Global Unshadowing Trick

- Variables (no matter how they’re declared!) that exist in any other scope than the global scope are completely inaccessible from a scope where they’ve been shadowed.

<br>

### Illegal Shadowing

- `let` **can** shadow `var`, but `var` **cannot** shadow `let`.
- `let` (in an inner scope) can always shadow an outer scope’s `var`. `var` (in an inner scope) can only shadow an outer scope’s `let` **if there is a function boundary in between**.
- **That boundary-crossing prohibition effectively stops at each function boundary**.

<br>

## Function Name Scope

- function expression—a function definition used as value instead of a standalone declaration—the function itself will not “**hoist**”

- One major difference between function declarations and function expressions is what happens to the name identifier of the function.

Consider a named function expression:

```js
var askQuestion = function ofTheTeacher() {
  // ..
};
```

`ofTheTeacher` is declared as an identifier inside the `function` itself:

```js
var askQuestion = function ofTheTeacher() {
  console.log(ofTheTeacher);
};

askQuestion();
// function ofTheTeacher()...
console.log(ofTheTeacher);
// ReferenceError: ofTheTeacher is not defined
```

Not only is `ofTheTeacher` declared inside the function rather than outside, but it’s also defined as read-only:

```js
var askQuestion = function ofTheTeacher() {
  "use strict";
  ofTheTeacher = 42; // TypeError
  //..
};
askQuestion();
// TypeError
```

Because we used `strict-mode`, the assignment failure is reported as a `TypeError`; in non-strict-mode, such an assignment fails silently with no exception.

A function expression with a name identifier is referred to as a “**named function expression**”, but one without a name identifier is referred to as an “**anonymous function expression**”. Anonymous function expressions clearly have no name identifier that affects either scope.
