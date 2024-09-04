# You Don't Know JS Yet: Objects & Classes <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 3: Classy Objects

<br><br>

## When Should I Class-Orient My Code?

If you find yourself wanting to define classes, and subclasses which inherit from them, and if you're going to be instantiating one or more of those classes multiple times, then class-orientation is a good candidate.

<br>

## Keep It `class` y

`class` defines either a **declaration** or **expression** for a class.

- As a **declaration**, a class definition appears in a statement position and looks like this:

```js
class Point2d {
  // ..
}
```

- As an **expression**, a class definition appears in a value position and can either have a name or be anonymous:

```js
// named class expression
const pointClass = class Point2d {
  // ..
};
// anonymous class expression
const anotherClass = class {
  // ..
};
```

<br>

Inside a `class` body, methods are defined without the `function` keyword, and there's no `,` or `;` separators between the method definitions.

```js
class Point2d {
  setX(x) {
    // ..
  }

  setY(y) {
    // ..
  }
}
```

<br>

### The Constructor

One special method that all classes have is called a "`constructor`". If omitted, there's a default empty `constructor` assumed in the definition.
The `constructor` is invoked any time a `new` instance of the `class` is created:

```js
class Point2d {
  constructor() {
    console.log("Here's your new instance!");
  }
}

var point = new Point2d();
// Here's your new instance!
```

<br>

Even though the syntax implies a function actually named `constructor` exists, JS defines a function as specified, but with the name of the `class` ( `Point2d` above):

```js
typeof Point2d; // "function"
```

<br>

### Class Methods

```js
class Point2d {
  constructor() {
    console.log("Here's your new instance!");
  }

  setX(x) {
    console.log(`Setting x to: ${x}`);
    // ..
  }
}

var point = new Point2d();

point.setX(3);
// Setting x to: 3
```

`setX(..)` only exists as `Point2d.prototype.setX` . Since point is `[[Prototype]]` linked to `Point2d.prototype` via the `new` keyword instantiation, the `point.setX(..) `reference traverses the `[[Prototype]`] chain and finds the method to execute.

`Class` methods should only be invoked via an instance; `Point2d.setX(..)` doesn't work because there is no such property.

<br>

## Class Instance `this`

The `this` keyword generally refers to the current instance that is the context of any method invocation.

Any properties not holding function values, which are added to a `class` instance (usually via the `constructor`), are referred to as **members**,

<br>

### Public Fields

Just like computed property names, field names can be computed:

```js
var coordName = "x";

class Point2d {
  // computed public field
  [coordName.toUpperCase()] = 42;
  // ..
}

var point = new Point2d(3, 4);

point.x; // 3
point.y; // 4

point.X; // 42
```

<br>

## Class Extension

The way to unlock the power of class inheritance is through the `extends` keyword, which defines a relationship between two classes:

```js
class Point2d {
  x = 3;
  y = 4;
  getX() {
    return this.x;
  }
}

class Point3d extends Point2d {
  x = 21;
  y = 10;
  z = 5;
  printDoubleX() {
    console.log(`double x: ${this.getX() * 2}`);
  }
}

var point = new Point2d();
point.getX(); // 3

var anotherPoint = new Point3d();
anotherPoint.getX(); // 21
anotherPoint.printDoubleX(); // double x: 42
```

<br>

### Extending Expressions

// TODO this part of the book is unfinished!

<br>

### Overriding Methods

If you want to access an inherited method from a subclass even if it's been overridden, you can use `super` instead of `this`:

```js
class Point2d {
  x = 3;
  y = 4;

  getX() {
    return this.x;
  }
}

class Point3d extends Point2d {
  x = 21;
  y = 10;
  z = 5;

  getX() {
    return this.x * 2;
  }

  printX() {
    console.log(`x: ${super.getX()}`);
  }
}

var point = new Point3d();
point.printX(); // x: 21
```

**Method polymorphism**:
The ability for methods of the same name, at different levels of the inheritance hierarchy, to exhibit different behavior when either accessed directly, or relatively with `super`, is called **method polymorphism**.

<br>

### That's Super!

In addition to a subclass method accessing an inherited method definition (even if overridden on the subclass) via `super`. reference, a subclass `constructor` must manually invoke the inherited base class constructor via `super(..)` function invocation:

```js
class Point2d {
  x;
  y;
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class Point3d extends Point2d {
  z;
  constructor(x, y, z) {
    super(x, y);
    this.z = z;
  }

  toString() {
    console.log(`(${this.x},${this.y},${this.z})`);
  }
}

var point = new Point3d(3, 4, 5);
point.toString(); // (3,4,5)
```

<br>

- An explicitly defined subclass constructor must call `super(..)` to run the inherited class's initialization, and that must occur before the subclass constructor makes any references to `this` or finishes/returns.

- If you omit the subclass constructor, the default constructor automatically invokes `super()` for you.

<br>

If you define a field (public or private) inside a subclass, and explicitly define a `constructor(..)` for this subclass, the field initializations will be processed not at the top of the constructor, but between the `super(..)` call and any subsequent code in the constructor.

```js
class Point2d {
  x;
  y;
  constructor(x, y) {
    console.log("Running Point2d(..) constructor");
    this.x = x;
    this.y = y;
  }
}

class Point3d extends Point2d {
  z = console.log("Initializing field 'z'");

  constructor(x, y, z) {
    console.log("Running Point3d(..) constructor");
    super(x, y);

    console.log(`Setting instance property 'z' to ${z}`);
    this.z = z;
  }
  toString() {
    console.log(`(${this.x},${this.y},${this.z})`);
  }
}
var point = new Point3d(3, 4, 5);
// Running Point3d(..) constructor
// Running Point2d(..) constructor
// Initializing field 'z'
// Setting instance property 'z' to 5
```

<br>

### Which Class?

You may need to determine in a constructor if that class is being instantiated directly, or being instantiated from a subclass with a `super()` call. We can use a special "pseudo property" `new.Target` :

```js
class Point2d {
  // ..
  constructor(x, y) {
    if (new.target === Point2) {
      console.log("Constructing 'Point2d' instance");
    }
  }
}

class Point3d extends Point2d {
  // ..
  constructor(x, y, z) {
    super(x, y);
    if (new.target === Point3d) {
      console.log("Constructing 'Point3d' instance");
    }
  }
}

var point = new Point2d(3, 4);
// Constructing 'Point2d' instance

var anotherPoint = new Point3d(3, 4, 5);
// Constructing 'Point3d' instance
```

<br>

### But Which Kind Of Instance?

You may want to introspect a certain object instance to see if it's an instance of a specific class. We do this with the `instanceof` operator:

```js
class Point2d {
  //...
}

class Point3d extends Point2d {
  // ...
}

var point = new Point2d(3, 4);
point instanceof Point2d; // true
point instanceof Point3d; // false

var anotherPoint = new Point3d(3, 4, 5);
anotherPoint instanceof Point2d; // true
anotherPoint instanceof Point3d; // true
```

If you instead wanted to check if the object instance was only and directly created by a certain class, check the instance's constructor property.

```js
point.constructor === Point2d; // true
point.constructor === Point3d; // false

anotherPoint.constructor === Point2d; // false
anotherPoint.constructor === Point3d; // true
```

<br>

### "Inheritance" Is Sharing, Not Copying

```js
class Point2d {
  x;
  y;
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class Point3d extends Point2d {
  z;
  constructor(x, y, z) {
    super(x, y);
    this.z = z;
  }
  toString() {
    console.log(`(${this.x},${this.y},${this.z})`);
  }
}

var anotherPoint = new Point3d(3, 4, 5);
```

If you inspect the `anotherPoint` object, you'll see it only has the `x` , `y` , and `z` properties (instance members) on it, but not the `toString()` method:

```js
Object.hasOwn(anotherPoint, "x"); // true
Object.hasOwn(anotherPoint, "y"); // true
Object.hasOwn(anotherPoint, "z"); // true
Object.hasOwn(anotherPoint, "toString"); // false
```

Where is that `toString()` method located? On the prototype object:

```js
Object.hasOwn(Point3d.prototype, "toString"); // true
```

And `anotherPoint` has access to that method via its `[[Prototype]]` linkage. In other words, the prototype objects share access to their method(s) with the subclass(es) and instance(s). The method(s) stay in place, and are not copied down the inheritance chain.

<br>

- As nice as the class syntax is, don't forget what's really happening under the syntax: JS is just wiring up objects to each other along a `[[Prototype]]` chain.

<br>

## Static Class Behavior

We've so far emphasized two different locations for data or behavior (methods) to reside:
on the constructor's prototype, or on the instance. But there's a third option: **on the constructor (function object) itself**.

All class-defined methods are "real" functions residing on the constructor's prototype, and you could therefore invoke them.

So, how does a `class` system enable defining such data and behavior that should be available with a class but independent of (unaware of) instantiated objects?
`Static properties and functions`.

We use the `static` keyword in our class bodies to distinguish these definitions:

```js
class Point2d {
  // class statics
  static origin = new Point2d(0, 0);

  static distance(point1, point2) {
    return Math.sqrt((point2.x - point1.x) ** 2 + (point2.y - point1.y) ** 2);
  }

  // instance members and methods
  x;
  y;
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return `(${this.x},${this.y})`;
  }
}

console.log(`Starting point: ${Point2d.origin}`);
// Starting point: (0,0)
var next = new Point2d(3, 4);

console.log(`Next point: ${next}`);
// Next point: (3,4)

console.log(`Distance: ${Point2d.distance(Point2d.origin, next)}`);
// Distance: 5
```

Don't forget that when you use the `class` syntax, the name `Point2d` is actually the name of a constructor function that JS defines. So `Point2d.origin` is just a regular property access on that function object. That's what I meant at the top of this section when I referred to a third location for storing things related to classes; in JS, `static` s are stored as properties on the constructor function. Take care not to confuse those with properties stored on the constructor's prototype (methods) and properties stored on the instance (members).

<br>

### Static Property Initializations

The value in a static initialization ( `static whatever = ..` ) can include `this` references, which refers to the class itself (actually, the constructor) rather than to an instance:

```js
class Point2d {
  // class statics
  static originX = 0;
  static originY = 0;
  static origin = new this(this.originX, this.originY);
  // ..
}
```

I don't recommend actually doing the `new this(..)` trick I've illustrated here. That's just for illustration purposes. The code would read more cleanly with `new Point2d(this.originX,this.originY)` , so prefer that approach.

- class static initializations always run immediately after the `class` has been defined.

<br>

Recently, the `static` keyword was extended so it can now define a block inside the class body for more sophisticated initialization of `static` s:

```js
class Point2d {
  // class statics
  static origin = new Point2d(0, 0);
  static distance(point1, point2) {
    return Math.sqrt((point2.x - point1.x) ** 2 + (point2.y - point1.y) ** 2);
  }

  // static initialization block (as of ES2022)
  static {
    let outerPoint = new Point2d(6, 8);
    this.maxDistance = this.distance(this.origin, outerPoint);
  }
  // ..
}
Point2d.maxDistance; // 10
```

The `let outerPoint = ..` here is not a special class feature; it's exactly like a normal `let` declaration in any normal block of scope.

<br>

### Static Inheritance

Class statics are inherited by subclasses (obviously, as statics!), can be overridden, and `super` can be used for base class references (and static function polymorphism), all in much the same way as inheritance works with instance members/methods:

```js
  class Point2d {
          static origin = /* .. */
          static distance(x,y) { /* .. */ }
          static {
                  // ..
                  this.maxDistance = /* .. */;
          }
          // ..
  }

  class Point3d extends Point2d {
          // class statics
          static origin = new Point3d(
                  // here, `this.origin` references wouldn't
                  // work (self-referential), so we use
                  // `super.origin` references instead
                  super.origin.x, super.origin.y, 0
          )

          static distance(point1,point2) {
                  // here, super.distance(..) is Point2d.distance(..),
                  // if we needed to invoke it
                  return Math.sqrt(
                          ((point2.x - point1.x) ** 2) +
                          ((point2.y - point1.y) ** 2) +
                          ((point2.z - point1.z) ** 2)
                  );
          }

          // instance members/methods
          z
          constructor(x,y,z) {
                  super(x,y); // <-- don't forget this line!
                  this.z = z;
          }

          toString() {
                  return `(${this.x},${this.y},${this.z})`;
          }
  }

  Point2d.maxDistance; // 10
  Point3d.maxDistance; // 10
```

Any time you define a subclass constructor, you'll need to call `super(..)` in it, usually as the first statement. I find that all too easy to forget.

The static "inheritance" is not a copying of these static properties/functions from base class to subclass; it's sharing via the `[[Prototype]]` chain.

<br>

## Private Class Behavior

Private members/methods are private **only to the class they're defined** in, and **are not inherited in any way by a subclass**.

<br>

### Private Members/Methods

```js
class Point2d {
  // statics
  static samePoint(point1, point2) {
    return point1.#ID === point2.#ID;
  }

  // privates
  #ID = null;
  #assignID() {
    this.#ID = Math.round(Math.random() * 1e9);
  }

  // publics
  x;
  y;

  constructor(x, y) {
    this.#assignID();
    this.x = x;
    this.y = y;
  }
}
```

- Unlike public fields/instance members, private fields/instance members must be declared in the `class` body.

- private fields can be re-assigned, they cannot be `delete` d from an instance, the way a public field/class member can.

<br>

### Subclassing + Privates

if you invoke an inherited method in a subclass, and that inherited method in turn accesses/invokes privates in its host (base) class, this works fine.

- The inherited function reference is the exact same function as the base function reference;

<br>

### Existence Check

Keep in mind that only the class itself knows about, and can therefore check for, such a private field/method.

You may want to check to see if a private field/method exists on an object instance. For example (as shown below), you may have a static function or method in a class, which receives an external object reference passed in. To check to see if the passed-in object reference is of this same class (and therefore has the same private members/methods in it), you basically need to do a "brand check" against the object. Such a check could be rather convoluted, because if you access a private field that doesn't already exist on the object, you get a JS exception thrown, requiring ugly try..catch logic.

There's a cleaner approach, so called an "**ergonomic brand check**", using the in keyword:

```js
class Point2d {
  // statics
  static samePoint(point1, point2) {
    // "ergonomic brand checks"
    if (#ID in point1 && #ID in point2) {
      return point1.#ID === point2.#ID;
    }

    return false;
  }

  // privates
  #ID = null;
  #assignID() {
    this.#ID = Math.round(Math.random() * 1e9);
  }

  // publics
  x;
  y;

  constructor(x, y) {
    this.#assignID();
    this.x = x;
    this.y = y;
  }
}

var one = new Point2d(3, 4);
var two = new Point2d(3, 4);
Point2d.samePoint(one, two); // false
Point2d.samePoint(one, one); // true
```

<br>

### Private Statics

Static properties and functions can also use # to be marked as private:

```js
class Point2d {
  static #errorMsg = "Out of bounds.";
  static #printError() {
    console.log(`Error: ${this.#errorMsg}`);
  }

  // publics
  x;
  y;
  constructor(x, y) {
    if (x > 100 || y > 100) {
      Point2d.#printError();
    }
    this.x = x;
    this.y = y;
  }
}
var one = new Point2d(30, 400);
// Error: Out of bounds.
```

- private statics are similarly not-inherited by subclasses just as private members/methods are not.

<br>

### Gotcha: Subclassing With Static Privates and `this`

```js
class Point2d {
  static #errorMsg = "Out of bounds.";
  static printError() {
    console.log(`Error: ${this.#errorMsg}`);
  }
  // ..
}

class Point3d extends Point2d {
  // ..
}

Point2d.printError();
// Error: Out of bounds.

Point3d.printError === Point2d.printError;
// true

Point3d.printError();
// TypeError: Cannot read private member #errorMsg
// from an object whose class did not declare it
```

The `printError()` static is inherited (shared via `[[Prototype]]` ) from `Point2d` to `Point3d` just fine, which is why the function references are identical.

Like the non-static snippet just above, you might have expected the `Point3d.printError()` static invocation to resolve via the `[[Prototype]]` chain to its original base class ( `Point2d` ) location, thereby letting it access the base class's `#errorMsg` static private.

**But it fails**, as shown by the last statement in that snippet.

**There's a fix, though**. In the static function, instead of `this.#errorMsg`, swap that for `Point2d.#errorMsg` , and now it works:

```js
class Point2d {
  static #errorMsg = "Out of bounds.";
  static printError() {
    // the fixed reference vvvvvv
    console.log(`Error: ${Point2d.#errorMsg}`);
  }
  // ..
}

class Point3d extends Point2d {
  // ..
}

Point2d.printError();
// Error: Out of bounds.

Point3d.printError();
// Error: Out of bounds. <-- phew, it works now!
```

If public static functions are being inherited, use the class name to access any private statics instead of using `this.` references. Beware that gotcha!

<br>

## Class Example

```js
class CalendarItem {
  static #UNSET = Symbol("unset");
  static #isUnset(v) {
    return v === this.#UNSET;
  }
  static #error(num) {
    return this[`ERROR_${num}`];
  }
  static {
    for (let [idx, msg] of [
      "ID is already set.",
      "ID is unset.",
      "Don't instantiate 'CalendarItem' directly.",
    ].entries()) {
      this[`ERROR_${(idx + 1) * 100}`] = msg;
    }
  }
  static isSameItem(item1, item2) {
    if (#ID in item1 && #ID in item2) {
      return item1.#ID === item2.#ID;
    } else {
      return false;
    }
  }
  #ID = CalendarItem.#UNSET;
  #setID(id) {
    if (CalendarItem.#isUnset(this.#ID)) {
      this.#ID = id;
    } else {
      throw new Error(CalendarItem.#error(100));
    }
  }
  description = null;
  startDateTime = null;
  constructor() {
    if (new.target !== CalendarItem) {
      let id = Math.round(Math.random() * 1e9);
      this.#setID(id);
    } else {
      throw new Error(CalendarItem.#error(300));
    }
  }
  getID() {
    if (!CalendarItem.#isUnset(this.#ID)) {
      return this.#ID;
    } else {
      throw new Error(CalendarItem.#error(200));
    }
  }
  getDateTimeStr() {
    if (this.startDateTime instanceof Date) {
      return this.startDateTime.toUTCString();
    }
  }
  summary() {
    console.log(
      `(${this.getID()}) ${this.description} at ${this.getDateTimeStr()}`
    );
  }
}
class Reminder extends CalendarItem {
  #complete = false; // <-- no ASI, semicolon needed
  [Symbol.toStringTag] = "Reminder";
  constructor(description, startDateTime) {
    super();
    this.description = description;
    this.startDateTime = startDateTime;
  }
  isComplete() {
    return !!this.#complete;
  }
  markComplete() {
    this.#complete = true;
  }
  summary() {
    if (this.isComplete()) {
      console.log(`(${this.getID()}) Complete.`);
    } else {
      super.summary();
    }
  }
}
class Meeting extends CalendarItem {
  #getEndDateTimeStr() {
    if (this.endDateTime instanceof Date) {
      return this.endDateTime.toUTCString();
    }
  }
  endDateTime = null; // <-- no ASI, semicolon needed
  [Symbol.toStringTag] = "Meeting";
  constructor(description, startDateTime, endDateTime) {
    super();
    this.description = description;
    this.startDateTime = startDateTime;
    this.endDateTime = endDateTime;
  }
  getDateTimeStr() {
    return `${super.getDateTimeStr()} - ${this.#getEndDateTimeStr()}`;
  }
}

var callMyParents = new Reminder(
  "Call my parents to say hi",
  new Date("July 7, 2022 11:00:00 UTC")
);
callMyParents.toString();
// [object Reminder]
callMyParents.summary();
// (586380912) Call my parents to say hi at
// Thu, 07 Jul 2022 11:00:00 GMT
callMyParents.markComplete();
callMyParents.summary();
// (586380912) Complete.
callMyParents instanceof Reminder;
// true
callMyParents instanceof CalendarItem;
// true
callMyParents instanceof Meeting;
// false
var interview = new Meeting(
  "Job Interview: ABC Tech",
  new Date("June 23, 2022 08:30:00 UTC"),
  new Date("June 23, 2022 09:15:00 UTC")
);
interview.toString();
// [object Meeting]
interview.summary();
// (994337604) Job Interview: ABC Tech at Thu,
// 23 Jun 2022 08:30:00 GMT - Thu, 23 Jun 2022
// 09:15:00 GMT
interview instanceof Meeting;
// true
interview instanceof CalendarItem;
//

interview instanceof Reminder;
// false
Reminder.isSameItem(callMyParents, callMyParents);
// true
Meeting.isSameItem(callMyParents, interview);
// false
```
