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
