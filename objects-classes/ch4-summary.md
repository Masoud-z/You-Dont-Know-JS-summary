# You Don't Know JS Yet: Objects & Classes <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 4: This Works

<br>

The determination of what value (usually, object) `this` points at is not made at author time, but rather determined at runtime.

In fact, a single `this` -aware function can be invoked at least four different ways, and any of those approaches will end up assigning a different `this` for that particular function invocation.

<br><br>

## This Aware

If a function does not have `this` in it anywhere, then the rules of how `this` behaves don't affect that function in any way. But if it does have even a single `this` in it, then you absolutely cannot determine how the function will behave without figuring out, for each invocation of the function, what `this` will point to.

<br>

### So What Is This?

Lexical scope (including all the variables closed over) represents a static context for the function's lexical identifier references to be evaluated against. It's fixed/static because at author time, when you place functions and variable declarations in various (nested) scopes, those decisions are fixed, and unaffected by any runtime conditions.

The `this` mechanism is, effectively, dynamic context (not scope); it's how a `this` -aware function can be dynamically invoked against different contexts -- something that's impossible with closure and lexical scope identifiers!

<br>

### Can We Get On With This?

Don't use `this` -aware code unless you really can justify it, and you've carefully weighed the costs.

<br><br>

## This Is It!

```js
var point = {
  x: null,
  y: null,
  init(x, y) {
    this.x = x;
    this.y = y;
  },
  rotate(angleRadians) {
    var rotatedX =
      this.x * Math.cos(angleRadians) - this.y * Math.sin(angleRadians);
    var rotatedY =
      this.x * Math.sin(angleRadians) + this.y * Math.cos(angleRadians);
    this.x = rotatedX;
    this.y = rotatedY;
  },
  toString() {
    return `(${this.x},${this.y})`;
  },
};
```

<br>

### Implicit Context Invocation

```js
point.init(3, 4);
```

We're invoking the `init(..)` function, but notice the `point.` in front of it? This is an implicit context binding. It says to JS: invoke the `init(..)` function with `this` referencing point.

<br>

### Default Context Invocation

```js
const init = point.init;
init(3, 4);
```

The call-site for the function is `init(3,4)` , which is different than `point.init(3,4)`. When there's no implicit context ( `point`. ), nor any other kind of `this` assignment mechanism, the default context assignment occurs.
The default assignment context won't "depend" on anything; it's pretty straightforward: `undefined` . That's it!

- Keep in mind: `undefined` does not mean "not defined"; it means, "defined with the special empty `undefined` value".

That means `init(3,4)` , if run in strict-mode, would throw an exception. Why? Because the `this.x` reference in `init(..)` is a `.x` property access on `undefined` (i.e., `undefined.x` ), which is not allowed

- never invoke a `this` -aware function without providing it a `this`

Just for completeness sake: in the less common non-strict mode, the default context is the global object -- JS defines it as `globalThis` , which in browser JS is essentially an alias to `window` , and in Node it's `global` . So, when `init(3,4)` runs in non-strict mode, the `this.x` expression is` globalThis.x` -- also known as `window.x` in the browser, or `global.x` in Node. Thus, `globalThis.x` gets set as 3 and `globalThis.y` gets set as 4 .

- That's unfortunate, because it's almost certainly not the intended outcome. Not only is it bad if it's a global variable, but it's also not changing the property on our point object, so program bugs are guaranteed.

- **Always make sure your code is running in strict-mode!**

<br>

### Explicit Context Invocation

Functions can alternately be invoked with explicit context, using the built-in call`(..)` or `apply(..)` utilities:

```js
var point = {
  x: null,
  y: null,
  init(x, y) {
    this.x = x;
    this.y = y;
  },
  rotate(angleRadians) {
    /* .. */
  },
  toString() {
    return `(${this.x},${this.y})`;
  },
};

point.init(3, 4);

var anotherPoint = {};
point.init.call(anotherPoint, 5, 6);

point.x; // 3
point.y; // 4

anotherPoint.x; // 5
anotherPoint.y; // 6
```

`init.call(point,3,4)` is effectively the same as `point.init(3,4)` , in that both of them assign point as the `this` context for the `init(..)` invocation

- Both `call(..)` and `apply(..)` utilities take as their first argument a `this` context value; that's almost always an object, but can technically can be any value (number, string, etc). The `call(..)` utility takes subsequent arguments and passes them through to the invoked function, whereas `apply(..)` expects its second argument to be an array of values to pass as arguments

<br>

```js
var point = {
  x: null,
  y: null,
  init(x, y) {
    this.x = x;
    this.y = y;
  },
  rotate(angleRadians) {
    /* .. */
  },
  toString() {
    return `(${this.x},${this.y})`;
  },
};
point.init(3, 4);
var anotherPoint = {};
point.init.call(anotherPoint, 5, 6);
point.x; // 3
point.y; // 4
anotherPoint.x; // 5
anotherPoint.y; // 6
```

Any `this` -aware functions can be borrowed like this: `point.rotate.call(anotherPoint, ..)` , `point.toString.call(anotherPoint)` .

<br>

#### Revisiting Implicit Context Invocation

Another approach to share behavior between `point` and `anotherPoint` would have been:

```js
var point = { /_ .. _/ };
var anotherPoint = {
init: point.init,
rotate: point.rotate,
toString: point.toString,
};
anotherPoint.init(5,6);
anotherPoint.x; // 5
anotherPoint.y; // 6
```

This is another way of "borrowing" the functions, by adding shared references to the functions on any target object (e.g., `anotherPoint` ). The call-site invocation `anotherPoint.init(5,6)` is the more natural/ergonomic style that relies on implicit context assignment.

It may seem this approach is a little cleaner, comparing `anotherPoint.init(5,6)` to `point.init.call(anotherPoint,5,6)`.

But the main downside is having to modify any target object with such shared function references, which can be verbose, manual, and **error-prone**. Sometimes such an approach is acceptable, but many other times, explicit context assignment with `call(..)` / `apply(..)` is more preferable

<br>

### New Context Invocation

We've so far seen three different ways of context assignment at the function call-site: **default**, **implicit**, and **explicit**. A fourth way to call a function, and assign the this for that invocation, is with the `new` keyword:

```js
var point = {
  // ..
  init: function () {
    /* .. */
  },
  // ..
};
var anotherPoint = new point.init(3, 4);
anotherPoint.x; // 3
anotherPoint.y; // 4
```

<!-- TODO: Delete  // prettier-ignore from github itself when you finished working on the book-->

// prettier-ignore
| TIP: |
| :--- |
| `init: function() { .. }` form shown here -- specifically, a function expression assigned to a property -- is required for the function to be validly called with the `new` keyword. From previous snippets, the concise method form of `init() { .. }` defines a function that cannot be called with `new` . |

You've typically seen `new` used with `class` for creating instances. But as an underlying mechanism of the JS language, `new` is not inherently a `class` operation.

In a sense, the `new` keyword hijacks a function and forces its behavior into a different mode than a normal invocation. Here are the 4 special steps that JS performs when a function is invoked with `new` :

1. create a brand new empty object, out of thin air.

2. link the `[[Prototype]]` of that new empty object to the function's `.prototype` object.

3. invoke the function with the `this` context set to that new empty object.

4. if the function doesn't return its own object value explicitly (with a `return ..`
   statement), assume the function call should instead return the new object (from steps 1-3).

<!-- TODO: Delete  // prettier-ignore from github itself when you finished working on the book-->

// prettier-ignore
| WARNING: |
| :--- |
| Step 4 implies that if you `new` invoke a function that does return its own object -- like `return { .. }` , etc -- then the new object from steps 1-3 is not returned. That's a tricky gotcha to be aware of, in that it effectively discards that new object before the program has a chance to receive and store a reference to it. Essentially, `new` should never be used to invoke a function that has explicit `return ..` statement(s) in it. |

Skipping some of the formality of these steps, let's recall an earlier snippet and see how `new` approximates a similar outcome:

```js
var point = {
  /* .. */
};
// this approach:

var anotherPoint = {};
point.init.call(anotherPoint, 5, 6);

// can instead be approximated as:
var yetAnotherPoint = new point.init(5, 6);
```

That's a bit nicer! But there's a caveat here.

Using the other functions that `point` holds against `anotherPoint` / `yetAnotherPoint` , we won't want to do with `new` . Why? Because `new` is creating a new object, but that's not what we want if we intend to invoke a function against an existing object. Instead, we'll likely use explicit context assignment:

```js
point.rotate.call(anotherPoint, /*angleRadians=*/ Math.PI);
point.toString.call(yetAnotherPoint);
// (5,6)
```

<br>

### Review This

We've seen four rules for this context assignment in function calls:

1.  Is the function invoked with `new` , creating and setting a new this ?

2.  Is the function invoked with `call(..)` or `apply(..)` , **explicitly** setting `this` ?

3.  Is the function invoked with an object reference at the call-site (e.g.,` point.init(..)` ), **implicitly** setting `this` ?

4.  If none of the above... are we in non-strict mode? If so, default the `this` to `globalThis` . But if in strict-mode, default the this to `undefined` .

<br><br>

## An Arrow Points Somewhere

Everything I've asserted so far about `this` in functions, and how its determined based on the call-site, makes one giant assumption: that you're dealing with a regular function (or method).

Here's a real example of an => arrow function:

```js
const clickHandler = (evt) =>
  evt.target.matches("button")
    ? this.theFormElem.submit()
    : evt.stopPropagation();
```

For comparison sake, let me also show the non-arrow equivalent:

```js
const clickHandler = function (evt) {
  evt.target.matches("button")
    ? this.theFormElem.submit()
    : evt.stopPropagation();
};
```

Or if we went a bit old-school about it -- this is my jam! -- we could try the standalone function declaration form:

```js
function clickHandler(evt) {
  evt.target.matches("button")
    ? this.theFormElem.submit()
    : evt.stopPropagation();
}
```

Or if the function appeared as a method in a class definition, or as a concise method in an object literal, it would look like this:

```js
// ..
clickHandler(evt) {
evt.target.matches("button") ?
this.theFormElem.submit() :
evt.stopPropagation();
}
```

What I really want to focus on is how each of these forms of the function will behave with respect to their this reference, and whether the first `=>` form differs from the others (hint: it does!).

<br>

For each of those function forms just shown, how do we know what each this will
reference?

<br>

### Where's The Call-site?

Hopefully, you responded with something like: "first, we need to see how the functions are called."

Fair enough.

Let's say our program looks like this:

```js
var infoForm = {
  theFormElem: null,
  theSubmitBtn: null,
  init() {
    this.theFormElem = document.getElementById("the-info-form");
    this.theSubmitBtn = theFormElem.querySelector("button[type=submit]");
    // is *this* the call-site?
    this.theSubmitBtn.addEventListener("click", this.clickHandler, false);
  },
  // ..
};
```

<br>

I'm guessing many of you these days are used to seeing component-framework code
(React, etc) somewhat like this:

```js
infoForm(props) {
  return (
  <form ref={this.theFormElem}>
    <button type={submit} onClick={this.clickHandler}>
      Click Me
    </button>
  </form>
  );
}
```

So how is clickHandler being invoked? What's the call-site, and which context assignment rule does it match?

### Hidden From Sight

The problem is, we cannot actually see the call-site in this program.

It doesn't matter that we passed in `this.clickHandler` . That is merely a reference to a function object value. It's not a call-site.

Under the covers, somewhere inside a framework, library, or even the JS environment itself, when a user clicks the button, a reference to the `clickHandler(..)` function is going to be invoked. And as we've implied, that call-site is even going to pass in the DOM event object as the `evt` argument.
Since we can't see the call-site, we have to imagine it. Might it look like...?

```js
eventCallback(domEventObj);
```

If it did, which this rule would apply? The default context rule (#4)?
Or, what if the call-site looked like this...?

```js
// ..
eventCallback.call(domElement, domEventObj);
```

Now which `this` rule would apply? The explicit context rule (#2)? Unless you open and view the source code for the framework/library, or read the documentation/specification, you won't know what to expect of that call-site. Which means that predicting, ultimately, what `this` points to in the `clickHandler` function you write, is... to put it mildly... a bit convoluted.

<br>

### This Is Wrong

Pretty much all implementations of a click-handler mechanism are going to do something
like the `.call(..)` , and they're going to set the DOM element (e.g., button) the event listener is bound to, as the explicit context for the invocation.

Recall that our `clickHandler(..)` function is `this` -aware, and that its `this.theFormElem` reference implies referencing an object with a `theFormElem` property, which in turn is pointing at the parent `<form>` element. DOM buttons do not, by default, have a `theFormElem` property on them. In other words, the this reference that our event handler will have set for it is almost certainly wrong. Oops.

Unless we want to rewrite the `clickHandler` function, we're going to need to fix that.

<br>

### Fixing `this`

Here's one way to address it:

```js
// store a fixed reference to the current
// `this` context
var context = this;
this.submitBtn.addEventListener(
  "click",
  function handler(evt) {
    return context.clickHandler(evt);
  },
  false
);
```

What's going on here? I recognized that the enclosing code, where the `addEventListener` call is going to run, has a current `this` context that is correct, and we need to ensure that same `this` context is applied when `clickHandler(..)` gets invoked. I defined a surrounding function ( `handler(..)` ) and then forced the call-site to look like:

```js
context.clickHandler(evt);
```

Now, it doesn't matter what the internal call-site of the library/framework environment looks like. But, why?
Because we're now actually in control of the call-site. It doesn't matter how `handler(..)` gets invoked, or what its this is assigned. It only matters than when `clickHandler(..)` is invoked, the this context is set to what we wanted.

I pulled off that trick not only by defining a surrounding function ( `handler(..)` ) so I can control the call-site, but... and this is important, so don't miss it... **I defined `handler(..)` as a NON- `this` -aware function!** There's no `this` keyword inside of `handler(..)` , so whatever `this` gets set (or not) by the library/framework/environment, is completely irrelevant.

The `var context = this` line is critical to the trick. It defines a lexical variable context , which is not some special keyword, holding a snapshot of the value in the outer `this` .
Then inside `clickHandler` , we merely reference a lexical variable ( `context` ), no
relative/magic `this` keyword.

<br>

### Lexical This
