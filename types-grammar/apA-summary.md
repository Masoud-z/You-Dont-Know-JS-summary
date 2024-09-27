# You Don't Know JS : Types & Grammar <ins>**_Summary_**</ins> - 1st Edition

<br>

# Appendix A: Mixed Environment JavaScript

<br><br>

## Annex B (ECMAScript)

The official ECMAScript specification includes “Annex B,” which discusses specific deviations from the official spec for the purposes of JS compatibility in browsers.

The main compatibility differences:

- Octal number literals are allowed, such as 0123 (decimal 83) in non-strict mode.

- `window.escape(..)` and `window.unescape(..)` allow you to escape or unescape strings with `%-delimited` hexadecimal escape sequences. For example: `window.escape( "? foo=97%&bar=3%" )` produces "`%3Ffoo%3D97%25%26bar %3D3%25`".

- `String.prototype.substr` is quite similar to `String.prototype.substring`, except that instead of the second parameter being the ending index (noninclusive), the second parameter is the length (number of characters to include)

<br>

### Web ECMAScript

The Web ECMAScript specification covers the differences between the official ECMAScript specification and the current JavaScript implementations in browsers.

<br>

## Host Objects

The well-covered rules for how variables behave in JS have exceptions to them when it comes to variables that are auto-defined, or otherwise created and provided to JS by the environment that hosts your code (browser, etc.)—so-called “host objects” (which include both built-in objects and functions).

```js
var a = document.createElement("div");
typeof a; // "object"--as expected
Object.prototype.toString.call(a); // "[object HTMLDivElement]"
a.tagName; // "DIV"
```

`a` is not just an object, but a special host object because it’s a DOM element. It has `a` different internal `[[Class]]` value ("`HTMLDivElement`") and comes with predefined (and often unchangeable) properties.

Other behavior variations with host objects to be aware of can include:

- Not having access to normal object built-ins like `toString()`
- Not being overwritable
- Having certain predefined read-only properties
- Having methods that cannot be this-overridden to other objects
- And more…

<br><br>

## Global DOM Variables

You’re probably aware that declaring a variable in the global scope (with or without `var`) creates not only a global variable, but also its mirror: a property of the same name on the global object (`window` in the browser)

creating DOM elements with id attributes creates global variables of those same names.

```js
<div id="foo"></div>;

if (typeof foo == "undefined") {
  foo = 42; // will never run
}
console.log(foo); // HTML element
```

<br><br>

## Native Prototypes

One of the most widely known and classic pieces of JavaScript best practice wisdom is: never extend native prototypes.

Don’t unconditionally define extensions (because you can overwrite natives accidentally).

<br>

should you ever rely on native built-in behavior if your code is running in any environment where it’s not the only code present?

The strict answer is no, but that’s awfully impractical. Your code usually can’t redefine its own private untouchable versions of all built-in behavior relied on. Even if you could, that’s pretty wasteful.

So, should you feature-test for the built-in behavior as well as compliance-test that it does what you expect? And what if that test fails—should your code just refuse to run?

In theory, that sounds plausible, but it’s also pretty impractical to design tests for every single built-in method.

If you don’t do it, and no one else does in the code in your application, you’re safe. Otherwise, you should build in at least a little bit of skepticism, pessimism, and expectation of possible breakage.

<br>

### Shims/Polyfills
