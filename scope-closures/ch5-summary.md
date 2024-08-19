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

- In addition to being hoisted, variables declared with `var` are also automatically initialized to `undefined` at the beginning of their scope.

- A `function` declaration is hoisted and initialized to its function value (again, called **function hoisting**). A `var` variable is also hoisted, and then auto-initialized to `undefined`. Any subsequent function expression assignments to that variable don’t happen until that assignment is processed during runtime execution.

<br>

### Variable Hoisting

- the identifier is hoisted,
- and it’s automatically initialized to the value `undefined` from the top of the scope.

<br>

### Hoisting: Yet Another Metaphor

The “rule” of the hoisting metaphor is that function declarations are hoisted first, then variables are hoisted immediately after all the functions.

<br>

**What is hoisting**: <br>
Hoisting should be used to refer to the **compile time operation** of **generating runtime instructions** for the **automatic registration of a variable at the beginning of its scope**, <ins>**each time that scope is entered**</ins>.
