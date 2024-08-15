# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 1: What's the Scope

<br>

- JS is in fact parsed/compiled in a separate phase before execution begins.

- The code author’s decisions on where to place variables, functions, and blocks with respect to each other are analyzed according to the rules of scope, during the initial parsing/compilation phase.

<br>

### Interpretation and compilation

Unlike a program being compiled all at once, with interpretation the source code is transformed line by line; each line or statement is executed before immediately proceeding to processing the next line of the source code.

- Modern JS engines actually employ numerous variations of both compilation and interpretation in the handling of JS programs.
- JS is most accurately portrayed as a compiled language.
- Scope is primarily determined during compilation.

<br>

- the most important observation we can make about processing of JS programs is that it occurs in (at least) two phases: parsing/compilation first, then execution.

<br>

### What does it mean that JS functions are first-class values?

JS functions are themselves first-class values; they can be assigned and passed around just like numbers or strings.

**Other than declarations, all occurrences of variables/identifiers in a program serve in one of two “roles”: either they’re the target of an assignment or they’re the source of a value.**

<br>

## How do you know if a variable is a target?

Check if there is a value that is being assigned to it; if so, it’s a target. If not, then the variable is a source.

<br>

### Targets

there are three other target assignment operations in the code that are perhaps less obvious. One of them:

```js
for (let student of students) {
```

That statement assigns a value to `student` for each iteration of the loop. Another target reference:

```js
getStudentName(73);
```

Look closely: the argument 73 is assigned to the parameter `studentID`.

<br>
And there’s one last (subtle) target reference in our program.

```js
function getStudentName(studentID) {
```

A `function` declaration is a special case of a target reference. You can think of it sort of like `var getStudentName = function(studentID)`, but that’s not exactly accurate.
An identifier `getStudentName` is declared (at compile time),but the `= function(studentID)` part is also handled at compilation; the association between `getStudentName` and the `function` is automatically set up at the beginning of the scope rather than waiting for an `=` assignment statement to be executed.

<br>

### Sources

- In `for (let student of students)`, we said that `student` is a target, but `students` is a source reference.
- In the statement `if (student.id == studentID)`, both `student` and `studentID` are source references.
- `student` is also a source reference in return `student.name`.
- In `getStudentName(73)`, `getStudentName` is a source reference (which we hope resolves to a function reference value).
- In `console.log(nextStudent)`, `console` is a source reference, as is `nextStudent`.

> [!NOTE]
> In case you were wondering, `id`, `name`, and `log` are all properties, **not variable references**.

<br>

## Tokenizing/Lexing:

breaking up a string of characters into meaningful (to the language) chunks, called **tokens**.

If the tokenizer were to invoke stateful parsing rules to figure out whether `a` should be considered `a` distinct token or just part of another token, that would be **lexing**.

<br>

## Lexical Scope

<ins>**JS’s scope is determined at compile time**</ins>; the term for this kind of scope is “**lexical scope**”. “**Lexical**” is associated with the “**lexing**” stage of compilation.

- If you place a variable declaration inside a function, the compiler handles this declaration as it’s parsing the function, and associates that declaration with the function’s scope.

- If a variable is block-scope declared (let / const), then it’s associated with the nearest enclosing { .. } block, rather than its enclosing function (as with var).

<br>

- **while scopes are identified during compilation, they’re not actually created until runtime, each time a scope needs to run**.
