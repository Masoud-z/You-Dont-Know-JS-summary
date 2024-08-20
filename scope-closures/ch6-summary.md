# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 6: Limiting Scope Exposure

<br>

## Least Exposure

### When variables used by one part of the program are exposed to another part of the program, via scope, there are three main hazards that often arise:

1. **Naming Collisions**

2. **Unexpected Behavior**: if you expose variables/functions whose usage is otherwise private to a piece of the program, it allows other developers to use them in ways you didn’t intend, which can violate expected behavior and cause bugs.

3. Unintended Dependency: if you expose variables/functions unnecessarily, it invites other developers to use and depend on those otherwise private pieces.

<br>

### What is The Principle of Least Exposure (POLE)?

POLE, as applied to variable/function scoping, essentially says, default to exposing the bare minimum necessary, keeping everything else as private as possible. Declare variables in as small and deeply nested of scopes as possible, rather than placing everything in the global (or even outer function) scope.

<br>

### Invoking Function Expressions Immediately

we’re defining a `function` expression that’s then immediately invoked. This common pattern has a name:

**Immediately Invoked Function Expression <ins>(IIFE)</ins>**.

An IIFE is useful when we want to create a scope to hide variables/functions. it can be used in any place in a JS program where an expression is allowed. An IIFE can be named, or unnamed/anonymous. And it can be standalone or, part of another statement.

- **Always surround an IIFE function with `( .. )`**.

- If the code you need to wrap a scope around has `return`, `this`, `break`, or `continue` in it, an **IIFE** is probably not the best approach. In that case, you might look to create the scope with a block instead of a function.

<br>

## Scoping with Blocks

- In general, any `{ .. }` curly-brace pair which is a statement will act as a block, but not necessarily as a scope.

A block only becomes a scope if necessary, to contain its block-scoped declarations (i.e., `let` or `const`).

Not all `{ .. }` curly-brace pairs create blocks (and thus are eligible to become scopes):

- **Object literals** use `{ .. }` curly-brace pairs to delimit their key-value lists, but such object values are not scopes.
- `class` uses `{ .. }` curly-braces around its body definition, but this is not a block or scope.
- A `function` uses `{ .. }` around its body, but this is not technically a block—it’s a single statement for the function body. It is, however, a (function) scope.
- The `{ .. }` curly-brace pair on a `switch` statement (around the set of case clauses) does not define a block/scope.

<br>

**Other than such non-block examples, a `{ .. }` curly-brace pair can define a block attached to a statement (like an `if` or `for`), or stand alone by itself**

- An explicit block of this sort—if it has no declarations, it’s not actually a scope— serves no operational purpose, though it can still be useful as a semantic signal.

- To minimize the risk of **TDZ** errors with `let`/`const` declarations, always put those declarations at the top of their scope.

<br>

## var and let

- `var` attaches to the nearest enclosing function scope, no matter where it appears.
- `var` should be reserved for use in the top-level scope of a function.
- If a declaration belongs in a block scope, use `let`. If it belongs in the function scope, use `var` (again, just my opinion).

<br>

## What’s the Catch?

In `try..catch` clause, the `catch` clause has used an additional (little-known) block scoping declaration capability ( in "`catch(err){…}`" clause the "err" variable declared by the `catch` clause is block-scoped to that block.)

- ES2019 changed catch clauses so their declaration is optional; if the declaration is omitted, the `catch` block is no longer (by default) a scope; it’s still a block, though!

<br>

## Function Declarations in Blocks (FiB)

The JS specification says that function declarations inside of blocks are block-scoped.

- Most browser-based JS engines (including v8, which comes from Chrome but is also used in Node) will behave as the identifier is scoped outside the block but the function value is not automatically initialized, so it remains `undefined`.

Never place a function declaration directly inside any block. Always place **function declarations** anywhere in the top-level scope of a function (or in the global scope).

> Function declaration: `function doStuff() {};`
> Function expression: `const doStuff = function() {}`
