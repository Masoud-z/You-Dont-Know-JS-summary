# You Don't Know JS Yet: Objects & Classes <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 4: This Works

<br><br>

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

The name for this pattern, by the way, is "lexical this", meaning a this that behaves like a lexical scope variable instead of like a dynamic context binding

- In an `=>` function, the `this` keyword... **is not a keyword**. It's absolutely no different from any other variable, like `context` or `happyFace` or `foobarbaz` .

```js
function outer() {
  console.log(this.value);
  // define a return an "inner"
  // function
  var inner = () => {
    console.log(this.value);
  };
  return inner;
}

var one = {
  value: 42,
};

var two = {
  value: "sad face",
};

var innerFn = outer.call(one); // 42

innerFn.call(two); // 42 <-- not "sad face"
```

The `innerFn.call(two)` would, for any regular function definition, have resulted in "sad face" here. But since the `inner` function we defined and returned (and assigned to `innerFn` ) was an irregular `=>` arrow function, it has no special `this` behavior, but instead has "lexical this" behavior.

When the `innerFn(..)` (aka `inner(..)` ) function is invoked, even with an explicit context assignment via `.call(..)` , that assignment is ignored.

<br>

When a `this` is encountered ( `this.value` ) inside an `=>` arrow function, `this` is treated like a normal lexical variable, not a special keyword. And since there is no `this` variable in that function itself, JS does what it always does with lexical variables: it goes up one level of lexical scope -- in this case, to the surrounding `outer(..)` function, and it checks to see if there's any registered `this` in that scope.

<br>

### Back To The... Button

Hear me on `this`: the `=>` arrow function is not -- I repeat, not -- about typing fewer characters. **The primary point of the `=>` function being added to JS was to give us "lexical this" behavior** without having to resort to `var context = this` (or worse, `var self = this` ) style hacks.

<br>

### Confession Time

I've said all along in this chapter, that how you write a function, and where you write the function, has nothing to do with how its `this` will be assigned.
For regular functions, that's true. But when we consider an irregular `=>` arrow function, it's not entirely accurate anymore.

When we write an `=>` arrow function, we know for sure that its `this` binding will exactly be the current `this` binding of whatever surrounding function is running, regardless of what the call-site of the `=>` arrow function looks like. So in other words, how we wrote the `=>` arrow function, and where we wrote it, does matter.

- That doesn't fully answer the `this` question, though. It just shifts the question to **how the enclosing function was invoked**.

<br>

Since an `=>` arrow function never has a `this` -assigning call-site (no matter what), that call-site isn't relevant to the question. We have to keep stepping up the call stack until we find a function invocation that is `this` -assigning -- even if such invoked function is not itself `this` -aware.

<br>

#### Find The Right Call-Site

Let me illustrate, with a convoluted mess of a bunch of nested functions/calls:

```js
globalThis.value = { result: "Sad face" };
function one() {
  function two() {
    var three = {
      value: { result: "Hmmm" },
      fn: () => {
        const four = () => this.value;
        return four.call({
          value: { result: "OK" },
        });
      },
    };
    return three.fn();
  }
  return two();
}
new one(); // ???
```

The call-stack for that new one() invocation is:

```js
four       |
three.fn   |
two        | (this = globalThis)
one        | (this = {})
[ global ] | (this = globalThis)
```

Since `four()` and `fn()` are both `=>` arrow functions, the `three.fn()` and `four.call(..)` call-sites are not `this` -assigning; thus, they're irrelevant for our query. What's the next invocation to consider in the call-stack? `two()` . That's a regular function (it can accept `this` -assignment), and the call-site matches the default context assignment rule (#4). Since we're not in strict-mode, `this` is assigned `globalThis` .

When `four()` is running, `this` is just a normal variable. It looks then to its containing function ( `three.fn()` ), but it again finds a function with no `this` . So it goes up another level, and finds a `two()` regular function that has a `this` defined. And that `this` is `globalThis` . So the `this.value` expression resolves to `globalThis.value` , which returns us... `{ result: "Sad face" }` .

<br>

Choosing `this` -oriented code (even `class` es) means choosing both the flexibility it affords us, as well as needing to be comfortable navigating the call-stack to understand how it will behave.

That's the only way to effectively write (and later read!) `this` -aware code.

<br>

### This Is Bound To Come Up

In addition to `call(..)` / `apply(..)` -- these invoke functions, remember! -- JS functions also have a third utility built in, called `bind(..)` -- **which does not invoke the function**, just to be clear.

The `bind(..)` utility defines a new wrapped/bound version of a function, where its `this` is preset and fixed, and cannot be overridden with a `call(..)` or `apply(..)` , or even an implicit context object at the call-site:

```js
this.submitBtn.addEventListener("click", this.clickHandler.bind(this), false);
```

Since I'm passing in a `this` -bound function as the event handler, it similarly doesn't matter how that utility tries to set a `this` , because I've already forced the `this` to be what I wanted: the value of `this` from the surrounding function invocation context.

<br>

#### Hardly New

This pattern is often referred to as "hard binding", since we're creating a function reference that is strongly bound to a particular `this` . A lot of JS writings have claimed that the `=>` arrow function is essentially just syntax for the `bind(this)` hard-binding. It's not. Let's dig in.

<br>

If you were going to create a `bind(..)` utility, it might look kinda like `this`:

```js
function bind(fn, context) {
  return function bound(...args) {
    return fn.apply(context, args);
  };
}
```

Does that look familiar? It's using the good ol' fake "lexical this" hack. And under the covers, it's an _explicit context_ assignment, in this case via `apply(..)` .

So wait... doesn't that mean we could just do it with an `=>` arrow function?

```js
function bind(fn, context) {
  return (...args) => fn.apply(context, args);
}
```

Eh... not quite. As with most things in JS, there's a bit of nuance. Let me illustrate:

```js
// candidate implementation, for comparison
function fakeBind(fn, context) {
  return (...args) => fn.apply(context, args);
}
// test subject
function thisAwareFn() {
  console.log(`Value: ${this.value}`);
}
// control data
var obj = {
  value: 42,
};
// experiment
var f = thisAwareFn.bind(obj);
var g = fakeBind(thisAwareFn, obj);

f(); // Value: 42
g(); // Value: 42

new f(); // Value: undefined
new g(); // <--- ???
```

First, look at the `new f()` call. That's admittedly a strange usage, to call `new` on a hardbound function. It's probably quite rare that you'd ever do so. But it shows something kind of interesting. Even though `f()` was hard-bound to a this context of `obj` , the new operator was able to hijack the hard-bound function's `this` and re-bind it to the newly created and empty object. That object has no value property, which is why we see "`Value: undefined`" printed out.

Refer back to the four rules presented earlier in this chapter. Remember how I asserted their order-of-precedence, and `new` was at the top (#1), ahead of explicit `call(..)` / `apply(..)` assignment rule (#2)?

Since we can sort of think of `bind(..)` as a variation of that rule, we now see that orderof-precedence proven. `new` is more precedent than, and can override, even a hard-bound function. Sort of makes you think the hard-bound function is maybe not so "hard"-bound, huh?!

<br>

But... what's going to happen with the `new g()` call, which is invoking new on the returned `=>` arrow function? Do you predict the same outcome as `new f()` ?

Sorry to disappoint.

That line will actually throw an exception, because **an `=>` function cannot be used with the `new` keyword**.

But why? My best answer, not being authoritative on TC39 myself, is that **conceptually and actually, an `=>` arrow function is not a function with a hard-bound `this` , it's a function that has no `this` at all. As such, `new` makes no sense against such a function, so JS just disallows it**.

- But the main point is: an `=>` arrow function is not a syntactic form of `bind(this)` .

<br>

### Losing This Battle

Presetting a `this` assignment, no matter how you do it, so that it's predictable, comes with a cost. A system level cost and a program maintenance/complexity cost. _It is **never** free_.

One way of reacting to that fact is to decide, OK, we're just going to manufacture all those `this` -assigned function references once, ahead of time, up-front. That way, we're sure to reduce both the system pressure, and the code pressure, to a minimum. Sounds reasonable, right? Not so fast

<br>

#### Pre-Binding Function Contexts

If you have a one-off function reference that needs to be `this` -bound, and you use an `=>` arrow or a `bind(this)` call, I don't see any problems with that.

But if most or all of the `this` -aware functions in a segment of your code invoked in ways where the `this` isn't the predictable context you expect, and so you decide you need to hard-bind them all... I think that's a big warning signal that you're going about things the wrong way.

Please recall the discussion in the "Avoid This" section from Chapter 3, which started with this snippet of code:

```js
class Point2d {
  x = null;
  getDoubleX = () => this.x * 2;
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  toString() {
    /* .. */
  }
}
var point = new Point2d(3, 4);
```

Now imagine we did this with that code:

```js
const getX = point.getDoubleX;
// later, elsewhere
getX(); // 6
```

As you can see, the problem we were trying to solve is the same as we've been dealing with here in this chapter. It's that we wanted to be able to invoke a function reference like `getX()` , and have that mean and behave like `point.getDoubleX()` . But `this` rules on regular functions don't work that way.

So we used an `=>` arrow function. No big deal, right!?
Wrong.
The real root problem is that we want two conflicting things out of our code, and we're trying to use the same hammer for both nails.

<br>

We want to have a `this` -aware method stored on the class prototype, so that there's only one definition for the function, and all our subclasses and instances nicely share that same function. And the way they all share is through the power of the dynamic `this` binding.

But at the same time, we also want those function references to magically stay `this` - assinged to our instance when we pass those function references around and other code is in charge of the call-site

In other words, sometimes we want something like `point.getDoubleX` to mean, "give me a reference that's `this` -assigned to point ", and other times we want the same expression `point.getDoubleX` to mean, give me a dynamic `this` -assignable function reference so it can properly get the context I need it to at this moment.

What JS authors of `class` -oriented code often run up against, sooner or later, is this exact tension. And you know what they do?

They don't consider the costs of simply pre-binding all the class's `this` -aware methods as instead `=>` arrow functions in member properties. They don't realize that it's completely defeated the entire purpose of the `[[Prototype]]` chain. And they don't realize that if fixed-context is what they really need, there's an entirely different mechanism in JS that is better suited for that purpose.

<br>

#### Take A More Critical Look

```js
class Point2d {
  x = null;
  y = null;
  getDoubleX = () => this.x * 2;
  toString = () => `(${this.x},${this.y})`;
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}
var point = new Point2d(3, 4);
var anotherPoint = new Point2d(5, 6);
var f = point.getDoubleX;
var g = anotherPoint.toString;
f(); // 6
g(); // (5,6)
```

I say, "ick!", to the hard-bound `this` -aware methods `getDoubleX()` and `toString()` there. To me, that's a code smell. But here's an even worse approach that has been favored by many developers in the past:

```js
class Point2d {
  x = null;
  y = null;
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.getDoubleX = this.getDoubleX.bind(this);
    this.toString = this.toString.bind(this);
  }
  getDoubleX() {
    return this.x * 2;
  }
  toString() {
    return `(${this.x},${this.y})`;
  }
}
var point = new Point2d(3, 4);
var anotherPoint = new Point2d(5, 6);
var f = point.getDoubleX;
var g = anotherPoint.toString;
f(); // 6
g(); // (5,6)
```

Double ick.

In both cases, you're using a `this` mechanism but completely betraying/neutering it, by taking away all the powerful dynamicism of `this` . You really should at least be contemplating `this` alternate approach, which skips the whole `this` mechanism altogether:

```js
function Point2d(px, py) {
  var x = px;
  var y = py;
  return {
    getDoubleX() {
      return x * 2;
    },
    toString() {
      return `(${x},${y})`;
    },
  };
}
var point = Point2d(3, 4);
var anotherPoint = Point2d(5, 6);
var f = point.getDoubleX;
var g = anotherPoint.toString;
f(); // 6
g(); // (5,6)
```

You see? No ugly or complex `this` to clutter up that code or worry about corner cases for. Lexical scope is super straightforward and intuitive. When all we want is for most/all of our function behaviors to have a fixed and predictable context, the most appropriate solution, the most straightforward and even performant solution, is lexical variables and scope closure.

When you go to all to the trouble of sprinkling this references all over a piece of code, and then you cut off the whole mechanism at the knees with `=>` "lexical this" or `bind(this)` , you chose to make the code more verbose, more complex, more overwrought. And you got nothing out of it that was more beneficial, except to follow the `this` (and `class` ) bandwagon.

<br><br>

## Variations

<br>

### Indirect Function Calls

Recall this example from earlier in the chapter?

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
    /* .. */
  },
};
var init = point.init;
init(3, 4); // broken!
```

This is broken because the init(3,4) call-site doesn't provide the necessary `this` - assignment signal. But there's other ways to observe a similar breakage.

For example:

```js
(1, point.init)(3, 4); // broken!
```

This strange looking syntax is first evaluating an expression `(1,point.init)` , which is a comma series expression. The result of such an expression is the final evaluated value, which in this case is the function reference (held by `point.init` ).

So the outcome puts that function reference onto the expression stack, and then invokes that value with (3,4) . That's an indirect invocation of the function. And what's the result?

It actually matches the default context assignment rule (#4) we looked at earlier in the chapter. Thus, in non-strict mode, the `this` for the `point.init(..)` call will be `globalThis` .

Had we been in strict-mode, it would have been `undefined` , and the `this.x = x` operation would then have thrown an exception for invalidly accessing the `x` property on the `undefined` value.

There's several different ways to get an indirect function invocation. For example:

```js
(() => point.init)()(3, 4); // broken!
```

And another example of indirect function invocation is the Immediately Invoked Function Expression (IIFE) pattern:

```js
(function () {
  // `this` assigned via "default" rule
})();
```

As you can see, the function expression value is put onto the expression stack, and then it's invoked with the `()` on the end.

But what about this code:

<!-- TODO: Delete  // prettier-ignore from github itself when you finished working on the book-->

```js
// prettier-ignore
(point.init)(3,4);
```

What will be the outcome of that code?
By the same reasoning we've seen in the previous examples, it stands to reason that the `point.init` expression puts the function value onto the expression stack, and then invoked indirectly with (3,4) .

Not quite, though! JS grammar has a special rule to handle the invocation form `(someIdentifier)(..)` as if it had been `someIdentifier(..)` (without the `(..)` around the identifier name).

Wondering why you might want to ever force the default context for `this` assignment via an indirect function invocation?

<br>

### Accessing globalThis

Before we answer that, let's introduce another way of performing indirect function this assignment. Thus far, the indirect function invocation patterns shown are sensitive to strict-mode. But what if we wanted an indirect function `this` assignment that doesn't respect strict-mode.

The `Function(..)` constructor takes a string of code and dynamically defines the equivalent function. However, it always does so as if that function had been declared in the global scope. And furthermore, it ensures such function does not run in strict-mode, no matter the strict-mode status of the program. That's the same outcome as running an indirect

One niche usage of such strict-mode agnostic indirect function this assignment is for getting a reliable reference to the true global object prior to when the JS specification actually defined the `globalThis` identifier (for example, in a polyfill for it):

```js
"use strict";
var gt = new Function("return this")();
gt === globalThis; // true
```

In fact, a similar outcome, using the comma operator trick (see previous section) and eval(..) :

```js
"use strict";
function getGlobalThis() {
  return (1, eval)("this");
}
g;
etGlobalThis() === globalThis; // true
```

Unfortunately, the `new Function(..)` and `(1,eval)(..)` approaches both have an important limitation: that code will be blocked in browser-based JS code if the app is served with certain Content-Security-Policy (CSP) restrictions, disallowing dynamic code evaluation (for security reasons).

Can we get around this? Yes, mostly.

The JS specification says that a getter function defined on the global object, or on any object that inherits from it (like `Object.prototype` ), runs the getter function with this context assigned to globalT`his , regardless of the program's strict-mode.

```js
// Adapted from: https://mathiasbynens.be/notes/globalthis#robust-polyfill
function getGlobalThis() {
  Object.defineProperty(Object.prototype, "__get_globalthis__", {
    get() {
      return this;
    },
    configurable: true,
  });
  var gt = __get_globalthis__;
  delete Object.prototype.__get_globalthis__;
  return gt;
}
g;
etGlobalThis() === globalThis; // true
```

<br>

### Template Tag Functions

There's one more unusual variation of function invocation we should cover: tagged
template functions.

Template strings -- what I prefer to call interpolated literals -- can be "tagged" with a prefix function, which is invoked with the parsed contents of the template literal:

```js
function tagFn(/* .. */) {
  // ..
}
tagFn`actually a function invocation!`;
```

As you can see, there's no `(..)` invocation syntax, just the tag function ( `tagFn` ) appearing before the `` `template literal` `` ; whitespace is allowed between them, but is very uncommon.

Despite the strange appearance, the function `tagFn(..)` will be invoked. It's passed the list of one or more string literals that were parsed from the template literal, along with any interpolated expression values that were encountered.

We're not going to cover all the ins and outs of tagged template functions -- they're seriously one of the most powerful and interesting features ever added to JS -- but since we're talking about `this` assignment in function invocations, for completeness sake we need to talk about how `this` will be assigned.

The other form for tag functions you may encounter is:

```js
var someObj = {
  tagFn() {
    /* .. */
  },
};
someObj.tagFn`also a function invocation!`;
```

Here's the easy explanation: `` tagFn`..` `` and `` someObj.tagFn`..` `` will each have `this` - assignment behavior corresponding to call-sites as `tagFn(..)` and `someObj.tagFn(..)` , respectively. In other words, `` tagFn`..` `` behaves by the default context assignment rule (#4), and `` someObj.tagFn`..` `` behaves by the implicit context assignment rule (#3).

Luckily for us, we don't need to worry about the `new` or` call(..)` / `apply(..)` assignment rules, as those forms aren't possible with tag functions.

It should be pointed out that it's pretty rare for a tagged template literal function to be defined as `this` -aware, so it's fairly unlikely you'll need to apply these rules. But just in case, now you're in the know.

<br><br>

## Stay Aware

The lesson here is that you should be intentional and aware of all aspects of this before you go sprinkling it about your code. Make sure you're using it most effectively and taking full advantage of this important pillar of JS
