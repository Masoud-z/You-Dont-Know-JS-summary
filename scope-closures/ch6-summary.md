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
