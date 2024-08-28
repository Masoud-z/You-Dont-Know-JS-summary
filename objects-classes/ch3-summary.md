# You Don't Know JS Yet: Objects & Classes <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 3: Classy Objects

<br>

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

    class Point2d {
            x
            y
            constructor(x,y) {
                    this.x = x;
                    this.y = y;
            }
    }

    class Point3d extends Point2d {
            z
            constructor(x,y,z) {
                    super(x,y);
                    this.z = z;
            }

            toString() {
                    console.log(`(${this.x},${this.y},${this.z})`);
            }
    }

    var point = new Point3d(3,4,5);
    point.toString(); // (3,4,5)
