# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 8: The Module Pattern

<br>

## Encapsulation and Least Exposure (POLE)

The goal of encapsulation is the bundling or co-location of information (data) and behavior (functions) that together serve a common purpose.

The spirit of encapsulation can be realized in something as simple as using separate files to hold bits of the overall program with common purpose.

Another key goal is the control of visibility of certain aspects of the encapsulated
data and functionality.

<br>

## What Is a Module?

A **module** is a collection of related data and functions (often referred to as methods in this context), characterized by a division between hidden private details and public accessible details, usually called the “public API.”

**A module is also stateful**: it maintains some information over time, along with functionality to access and update that information

<br>

### Namespaces (Stateless Grouping)

If you group a set of related functions together, without data, then you don’t really have the expected encapsulation a module implies. The better term for this grouping of stateless functions is a namespace

<br>

### Data Structures (Stateful Grouping)

Even if you bundle data and stateful functions together, if you’re not limiting the visibility of any of it, then you’re stopping short of the POLE aspect of encapsulation; it’s not particularly helpful to label that a module.

<br>

### Modules (Stateful Access Control)

To embody the full spirit of the module pattern, we not only need grouping and state, but also access control through visibility (private vs. public)

<br>

### Classic Module Definition

So to clarify what makes something a classic module:

- There must be an outer scope, typically from a module factory function running at least once.

- The module’s inner scope must have at least one piece of hidden information that represents state for the module.

- The module must return on its public API a reference to at least one function that has closure over the hidden module state (so that this state is actually preserved)

<br>

## Node CommonJS Mod

- CommonJS modules are file-based; one module per file.
- CommonJS modules behave as singleton instances,
- `require(..)` is an all-or-nothing mechanism; it includes a reference of the entire exposed public API of the module.

<br>

## Modern ES Modules (ESM)

ESM is file-based, and module instances are singletons, with everything private by default.

- ESM files are assumed to be strict-mode, without needing a `"use strict"` pragma at the top.

- Instead of `module.exports` in CommonJS, ESM uses an `export` keyword to expose something on the public API of the module. The `import` keyword replaces the` require(..)` statement.

- `export` must be at the top-level scope; it cannot be inside any other block or function.

<br>

### “default export”:

In essence, a “default export” is a shorthand for consumers of the module when they import, giving them a terser syntax when they only need this single default API member.

<br>

### “named exports”:

Non-default exports are referred to as “named exports.”

<br>

<br>

- The `import` keyword—like `export`, it must be used only at the top level of an ESM outside of any blocks or functions.

- Multiple API members can be listed inside the `{ .. }` set, separated with commas.

- A named `import` can also be renamed with the `as` keyword:

```js
import { getName as getStudentName } from "/path/to/students.js";
getStudentName(73);
```

<br>

- The other major variation on import is called “namespace import”:

```js
import * as Student from "/path/to/students.js";
Student.getName(73); // Suzy
```

As is likely obvious, the \* imports everything exported to the API, default and named, and stores it all under the single namespace identifier as specified.

<br>

### Exit Scope

- And underneath modules, the magic of how all our module state is maintained is closures leveraging the lexical scope system.
