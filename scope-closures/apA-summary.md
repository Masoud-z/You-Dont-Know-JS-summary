# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Appendix A: Exploring Further

<br>

## Implied Scopes

### Parameter Scope

```js
// outer/global scope: RED(1)
function getStudentName(studentID) {
  // function scope: BLUE(2)
  // ..
}
```

Here, `studentID` is a considered a “simple” parameter, so it does behave as a member of the BLUE(2) function scope. But if we change it to be a non-simple parameter, that’s no longer technically the case. Parameter forms considered non-simple include parameters with **default values**, **rest parameters** (using **`...`**), and **destructured parameters**.

```js
function getStudentName(studentID = maxID, maxID) {
  // ..
}
```

Assuming left-to-right operations, the default `= maxID` for the `studentID` parameter requires a `maxID` to already exist (and to have been initialized). This code produces a TDZ error. The reason is that `maxID` is declared in the parameter scope, but it’s not yet been initialized because of the order of parameters. If the parameter order is flipped, no TDZ error occurs:

```js
function getStudentName(maxID, studentID = maxID) {
  // ..
}
```

<br><br><br>

```js
function whatsTheDealHere(id, defaultID = () => id) {
  var id = 5;
  console.log(defaultID());
}
whatsTheDealHere(3);
// 3
```

The `var id = 5` is shadowing the `id` parameter, but the closure of the `defaultID()` function is over the parameter, not the shadowing variable in the function body.

<ins>This proves **there’s a scope bubble around the parameter list**.</ins>

<br><br><br>

```js
function whatsTheDealHere(id, defaultID = () => id) {
  var id;
  console.log(`local variable 'id': ${id}`);
  console.log(`parameter 'id' (closure): ${defaultID()}`);
  console.log("reassigning 'id' to 5");
  id = 5;
  console.log(`local variable 'id': ${id}`);
  console.log(`parameter 'id' (closure): ${defaultID()}`);
}

whatsTheDealHere(3);
// local variable 'id': 3 <--- Huh!? Weird!
// parameter 'id' (closure): 3
// reassigning 'id' to 5
// local variable 'id': 5
// parameter 'id' (closure): 3
```

The strange bit here is the first console message. At that moment, the shadowing `id` local variable has just been `var id` declared, which Chapter 5 asserts is typically auto-initialized to `undefined` at the top of its scope. Why doesn’t it print `undefined`?

JS doesn’t auto-initialize `id` to `undefined`, but rather to the value of the `id` parameter (3)! Though the two `id`s look at that moment like they’re one variable, **they’re actually still separate (and in separate scopes)**.

The `id = 5` assignment makes the divergence observable, where the `id` parameter stays 3 and the local variable becomes 5.

-Never shadow parameters with local variables

-Avoid using a default parameter function that closes over any of the parameters

**The parameter list is its own scope if any of the parameters are non-simple.**

<br> <br>

### Function Name Scope

```js
var askQuestion = function ofTheTeacher() {
  // ..
};
```

It’s true that `ofTheTeacher` is not added to the enclosing scope (where `askQuestion` is declared), but it’s also not just added to the scope of the function,

**The name identifier of a function expression is in its own implied scope**, nested between the outer enclosing scope and the main inner function scope.

> Function declaration: `function doStuff() {};`

> Function expression: `const doStuff = function() {}`

If `ofTheTeacher` was in the function’s scope, we’d expect an error here:

```js
var askQuestion = function ofTheTeacher() {
  // why is this not a duplicate declaration error?
  let ofTheTeacher = "Confused, yet?";
};
```

The `let` declaration form does not allow re-declaration. But this is perfectly legal shadowing, not re-declaration, because the two `ofTheTeacher` identifiers are in separate scopes.

<br>

**Never shadow function name identifiers.**

<br><br>

## Anonymous vs. Named Functions

As you contemplate naming your functions, consider:

- Name inference is incomplete
- Lexical names allow self-reference
- Names are useful descriptions
- Arrow functions have no lexical names
- IIFEs also need names

<br>

### Arrow Functions

**Arrow functions are always anonymous**, even if (rarely) they’re used in a way that gives them an inferred name.

<br>

Arrow functions have a purpose:

Briefly: **arrow functions don’t define a `this` identifier keyword at all**. If you use a `this` inside an arrow function, it behaves exactly as any other variable reference, which is that the scope chain is consulted to find a function scope (non arrow function) where it is defined, and to use that one.

<br>

## Hoisting: Functions and Variables

**Even though let and const hoist, you cannot use those variables in their TDZ.**

<br>

## The Case for var

<br>

### The Case for var

`var` has a few other characteristics that, in certain limited circumstances, make it more powerful.
One example is when a loop is exclusively using a variable, but its conditional clause cannot see block-scoped declarations inside the iteration:

```js
function commitAction() {
  do {
    let result = commit();
    var done = result && result.code == 1;
  } while (!done);
}
```

Here, result is clearly only used inside the block, so we use let. But done is a bit different. It’s only useful for the loop, but the while clause cannot see let declarations that appear inside the loop. So we compromise and use var, so that done is hoisted to the outer scope where it can be seen.

<br>

Another helpful characteristic of var is seen with declarations inside unintended blocks.

```js
function getStudents() {
  try {
    // not really a block scope
    var records = fromCache("students");
  } catch (err) {
    // oops, fall back to a default
    var records = [];
  }
  // ..
}
```

<br>

## Are Synchronous Callbacks Still Closures?

- Closure is a function instance remembering its outer variables even as that function is passed around and invoked in other scopes.
- Closure is a function instance and its scope environment being preserved in-place while any references to it are passed around and invoked from other scopes.

<br>

### Synchronous Callback?

**Dependency Injection (DI)** can be summarized as passing in necessary part(s) of functionality to another part of the program so that it can invoke them to complete its work.
_Inversion of Control (IoC )_ means that instead of the current area of your program controlling what’s happening, you hand control off to another part of the program.

In the context of our discussion, either DI or IoC could work as an alternative label for a synchronous callback.

<br>

Let’s refer to (the functions formerly known as) synchronous callbacks, **as inter-invoked functions (IIFs).**

Yes, exactly, I’m playing off IIFEs.

#### Inter-invoked functions (IIFs) :

These kinds of functions are inter-invoked, meaning: another entity invokes them, as opposed to IIFEs, which invoke themselves immediately.

vet’s refer to (the functions formerly known as) synchronous callbacks, as inter-invoked functions (IIFs)

An asynchronous callback is an IIF that’s invoked asynchronously instead of synchronously.
