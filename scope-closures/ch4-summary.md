# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 4: Around the Global Scope

<br><br>

## Why Global Scope?

In addition to (potentially) accounting for where an application’s code resides during runtime, and how each piece is able to access the other pieces to cooperate, the global scope is also where:

- JS exposes its built-ins:
  - primitives: undefined, null, Infinity, NaN
  - natives: Date(), Object(), String(), etc.
  - global functions: eval(), parseInt(), etc.
  - namespaces: Math, Atomics, JSON
  - friends of JS: Intl, WebAssembly
- The environment hosting the JS engine exposes its own built-ins:
  - console (and its methods)
  - the DOM (window, document, etc)
  - timers (setTimeout(..), etc)
  - web platform APIs: navigator, history, geolocation, WebRTC, etc.

<br>
 
## Where Exactly is this Global Scope?

**Browser “Window”**
With respect to treatment of the global scope, the most pure environment JS can be run in is as a standalone .js file loaded in a web page environment in a browser.

<br>

**DOM Globals**
The auto-registration of all id-bearing DOM elements as global variables is an old legacy browser behavior that nevertheless must remain because so many old sites still rely on it.

```js
window["my-todo-list"];
// <ul id="my-todo-list">..</ul>
```

<br>

**`var` does not shadow the pre-defined name global property.**

<br>

## Web Workers

- Web Workers are a web platform extension on top of browser JS behavior, which allows a JS file to run in a completely separate thread (operating system wise) from the thread that’s running the main JS program.

- Since a Web Worker is treated as a wholly separate program, **it does not** share the global scope with the main JS program.

- However, the browser’s JS engine is still running the code, so we can expect similar purity of its global scope behavior. Since there is no DOM access, the window alias for the global scope doesn’t exist.

- In a Web Worker, the global object reference is typically made using `self`.

- Just as with main JS programs, `var` and `function` declarations create mirrored properties on the global object (aka, `self`), where other declarations (`let`, etc) do not.

<br>

## Developer Tools Console/REPL

- Developer Tools, while optimized to be convenient and useful for a variety of developer activities, are not suitable environments to determine or verify explicit and nuanced behaviors of an actual JS program context.

<br>

## ES Modules (ESM)

- **Global variables** <ins>**don’t**</ins> get created by declaring variables in the top-level scope of a module.

- The module’s top-level scope is descended from the global scope, almost as if the entire contents of the module were wrapped in a function. Thus, all variables that exist in the global scope (whether they’re on the global object or not!) are available as lexical identifiers from inside the module’s scope.

- ESM encourages a minimization of reliance on the global scope, where you import whatever modules you may need for the current module to operate. As such, you less often see usage of the global scope or its global object

<br>

## Node

**how do you define actual global variables in Node?**
The only way to do so is to add properties to another of Node’s automatically provided “globals,” which is ironically called global. global is a reference to the real global scope object, somewhat like using window in a browser JS environment.

- Remember, the identifier `global` is not defined by JS; it’s specifically defined by Node.

<br>

## Global This

```js
const theGlobalScopeObject = new Function("return this")();
```

A function can be dynamically constructed from code stored in a string value with the `Function()` constructor, similar to `eval(..)`. Such a function will automatically be run in non-strict-mode (for legacy reasons) when invoked with the normal () function invocation as shown; its this will point at the global object.

- As of ES2020, JS has finally defined a standardized reference to the global scope object, called `globalThis`.

We could even attempt to define a cross-environment polyfill that’s safer across pre-globalThis JS environments, such as:

```js
const theGlobalScopeObject =
  typeof globalThis != "undefined"
    ? globalThis
    : typeof global != "undefined"
    ? global
    : typeof window != "undefined"
    ? window
    : typeof self != "undefined"
    ? self
    : new Function("return this")();
```
