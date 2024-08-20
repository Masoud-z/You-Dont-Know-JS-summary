# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 7: Using Closures

<br>

## Closure:

for variables we need to use over time, instead of placing them in larger outer scopes, we can encapsulate (more narrowly scope) them but still preserve access from inside functions, for broader use. Functions remember these referenced scoped variables via **closure**.

- Closure is a behavior of functions and only functions.
- Each reference from the inner function to the variable in an outer scope is called a **closure**.

<br>

### Adding Up Closures

closure is observed as a runtime characteristic of function instances.

<br>

### Live Link, Not a Snapshot

Closure is actually a live link, preserving access to the full variable itself.

By closing over a variable in a function, we can keep using that variable (read and write) as long as that function reference exists in the program, and from anywhere we want to invoke that function

Though the enclosing scope of a closure is typically from a function, that’s not actually required; there only needs to be an inner function present inside an outer scope

<br>

### Common Closures: Ajax and Events

Closure is most commonly encountered with callbacks:

```js
function lookupStudentRecord(studentID) {
  ajax(`https://some.api/student/${studentID}`, function onRecord(record) {
    console.log(`${record.name} (${studentID})`);
  });
}
lookupStudentRecord(114);
// Frank (114)
```

<br>

Event handlers are another common usage of closure:

```js
function listenForClicks(btn, label) {
  btn.addEventListener("click", function onClick() {
    console.log(`The ${label} button was clicked!`);
  });
}
var submitBtn = document.getElementById("submit-btn");
listenForClicks(submitBtn, "Checkout");
```

<br>

### What If I Can’t See It?

If a closure exists (in a technical, implementation, or academic sense) but it cannot be observed in our programs, does it matter? **No**.

For example, invoking a function that makes use of lexical scope lookup:

```js
function say(myName) {
  var greeting = "Hello";
  output();
  function output() {
    console.log(`${greeting}, ${myName}!`);
  }
}
say("Kyle");
// Hello, Kyle!
```

The inner function `output()` accesses the variables greeting and `myName` from its enclosing scope. But the invocation of `output()` happens in that same scope, where of course `greeting` and `myName` are still available; **that’s just lexical scope**, not closure

In fact, global scope variables essentially cannot be (observably) closed over, because they’re always accessible from everywhere.

All function invocations can access global variables, regardless of whether closure is supported by the language or not. Global variables don’t need to be closed over.

<br>

### Observable Definition

Closure is observed when a function uses variable(s) from outer scope(s) even while running in a scope where those variable(s) wouldn’t be accessible.

<br>

## The Closure Lifecycle and Garbage Collection (GC)

Since closure is inherently tied to a function instance, its closure over a variable lasts as long as there is still a reference to that function.

<br>

### Per Variable or Per Scope?

Should we think of closure as applied only to the referenced outer variable(s), or does closure preserve the entire scope chain with all its variables?

**Conceptually, closure is per variable rather than per scope**

<br>

### An Alternative Perspective

Closure instead describes the magic of keeping alive a function instance, along with its whole scope environment and chain, for as long as there’s at least one reference to that function instance floating around in any other part of the program.
