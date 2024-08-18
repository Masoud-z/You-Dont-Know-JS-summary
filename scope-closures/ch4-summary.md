# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 4: Around the Global Scope

<br>

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
