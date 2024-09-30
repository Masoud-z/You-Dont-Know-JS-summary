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

It’s usually said that the only safe place to extend a native is in an older (non-spec-compliant) environment, since that’s unlikely to ever change—new browsers with new spec features replace older browsers rather than amending them.

```js
if (!Array.prototype.foobar) {
  // silly, silly
  Array.prototype.foobar = function () {
    this.push("foo", "bar");
  };
}
```

If there’s already a spec for `Array.prototype.foobar`, and the specified behavior is equal to this logic, you’re pretty safe in defining such a snippet, and in that case it’s generally called a “polyfill” (or “shim”).

Such code is very useful to include in your code base to “patch” older browser environments that aren’t updated to the newest specs. Using polyfills is a great way to create predictable code across all your supported environments.

<br>

## <script>s

The one thing they (script tags) share is the single global object (`window` in the browser), which means multiple files can append their code to that shared namespace and they can all interact.

**But global variable scope hoisting does not occur across these boundaries**

<br>

So the following code would not work:

```html
<script>
  foo();
</script>

<script>
  function foo() { .. }
</script>
```

<br>

But either of these would work instead:

```html
<script>
  foo();
  function foo() { .. }
</script>
```

or

```html
<script>
  function foo() { .. }
</script>
<script>
  foo();
</script>
```

Also, if an error occurs in a script element (inline or external), as a separate standalone JS program it will fail and stop, but any subsequent scripts will run (still with the shared global) unimpeded.

<br>

You can create script elements dynamically from your code, and inject them into the DOM of the page, and the code in them will behave basically as if loaded normally in a separate file:

```js
var greeting = "Hello World";
var el = document.createElement("script");
el.text = "function foo(){ alert( greeting );} setTimeout( foo, 1000 );";

document.body.appendChild(el);
```

The charset attribute will not work on inline script elements.

<br><br>

## Reserved Words

The ES5 spec defines a set of “reserved words” in Section 7.6.1 that cannot be used as standalone variable names. Technically, there are four categories: “keywords,” “future reserved words,” the `null` literal, and the `true`/`false` boolean literals.

Prior to ES5, the reserved words also could not be property names or keys in object literals, but that restriction no longer exists.

StackOverflow user “art4theSould” creatively worked all these reserved words into a fun little poem:

> Let this long package float,
> Goto private class if short.
> While protected with debugger case,
> Continue volatile interface.
> Instanceof super synchronized throw,
> Extends final export throws.
>
> Try import double enum?
> False, boolean, abstract function,
> Implements typeof transient break!
> Void static, default do,
>
> Switch int native new.
> Else, delete null public var
> In return for const, true, char
> …Finally catch byte.
