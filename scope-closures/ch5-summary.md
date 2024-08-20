# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 5: The (Not So) Secret Lifecycle of Variables

<br>

## When Can I Use a Variable?

Every identifier is created at the beginning of the scope it belongs to, every time that scope is entered.

**Hoisting**:
The term most commonly used for a variable being visible from the beginning of its enclosing scope, even though its declaration may appear further down in the scope, is called <ins>**hoisting**<ins>.

<br>

**Function hoisting**:
When a `function` declaration’s name identifier is registered at the top of its scope, it’s additionally auto-initialized to that function’s reference. That’s why the function can be called throughout the entire scope!

- One key detail is that both _function hoisting_ and _var-flavored_ variable hoisting attach their name identifiers to the nearest enclosing **function scope** (or, if none, the global scope), not a block scope.

- Declarations with `let` and `const` still hoist. But these two declaration forms attach to their enclosing block rather than just an enclosing function as with `var` and `function` declarations.

<br>

### Hoisting: Declaration vs. Expression

- Function hoisting only applies to formal function declarations (specifically those which appear outside of blocks), not to `function` expression assignments.

> Function declaration: `function doStuff() {};`

> Function expression: `const doStuff = function() {}`

- In addition to being hoisted, variables declared with `var` are also automatically initialized to `undefined` at the beginning of their scope.

- A `function` declaration is hoisted and initialized to its function value (again, called **function hoisting**). A `var` variable is also hoisted, and then auto-initialized to `undefined`. Any subsequent function expression assignments to that variable don’t happen until that assignment is processed during runtime execution.

<br>

### Variable Hoisting

- the identifier is hoisted,
- and it’s automatically initialized to the value `undefined` from the top of the scope.

<br>

### Hoisting: Yet Another Metaphor

The “rule” of the hoisting metaphor is that function declarations are hoisted first, then variables are hoisted immediately after all the functions.

> Function declaration: `function doStuff() {};`

> Function expression: `const doStuff = function() {}`

<br>

**What is hoisting**: <br>
Hoisting should be used to refer to the **compile time operation** of **generating runtime instructions** for the **automatic registration of a variable at the beginning of its scope**, <ins>**each time that scope is entered**</ins>.

<br>

### Re-declaration

- A repeated `var` declaration of the same identifier name in a scope is effectively a do-nothing operation.

- Two same declarations involving `let` will throw **SyntaxError** error. If either declaration uses `let`, the other can be either `let` or `var` or `function`, and **the error will still occur**.

- the only way to “re-declare” a variable is to use `var` for all (two or more) of its declarations.

<br>

### Constants

> [!WARNING]
> .**Syntax errors** represent faults in the program that **stop it from even starting execution**.
> **Type errors** represent faults that arise **during program execution**.

<br>

### Loops

All the rules of scope (including “re-declaration” of `let` created variables) are applied per scope instance. In other words, each time a scope is entered during execution, everything resets.

- `const` can’t be used with the classic `for`-loop form because of the required re-assignment.

<br>

## Uninitialized Variables (aka, TDZ)

With `var` declarations, the variable is “hoisted” to the top of its scope. But it’s also automatically initialized to the `undefined` value, so that the variable can be used throughout the entire scope.

- `var` variable automatically initializes at the top of the scope, where `let` variable does not.

### What is Temporal Dead Zone (TDZ)?

period of time from the entering of a scope to where the auto-initialization of the variable occurs is: **Temporal Dead Zone (TDZ)**. The TDZ is the time window where a variable exists but is still uninitialized, and therefore cannot be accessed in any way.

- A `var` also has technically has a TDZ, but it’s zero in length and thus unobservable to our programs! Only `let` and `const` have an observable TDZ.

`let`/`const` declarations do not automatically initialize at the beginning of the scope, the way `var` does.

- **auto-registration** of a variable at the top of the scope (i.e., what I call **“hoisting”**) and **auto-initialization** at the top of the scope (to `undefined`) are distinct operations.

<br>

**TDZ** errors occur because `let`/`const` declarations **do hoist** their <ins>declarations</ins> to the top of their scopes, but unlike `var`, they defer the **auto-initialization** of their variables until the moment in the code’s sequencing where the original declaration appeared. **This window of time (hint: temporal), whatever its length, is the TDZ**.

- Always put your `let` and `const` declarations at the top of any scope. Shrink the TDZ window to zero (or near zero) length.
