# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 2: Illustrating Lexical Scope

<br>

- scope is determined during code compilation, a model called “**lexical scope**”. <ins>The term “**lexical**” refers to the first stage of compilation (lexing/parsing)</ins>.

- Scope bubbles are determined during compilation based on where the functions/blocks of scope are written,

- Each scope bubble is entirely contained within its parent scope bubble.

- Scopes can be lexically nested to any arbitrary depth as the program defines.

- Each scope gets its own Scope Manager instance each time that scope is executed (one or more times). **Each scope automatically has all its identifiers registered at the start of the scope being executed (<ins>this is called “variable hoisting”</ins>)**

<br>

## What is variable hoisting?

Each scope automatically has all its identifiers registered at the start of the scope being executed, this is called “**variable hoisting**”.

- At the beginning of a scope, if any identifier came from a function declaration, that variable is automatically initialized to its associated function reference. And if any identifier came from a var declaration (as opposed to let/const), that variable is automatically initialized to undefined so that it can be used; otherwise, the variable remains uninitialized and cannot be used until its full declaration-and-initialization are executed.

- One of the key aspects of lexical scope is that any time an identifier reference cannot be found in the current scope, the next outer scope in the nesting is consulted; that process is repeated until an answer is found or there are no more scopes to consult.

- “`Not defined`” really means “**not declared**”—or, rather, “**undeclared**”, <ins>as in a variable that has no matching formal declaration in any lexically available scope</ins>. By contrast, “`undefined`” really means a variable was found (**declared**), but the variable otherwise has no other value in it at the moment, so it defaults to the `undefined` value
