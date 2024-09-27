# You Don't Know JS : Types & Grammar <ins>**_Summary_**</ins> - 1st Edition

<br>

# Chapter 5: Grammar

<br><br>

## Statements & Expressions

A “sentence” is one complete formation of words that expresses a thought. It’s comprised of one or more “phrases,” each of which can be connected with punctuation marks or conjunctions (“and,” “or,” etc.).

And so it goes with JavaScript grammar. **Statements** are sentences, **expressions** are phrases, and **operators** are conjunctions/punctuation.

<br>

```js
var a = 3 * 6;
var b = a;
b;
```

In this snippet, `3 * 6` is an **expression** (evaluates to the value 18). But `a` on the second line is also an **expression**, as is `b` on the third line.
Moreover, each of the three lines is a **statement** containing expressions. `var a = 3 * 6` and `var b = a` are called “**declaration statements**”
The `a = 3 * 6` and `b = a` assignments (minus the `var`s) are called **assignment expressions**.

The third line contains just the expression `b`, but **it’s also a statement all by itself**. As such, this is generally referred to as an “**expression statement**.

<br>

### Statement Completion Values

**statements** all have completion values (even if that value is just `undefined`).

**How would you even go about seeing the completion value of a statement?**
The most obvious answer is to type the statement into your browser’s developer console, because when you execute it, the console by default reports the completion value of the most recent statement it executed.

any regular `{ .. }` block has a completion value of the completion value of its last contained statement/expression.

```js
var b;
if (true) {
  b = 4 + 38;
}
```

If you typed that into your console/REPL, you’d probably see `42` reported, since `42` is the completion value of the if block

- We can’t capture the completion value of a statement and assign it into another variable in any easy syntactic/grammatical way

<br>

We could use the much maligned `eval(..)` (sometimes pronounced “evil”) function to capture this completion value:

```js
var a, b;
a = eval("if (true) { b = 4 + 38; }");
a; // 42
```

There’s a proposal for ES7 called the “do expression.” instead of `eval(…)` to do this.

### Expression Side Effects

**Most expressions don’t have side effects**.

The most common example of an expression with (possible) side effects is a function call expression:

```js
function foo() {
  a = a + 1;
}

var a = 1;
foo(); // result: `undefined`, side effect: changed `a`
```

<br>

There are other side-effecting expressions, though. For example:

```js
var a = 42;
var b = a++;

a; // 43
b; // 42
```

Would you think `++a++` was legal syntax? If you try it, you’ll get a `ReferenceError` error, but why? Because side-effecting operators require a variable reference to target their side effects to. For `++a++`, the `a++` part is evaluated first (because of operator precedence), which gives back the value of a before the increment. But then it tries to evaluate `++42`, which (if you try it) gives the same `ReferenceError` error, since **`++` can’t have a side effect directly on a value like `42`**.

<br>

The **,** statement-series comma operator. This operator allows you to string together multiple standalone expression statements into a single statement:

```js
var a = 42,
  b;
b = (a++, a);
a; // 43
b; // 43
```

The expression `a++`, `a` means that the second `a` statement expression gets evaluated after the after side effects of the `a++` expression, which means it returns the `43` value for assignment to `b`.

- Another example of a side-effecting operator is `delete`.

`delete` is used to remove a property from an object or a slot from an array.

```js
var obj = {
  a: 42,
};
obj.a; // 42
delete obj.a; // true
obj.a; // undefined
```

The result value of the `delete` operator is `true` if the requested operation is valid/allowable, or `false` otherwise. But the side effect of the operator is that it removes the property (or array slot).

<br>

**What do we mean by valid/allowable**? **Nonexistent properties**, or **properties that exist and are configurable** will return `true` from the `delete` operator. Otherwise, the result will be `false` or an error.

```js
var obj = {
  a: 42,
};
obj.a; // 42

delete obj.b; // true
delete obj.c; // true
```

<br>

One last example of a side-effecting operator, which may at once be both obvious and nonobvious, is the `=` assignment operator.

```js
var a;
a = 42; // 42
a; // 42
```

It may not seem like `=` in `a = 42` is a side-effecting operator for the expression. But if we examine the result value of the `a = 42` statement, it’s the value that was just assigned (`42`), so the assignment of that same value into `a` is essentially a side effect.

<br>

The same reasoning about side effects goes for the compound-assignment operators like `+=`, `-=`, etc.

<br>

This behavior that an assignment expression (or statement) results in the assigned value is primarily useful for chained assignments, such as:

```js
var a, b, c;
a = b = c = 42;
```

A common mistake developers make with chained assignments is like `var a = b = 42`. While this looks like the same thing, it’s not. If that statement were to happen without there also being a separate `var b` (somewhere in the scope) to formally declare `b`, then `var a = b = 42` would not declare `b` directly. Depending on strict mode, that would either throw an error or create an accidental global

<br>

Another scenario to consider:

```js
function vowels(str) {
  var matches;
  if (str) {
    // pull out all the vowels
    matches = str.match(/[aeiou]/g);
    if (matches) {
      return matches;
    }
  }
}
vowels("Hello World"); // ["e","o","o"]
```

This works, and many developers prefer such. But using an idiom where we take advantage of the assignment side effect, we can simplify by combining the two if statements into one:

```js
function vowels(str) {
  var matches;
  // pull out all the vowels
  if (str && (matches = str.match(/[aeiou]/g))) {
    return matches;
  }
}
vowels("Hello World"); // ["e","o","o"]
```

- The `( .. )` around `matches = str.match..` is required. The reason is operator precedence,

<br>

### Contextual Rules

There are quite a few places in the JavaScript grammar rules where the same syntax means different things depending on where/how it’s used.

#### Curly braces

There’s two main places (and more coming as JS evolves!) that a pair of curly braces `{ .. }` will show up in your code.

<br>

##### Object literals

```js
var a = {
  foo: bar(),
};
```

<br>

##### Labels

```js
{
  foo: bar();
}
```

Here,`{ .. }` is just a regular code block.
`foo` is a label for the statement `bar()`

But what’s the point of a labeled statement?
If JavaScript had a `goto` statement, you’d theoretically be able to say `goto foo` and have execution jump to that location in code.

However, JS does support a limited, special form of goto: **labeled jumps**. Both the `continue` and `break` statements can optionally accept a specified label, in which case the program flow “jumps” kind of like a goto.

```js
// `foo` labeled-loop
foo: for (var i = 0; i < 4; i++) {
  for (var j = 0; j < 4; j++) {
    // whenever the loops meet, continue outer loop
    if (j == i) {
      // jump to the next iteration of
      // the `foo` labeled-loop
      continue foo;
    }
    // skip odd multiples
    if ((j * i) % 2 == 1) {
      // normal (nonlabeled) `continue` of inner loop
      continue;
    }
    console.log(i, j);
  }
}
// 1 0
// 2 0
// 2 1
// 3 0
// 3 2
```

`continue foo` does not mean “go to the `foo` labeled position to continue,” but rather, “continue the loop that is labeled `foo` with its next iteration.” So, it’s not really an arbitrary goto

<br>

Perhaps a slightly more useful form of the labeled-loop jump is with `break`from inside an inner loop where you want to break out of the outer loop.

```js
// `foo` labeled-loop
foo: for (var i = 0; i < 4; i++) {
  for (var j = 0; j < 4; j++) {
    if (i * j >= 3) {
      console.log("stopping!", i, j);
      break foo;
    }
    console.log(i, j);
  }
}

// 0 0
// 0 1
// 0 2
// 0 3
// 1 0
// 1 1
// 1 2
// stopping! 1 3
```

break foo does not mean “go to the foo labeled position to continue,” but rather, “break out of the loop/block that is labeled foo and continue after it.”

<br>

A `label` can apply to a nonloop block, but only `break` can reference such a nonloop label.
You can do a labeled break out of any labeled block, but you cannot `continue` a nonloop label, nor can you do a nonlabeled break out of a block:

```js
// `bar` labeled-block
function foo() {
  bar: {
    console.log("Hello");
    break bar;
    console.log("never runs");
  }
  console.log("World");
}

foo();
// Hello
// World
```

It’s best to avoid them if possible; for example, by using function calls instead of the loop jumps. But there are perhaps some limited cases where they might be useful. If you’re going to use a labeled jump, make sure to document what you’re doing with plenty of comments!

<br>

- `JSON` is not valid JS grammar by itself.

One extremely common misconception along these lines is that if you were to load a JS file into a `<script src=..>` tag that only has `JSON` content in it (like from an API call), the data would be read as valid JavaScript but just be inaccessible to the program.

`JSON-P` (the practice of wrapping the JSON data in a function call, like `foo({"a":42})`) is usually said to solve this inaccessibility by sending the value to one of your program’s functions.

Not true! The totally valid `JSON` value y by itself would actually throw a JS error because it’d be interpreted as a statement block with an invalid label. But `foo({"a":42})` is valid JS because in it, `{"a":42}` is an object literal value being passed to `foo(..)`. So, properly said, `JSON-P` makes `JSON` into valid JS grammar!

<br>

##### Blocks

Another commonly cited JS gotcha is:

```
[] + {}; // "[object Object]"

{} + []; // 0
```

On the first line, `{}` appears in the `+` operator’s expression, and is therefore interpreted as an actual value (an empty object). Chapter 4 explained that `[]` is coerced to `""` and thus `{}` is coerced to a `string` value as well: "`[object Object]`".

But on the second line, `{}` is interpreted as a standalone `{}` empty block (which does nothing). **Blocks don’t need semicolons to terminate them**, so the lack of one here isn’t a problem. Finally, + [] is an expression that explicitly coerces the [] to a number, which is the 0 value.

<br>

##### Object destructuring

Starting with ES6, another place that you’ll see `{ .. }` pairs showing up is with “**destructuring assignments**”

```js
function getData() {
  // ..
  return {
    a: 42,
    b: "foo",
  };
}
var { a, b } = getData();
console.log(a, b); // 42 "foo"
```

`{ a, b }` is actually ES6 destructuring shorthand for` { a: a, b: b }`, so either will work, but it’s expected that the shorter `{ a, b }` will be become the preferred form.

Object destructuring with a `{ .. }` pair can also be used for named function arguments, which is sugar for this same sort of implicit object property assignment:

```js
function foo({ a, b, c }) {
  // no need for:
  // var a = obj.a, b = obj.b, c = obj.c
  console.log(a, b, c);
}

foo({
  c: [1, 2, 3],
  a: 42,
  b: "foo",
}); // 42 "foo" [1, 2, 3]
```

<br>

##### else if and optional blocks

It’s a common misconception that JavaScript has an else if clause, because you can do:

```js
if (a) {
  // ..
} else if (b) {
  // ..
} else {
  // ..
}
```

But there’s a hidden characteristic of the JS grammar here: there is no `else if`. But `if` and `else` statements are allowed to omit the `{ }` around their attached block if they only contain a single statement.

the `else if` form you’ve likely always coded is actually parsed as:

```js
if (a) {
  // ..
} else {
  if (b) {
    // ..
  } else {
    // ..
  }
}
```

The `if (b) { .. } else { .. }` is a single statement that follows the `else`, so you can either put the surrounding `{ }` in or not. In other words, when you use `else if`, you’re technically breaking that common style guide rule and just defining your `else` with a single `if` statement.

<br><br>

## Operator Precedence

JavaScript’s version of `&&` and `||` are interesting in that they select and return one of their operands, rather than just resulting in `true` or `false`.

The `&&` operator is evaluated first and the `||` operator is evaluated second.

<br>

### Short Circuited

For both `&&` and `||` operators, the righthand operand will not be evaluated if the lefthand operand is sufficient to determine the outcome of the operation. Hence, the name “**short circuited**” (in that if possible, it will take an early shortcut out).

For example, with `a && b`, `b` is not evaluated if `a` is falsy, because the result of the `&&` operand is already certain, so there’s no point in bothering to check `b`. Likewise, with `a || b`, if `a` is truthy, the result of the operand is already certain, so there’s no reason to check `b`.

```js
function doSomething(opts) {
  if (opts && opts.cool) {
    // ..
  }
}
```

The `opts` part of the `opts && opts.cool` test acts as sort of a guard, because if `opts` is unset (or is otherwise not an object), the expression `opts.cool` would throw an error. The `opts` test failing plus the short circuiting means that `opts.cool` won’t even be evaluated, thus no error!

<br>

Similarly, you can use `||` short circuiting:

```js
function doSomething(opts) {
  if (opts.cache || primeCache()) {
    // ..
  }
}
```

Here, we’re checking for `opts.cache` first, and if it’s present, we don’t call the `primeCache()` function, thus avoiding potentially unnecessary work.

<br>

### Tighter Binding

`&&` is more precedent than `||`, and `||` is more precedent than `? :`

```
a && b || c ? c || b ? a : c && b : a
```

Above code will operate like this:

```
 (a && b || c) ? (c || b) ? a : (c && b) : a
```

<br>

### Associativity

In general, operators are either **left-associative** or **right-associative**, referring to **whether grouping happens from the left or from the right**.

**left-associative or right-associative refer to grouping NOT processing**.

Technically, `a && b && c` will be handled as `(a && b) && c`, because `&&` is left-associative (so is `||`, by the way).

If hypothetically `&&` was right-associative, it would be processed the same as if you manually used `( )` to create a grouping like `a && (b && c)`. But that still doesn’t mean that `c` would be processed before `b`. **Right-associativity does not mean right-to-left evaluation, it means right-to left grouping**. Either way, regardless of the grouping/associativity, the strict ordering of evaluation will be `a`, then `b`, then `c` (aka left to right).

<br>

**`? :` is right-associative:**

```
a ? b : c ? d : e;
```

So the above code will be as:

```
a ? b : (c ? d : e)
```

<br>

Another example of right-associativity (grouping) is the `=` operator.

```js
var a, b, c;
a = b = c = 42;
```

We asserted earlier that `a = b = c = 42` is processed by first evaluating the `c = 42` assignment, then `b = ..`, and finally `a = ..`.
Why? Because of the right-associativity, which actually treats the statement like this: `a = (b = (c = 42))`.

<br>

```
var a = 42;
var b = "foo";
var c = false;

var d = a && b || c ? c || b ? a : c && b : a;

d; // 42
```

its grouping behavior like this:

```
((a && b) || c) ? ((c || b) ? a : (c && b)) : a
```

1. `(a && b)` is `"foo"`.
2. `"foo" || c` is `"foo"`.
3. For the first `?` test, `"foo"` is truthy.
4. `(c || b)` is `"foo"`.
5. For the second `?` test, `"foo"` is truthy.
6. `a` is `42`.

<br>

### Disambiguation

In my opinion, there’s an important middle ground. We should mix both operator precedence/associativity and `( )` manual grouping into our programs.

<br><br>

## Automatic Semicolons

ASI (Automatic Semicolon Insertion) is when JavaScript assumes a `;` in certain places in your JS program even if you didn’t put one there.

Basically, if the JS parser parses a line where a parser error would occur (a missing expected `;`), and it can reasonably insert one, it does so.
What’s reasonable for insertion? Only if there’s nothing but whitespace and/or comments between the end of some statement and that line’s newline/line break.

<br>

There’s certain places where ASI is helpful, like for instance:

```
var a = 42;
do {
  // ..
} while (a) // <-- ; expected here!
a;
```

The grammar requires a `;` after a `do..while` loop, but not after `while` or `for` loops. But most developers don’t remember that! So, ASI helpfully steps in and inserts one.

<br>

As we said earlier in the chapter, statement blocks do not require ; termination, so ASI isn’t necessary:

```js
var a = 42;
while (a) {
  // ..
} // <-- no ; expected here
a;
```

<br>

The other major case where ASI kicks in is with the `break`, `continue`, `return`, and (ES6) `yield` keywords.

```
function foo(a) {
  if (!a) return
  a *= 2;
}
```

The `return` statement doesn’t carry across the newline to the `a *= 2` expression, as ASI assumes the `;` terminating the `return` statement.

Identical reasoning applies to `break`, `continue`, and `yield`.

<br>

### Error Correction

Most, but not all, semicolons are optional, but **the two `;`s in the `for( .. )` .. loop header are required**.

<br><br>

## Errors

Not only does JavaScript have different subtypes of errors (`TypeError`, `ReferenceError`, `SyntaxError`, etc.), but also the grammar defines certain errors to be enforced at compile time, as compared to all other errors that happen during runtime.

In particular, there have long been a number of specific conditions that should be caught and reported as “early errors” (during compilation). Any straight-up syntax error is an early error (e.g., `a = ,`), but also the grammar defines things that are syntactically valid but disallowed nonetheless.

Since execution of your code has not begun yet, these errors are not catchable with `try..catch`; instead, they will just fail the parsing/compilation of your program.

```js
var a = /+foo/; // Error!
```

```js
var a;
42 = a; // Error!
```

```js
function foo(a, b, a) {} // just fine
function bar(a, b, a) {
  "use strict";
} // Error!
```

```js
(function () {
  "use strict";
  var a = {
    b: 42,
    b: 43,
  }; // Error!
})();
```

Semantically speaking, such errors aren’t technically syntax errors but more grammar errors— the above snippets are syntactically valid. But since there is no GrammarError type, some browsers use `SyntaxError` instead.

<br>

### Using Variables Too Early

The **TDZ** (“**Temporal Dead Zone**”) refers to places in code where a variable reference cannot yet be made, because it hasn’t reached its required initialization.

```js
{
  a = 2; // ReferenceError!
  let a;
}
```

Interestingly, while `typeof` has an exception to be safe for undeclared variables (see Chapter 1), no such safety exception is made for TDZ references:

```js
{
  typeof a; // undefined
  typeof b; // ReferenceError! (TDZ)
  let b;
}
```

<br><br>

## Function Arguments

```js
var b = 3;
function foo(a = 42, b = a + b + 5) {
  // ..
}
```

The `b` reference in the assignment would happen in the TDZ for the parameter `b` (not pull in the outer `b` reference), so it will throw an error. However, the `a` is fine since by that time it’s past the TDZ for parameter `a`.

When using ES6’s default parameter values, the default value is applied to the parameter if you either omit an argument, or you pass an `undefined` value in its place:

```js
function foo(a = 42, b = a + 1) {
  console.log(a, b);
}

foo(); // 42 43
foo(undefined); // 42 43
foo(5); // 5 6
foo(void 0, 7); // 42 7
foo(null); // null 1
```

- `null` is coerced to a `0` value in the `a + 1` expression.

<br>

From the ES6 default parameter values perspective, there’s no difference between omitting an argument and passing an `undefined` value. However, there is a way to detect the difference in some cases:

```js
function foo(a = 42, b = a + 1) {
  console.log(arguments.length, a, b, arguments[0], arguments[1]);
}

foo(); // 0 42 43 undefined undefined
foo(10); // 1 10 11 10 undefined
foo(10, undefined); // 2 10 11 10 undefined
foo(10, null); // 2 10 null 10 null
```

- Use of the arguments array has been deprecated (especially in favor of ES6 ... rest parameters.

<br><br>

## try..finally

The code in the `finally` clause always runs (no matter what), and it always runs right after the `try` (and `catch` if present) finish, before any other code runs.

```js
function foo() {
  try {
    console.log("First");
    return 42;
  } finally {
    console.log("Hello");
  }

  console.log("never runs");
}
console.log(foo());
// First
// Hello
// 42
```

The `return 42` runs right away, which sets up the completion value from the `foo()` call. This action completes the `try` clause and the `finally` clause immediately runs next. Only then is the `foo()` function complete, so that its completion value is returned back for the `console.log(..)` statement to use.

<br>

The exact same behavior is true of a `throw` inside `try`:

```js
function foo() {
  try {
    throw 42;
  } finally {
    console.log("Hello");
  }
  console.log("never runs");
}
console.log(foo());
// Hello
// Uncaught Exception: 42
```

Now, if an exception is thrown (accidentally or intentionally) inside a `finally` clause, it will override as the primary completion of that function. If a previous `return` in the `try` block had set a completion value for the function, that value will be abandoned:

```js
function foo() {
  try {
    return 42;
  } finally {
    throw "Oops!";
  }
  console.log("never runs");
}
console.log(foo());
// Uncaught Exception: Oops!
```

<br>

It shouldn’t be surprising that other nonlinear control statements like `continue` and `break` exhibit similar behavior to return and throw:

```js
for (var i = 0; i < 10; i++) {
  try {
    continue;
  } finally {
    console.log(i);
  }
}
// 0 1 2 3 4 5 6 7 8 9
```

The `console.log(i)` statement runs at the end of the loop iteration, which is caused by the `continue` statement. However, it still runs before the `i++` iteration update statement, which is why the values printed are `0..9` instead of `1..10`.

<br>

A `return` inside a `finally` has the special ability to override a previous `return` from the `try` or `catch` clause, but only if `return` is explicitly called:

```js
function foo() {
  try {
    return 42;
  } finally {
    // no `return ..` here, so no override
  }
}

function bar() {
  try {
    return 42;
  } finally {
    // override previous `return 42`
    return;
  }
}

function baz() {
  try {
    return 42;
  } finally {
    // override previous `return 42`
    return "Hello";
  }
}

foo(); // 42
bar(); // undefined
baz(); // Hello
```

<br><br>

## switch

In `switch` if a match is found, execution will begin in that matched `case`, and will either go until a `break` is encountered or until the end of the `switch` block is found.

```js
switch (a) {
  case 2:
    // do something
    break;
  case 42:
    // do another thing
    break;
  default:
  // fallback to here
}
```

First, the matching that occurs between the `a` expression and each `case` expression is identical to the `===` algorithm

<br>

Lastly, the `default` clause is optional, and it doesn’t necessarily have to come at the end (although that’s the strong convention). Even in the `default` clause, the same rules apply about encountering a break or not:

```js
var a = 10;
switch (a) {
  case 1:
  case 2:
  // never gets here
  default:
    console.log("default");
  case 3:
    console.log("3");
    break;
  case 4:
    console.log("4");
}
// default
// 3
```

Because it doesn't have `break` after default it continues to execute everything till it finds `break` or until the end of the switch block is found.
