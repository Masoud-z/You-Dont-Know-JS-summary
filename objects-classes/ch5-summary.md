# You Don't Know JS Yet: Objects & Classes <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 5: Delegation

<br><br>

## Preamble

Before we begin looking at delegation, I want to offer a word of caution. This perspective on JS's object `[[Prototype]]` and this function context mechanisms is not mainstream. It's not how framework authors and libraries utilize JS. You won't, to my knowledge, find any big apps out there using this pattern.

<br><br>

## What's A Constructor, Anyway?

The `constructor(..)` doesn't actually do any creation work, it's only initialization work. In other words, the instance is already created by the time the `constructor(..)` runs and initializes it -- e.g., `this.whatever` types of assignments.

So where does the creation work actually happen? In the `new` operator. As the section "New Context Invocation" in Chapter 4 explains, there are four steps the `new` keyword performs; the first of those is the creation of a new empty object (the instance). The `constructor(..)` isn't even invoked until step 3 of new 's efforts.

But `new` is not the only -- or perhaps even, best -- way to create an object "instance". Consider:

```js
// a non-class "constructor"
function Point2d(x, y) {
  // create an object (1)
  var instance = {};
  // initialize the instance (3)
  instance.x = x;
  instance.y = y;
  // return the instance (4)
  return instance;
}
var point = Point2d(3, 4);
point.x; // 3
point.y; // 4
```

The term that's most often used to refer to this pattern of code is that `Point2d(..)` here is a _factory function_. Invoking it causes the construction (creation and initialization) of an object, and returns that back to us.

I comment-annotated (1) , (3) , and (4) in that snippet, which roughly correspond to steps 1, 3, and 4 of the `new` operation. But where's step 2?

If you recall, step 2 of `new` is about linking the object (created in step 1) to another object, via its `[[Prototype]]` slot (see Chapter 2). So what object might we want to link our instance object to? We could link it to an object that holds functions we'd like to associate/use with our instance.

Let's amend the previous snippet:

```js
var prototypeObj = {
  toString() {
    return `(${this.x},${this.y})`;
  },
};

// a non-class "constructor"
function Point2d(x, y) {
  // create an object (1)
  var instance = {
    // link the instance's [[Prototype]] (2)
    __proto__: prototypeObj,
  };
  // initialize the instance (3)
  instance.x = x;
  instance.y = y;
  // return the instance (4)
  return instance;
}
var point = Point2d(3, 4);
point.toString(); // (3,4)
```

Now you see the `__proto__` assignment that's setting up the internal `[[Prototype]]` linkage, which was the missing step 2. I used the `__proto__` here merely for illustration purposes; using `setPrototypeOf(..)` as shown in Chapter 4 would have accomplished the same task.

<br>

### New Factory Instance

What do you think would happen if we used new to invoke the `Point2d(..)` function as shown here?

```js
var anotherPoint = new Point2d(5, 6);
anotherPoint.toString(5, 6); // (5,6)
```

Wait! What's going on here? A regular, non- `class` factory function in invoked with the `new` keyword, as if it was a `class` . Does that change anything about the outcome of the code?

No... and yes. `anotherPoint` here is exactly the same object as it would have been had I not used `new` . But! The object that `new` creates, links, and assigns as `this` context? That object was completely ignored and thrown away, ultimately to be garbage collected by JS. Unfortunately, the JS engine cannot predict that you're not going to use the object that you asked `new` to create, so it always still gets cteated even if it goes unused.

That's right! Using a `new` keyword against a factory function might feel more ergonomic or familiar, but it's quite wasteful, in that it creates two objects, and wastefully throws one of them away.

<br>

### Factory Initialization

In the current code example, the `Point2d(..)` function still looks an awful lot like a normal `constructor(..)` of a class definition. But what if we moved the initialization code to a separate function, say named `init(..)` :

```js
var prototypeObj = {
  init(x, y) {
    // initialize the instance (3)
    this.x = x;
    this.y = y;
  },
  toString() {
    return `(${this.x},${this.y})`;
  },
};
// a non-class "constructor"
function Point2d(x, y) {
  // create an object (1)
  var instance = {
    // link the instance's [[Prototype]] (2)
    __proto__: prototypeObj,
  };
  // initialize the instance (3)
  instance.init(x, y);
  // return the instance (4)
  return instance;
}
var point = Point2d(3, 4);
point.toString(); // (3,4)
```

The `instance.init(..)` call makes use of the `[[Prototype]]` linkage set up via `__proto__` assignment. Thus, it delegates up the prototype chain to `prototypeObj.init(..)` , and invokes it with a this context of instance -- via implicit context assignment (see Chapter 4).

Let's continue the deconstruction. Get ready for a switcheroo!

```js
var Point2d = {
  init(x, y) {
    // initialize the instance (3)
    this.x = x;
    this.y = y;
  },
  toString() {
    return `(${this.x},${this.y})`;
  },
};
```

Whoa, what!? I discarded the `Point2d(..)` function, and instead renamed the `prototypeObj` as `Point2d` . Weird.

But let's look at the rest of the code now:

```js
// steps 1, 2, and 4
var point = { __proto__: Point2d };
// step 3
point.init(3, 4);
point.toString(); // (3,4)
```

And one last refinement: let's use a built-in utility JS provides us, called `Object.create(..)` :

```js
// steps 1, 2, and 4
var point = Object.create(Point2d);
// step 3
point.init(3, 4);
point.toString(); // (3,4)
```

What operations does `Object.create(..)` perform?

1. create a brand new empty object, out of thin air.
2. link the [[Prototype]] of that new empty object to the function's .prototype object.

If those look familiar, it's because those are exactly the same first two steps of the new keyword (see Chapter 4).

Let's put this back together now:

```js
var Point2d = {
  init(x, y) {
    this.x = x;
    this.y = y;
  },
  toString() {
    return `(${this.x},${this.y})`;
  },
};
var point = Object.create(Point2d);
point.init(3, 4);
point.toString(); // (3,4)
```

Hmmm. Take a few moments to ponder what's been derived here. How does it compare to the `class` approach?

This pattern ditches the `class` and `new` keywords, but accomplishes the exact same
outcome. The cost? The single `new` operation was broken up into two statements:
`Object.create(Point2d)` and `point.init(3,4)` .

<br>

#### Help Me Reconstruct!

If having those two operations separate bothers you -- is it too deconstructed!? -- they can always be recombined in a little factory helper:

```js
function make(objType, ...args) {
  var instance = Object.create(objType);
  instance.init(...args);
  return instance;
}
var point = make(Point2d, 3, 4);
point.toString(); // (3,4)
```

Such a `make(..)` factory function helper works generally for any object-type, as long as you follow the implied convention that each `objType` you link to has a function named `init(..)` on it.

<br><br>

## Ditching Class Thinking

Class-oriented design inherently creates a hierarchy of classification, meaning how we divide up and group characteristics, and then stack them vertically in an inheritance chain. Moreover, defining a subclass is a specialization of the generalized base class. Instantiating is a specialization of the generalized class.

Behavior in a traditional class hierarchy is a vertical composition through the layers of the inheritance chain. Attempts have been made over the decades, and even become rather popular at times, to flatten out deep hierarchies of inheritance, and favor a more horizontal composition through mixins and related ideas.

<br><br>

## Delegation Illustrated

So what is delegation about? At its core, it's about two or more things sharing the effort of completing a task.

Instead of defining a `Point2d` general parent thing that represents shared behavior that a set of one or more child `point` / `anotherPoint` things inherit from, delegation moves us to building our program with discrete peer things that cooperate with each other.

I'll sketch that out in some code:

```js
var Coordinates = {
  setX(x) {
    this.x = x;
  },
  setY(y) {
    this.y = y;
  },
  setXY(x, y) {
    this.setX(x);
    this.setY(y);
  },
};
var Inspect = {
  toString() {
    return `(${this.x},${this.y})`;
  },
};
var point = {};

Coordinates.setXY.call(point, 3, 4);
Inspect.toString.call(point); // (3,4)
var anotherPoint = Object.create(Coordinates);
anotherPoint.setXY(5, 6);
Inspect.toString.call(anotherPoint); // (5,6)
```

Let's break down what's happening here.

I've defined `Coordinates` as a concrete object that holds some behaviors I associate with setting point coordinates ( `x` and `y` ). I've also defined `Inspect` as a concrete object that holds some debug inspection logic, such as `toString()` .

I then create two more concrete objects, `point` and `anotherPoint` .

point has no specific `[[Prototype]]` (default: `Object.prototype` ). Using explicit context assignment (see Chapter 4), I invoke the `Coordinates.setXY(..)` and Inspect `toString()` utilities in the context of `point` . That is what I call _explicit delegation_.

`anotherPoint` is `[[Prototype]]` linked to `Coordinates` , mostly for a bit of convenience. That lets me use implicit context assignment with `anotherPoint.setXY(..)` . But I can still explicitly share `anotherPoint` as context for the `Inspect.toString()` call. That's what I call implicit delegation.

**Don't miss this:** We still accomplished composition: we composed the behaviors from `Coordinates` and `Inspect` , during runtime function invocations with `this` context sharing. We didn't have to author-combine those behaviors into a single `class` (or base-subclass `class` hierarchy) for `point` / `anotherPoint` to inherit from. I like to call this runtime composition, **virtual composition**.

The point here is: none of these four objects is a parent or child. They're all peers of each other, and they all have different purposes. We can organize our behavior in logical chunks (on each respective object), and share the context via `this` (and, optionally `[[Prototype]]` linkage), which ends up with the same composition outcomes as the other patterns we've examined thus far in the book.

That is the heart of the **delegation** pattern, as JS embodies it.

<br><br>

## Composing Peer Objects

In the preceding snippet, point and anotherPoint merely held data, and the behaviors they delegated to were on other objects ( `Coordinates` and `Inspect` ). But we can add behaviors directly to any of the objects in a delegation chain, and those behaviors can even interact with each other, all through the magic of _virtual composition_ ( this context sharing).

To illustrate, we'll evolve our current point example a fair bit. And as a bonus we'll actually draw our points on a `<canvas>` element in the DOM. Let's take a look:

```js
var Canvas = {
  setOrigin(x, y) {
    this.ctx.translate(x, y);
    // flip the canvas context vertically,
    // so coordinates work like on a normal
    // 2d (x,y) graph
    this.ctx.scale(1, -1);
  },
  pixel(x, y) {
    this.ctx.fillRect(x, y, 1, 1);
  },
  renderScene() {
    // clear the canvas
    var matrix = this.ctx.getTransform();
    this.ctx.resetTransform();
    this.ctx.clearRect(0, 0, this.ctx.canvas.width, this.ctx.canvas.height);
    this.ctx.setTransform(matrix);
    this.draw(); // <-- where is draw()?
  },
};
var Coordinates = {
  setX(x) {
    this.x = Math.round(x);
  },
  setY(y) {
    this.y = Math.round(y);
  },
  setXY(x, y) {
    this.setX(x);
    this.setY(y);
    this.render(); // <-- where is render()?
  },
};
var ControlPoint = {
  // delegate to Coordinates
  __proto__: Coordinates,
  // NOTE: must have a <canvas id="my-canvas">
  // element in the DOM
  ctx: document.getElementById("my-canvas").getContext("2d"),
  rotate(angleRadians) {
    var rotatedX =
      this.x * Math.cos(angleRadians) - this.y * Math.sin(angleRadians);
    var rotatedY =
      this.x * Math.sin(angleRadians) + this.y * Math.cos(angleRadians);
    this.setXY(rotatedX, rotatedY);
  },
  draw() {
    // plot the point
    Canvas.pixel.call(this, this.x, this.y);
  },
  render() {
    // clear the canvas, and re-render
    // our control-point
    Canvas.renderScene.call(this);
  },
};
// set the logical (0,0) origin at this
// physical location on the canvas
Canvas.setOrigin.call(ControlPoint, 100, 100);
ControlPoint.setXY(30, 40);
// [renders point (30,40) on the canvas]
// ..
// later:

// rotate the point about the (0,0) origin
// 90 degrees counter-clockwise
ControlPoint.rotate(Math.PI / 2);
// [renders point (-40,30) on the canvas]
```

I added a couple of new concrete objects ( Canvas and ControlPoint ) alongside the previous Coordinates object.

Make sure you see and understand the interactions between these three concrete objects.

`ControlPoint` is linked (via `__proto__` ) to implicitly delegate ( `[[Prototype]]` chain) to `Coordinates` .

Here's an explicit delegation: `Canvas.setOrigin.call(ControlPoint,100,100);` ; I'm invoking the `Canvas.setOrigin(..)` call in the context of `ControlPoint` . That has the effect of sharing `ctx` with `setOrigin(..)` , via `this` .

`ControlPoint.setXY(..)` delegates implicitly to `Coordinates.setXY(..)` , but still in the context of `ControlPoint` . Here's a key detail that's easy to miss: see the `this.render()` inside of `Coordinates.setXY(..)` ? Where does that come from? Since the `this` context is `ControlPoint` (not `Coordinates` ), it's invoking `ControlPoint.render()` .

`ControlPoint.render()` explicitly delegates to `Canvas.renderScene()` , again still in the `ControlPoint` context. `renderScene()` calls `this.draw()` , but where does that come from? Yep, still from `ControlPoint` (via `this` context).

And `ControlPoint.draw()` ? It explicitly delegates to `Canvas.pixel(..)` , yet again still in the `ControlPoint` context.

All three objects have methods that end up invoking each other. But these calls aren't particularly hard-wired. `Canvas.renderScene()` doesn't call `ControlPoint.draw()` , it calls `this.draw()` . That's important, because it means that `Canvas.renderScene()` is more flexible to use in a different `this` context -- e.g., against another kind of point object besides `ControlPoint` .

It's through the `this` context, and the `[[Prototype]]` chain, that these three objects basically are mixed (composed) virtually together, as needed at each step, so that **they work together as if they're one object rather than three seperate objects**.

<br>

## Flexible Context

I mentioned above that we can pretty easily add other concrete objects into the mix. Here's an example:

```js
var Coordinates = {
  /* .. */
};
var Canvas = {
  /* .. */
  line(start, end) {
    this.ctx.beginPath();
    this.ctx.moveTo(start.x, start.y);
    this.ctx.lineTo(end.x, end.y);
    this.ctx.stroke();
  },
};
function lineAnchor(x, y) {
  var anchor = {
    __proto__: Coordinates,
    render() {},
  };
  anchor.setXY(x, y);
  return anchor;
}
var GuideLine = {
  // NOTE: must have a <canvas id="my-canvas">
  // element in the DOM
  ctx: document.getElementById("my-canvas").getContext("2d"),
  setAnchors(sx, sy, ex, ey) {
    this.start = lineAnchor(sx, sy);
    this.end = lineAnchor(ex, ey);
    this.render();
  },
  draw() {
    // plot the point
    Canvas.line.call(this, this.start, this.end);
  },
  render() {
    // clear the canvas, and re-render
    // our line
    Canvas.renderScene.call(this);
  },
};
// set the logical (0,0) origin at this
// physical location on the canvas

Canvas.setOrigin.call(GuideLine, 100, 100);
GuideLine.setAnchors(-30, 65, 45, -17);
// [renders line from (-30,65) to (45,-17)
// on the canvas]
```

That's pretty nice, I think!

But I think another less-obvious benefit is that having objects linked dynamically via `this` context tends to make testing different parts of the program independently, somewhat easier.

For example, `Object.setPrototypeOf(..)` can be used to dynamically change the `[[Prototype]]` linkage of an object, delegating it to a different object such as a mock object. Or you could dynamically redefine `GuideLine.draw()` and `GuideLine.render()` to explicitly delegate to a `MockCanvas` instead of `Canvas` .

The `this` keyword, and the [[Prototype]] link, are a tremendously flexible mechanism when you understand and leverage them fully.

<br><br>

## Why This?

You might rightly ask, why not just always pass around that context explicitly? We can certainly do so, but... to manually pass along the necessary context, we'll have to change pretty much every single function signature, and any corresponding call-sites.

By contrast, the delegation style I'm advocating for in this chapter is unfamiliar and uses `[[Prototype]]` and `this` sharing in ways you're not likely familiar with. To use such a style effectively, you'll have to invest the time and practice to build a deeper familiarity.

But in my opinion, the "cost" of avoiding virtual composition through delegation can be felt across all the function signatures and call-sites; I find them way more cluttered. That explicit context passing is quite a tax.

In fact, I'd never advocate that style of code at all. If you want to avoid delegation, it's probably best to just stick to `class` style code, as seen in Chapter 3.
