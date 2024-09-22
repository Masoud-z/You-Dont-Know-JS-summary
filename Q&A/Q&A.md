# Some Questions that You Don't Know JS Answers

<br><br>

---

## What is different about an `=>` arrow function?

The `=>` arrow function is not about typing fewer characters. **The primary point of the `=>` function being added to JS was to give us "lexical this" behavior**.

When a `this` is encountered inside an `=>` arrow function, `this` is treated like a normal lexical variable, not a special keyword.

---

<br><br>

---

## What will happen if we invoke an `=>` arrow function with `new` keyword?

It will actually throw an exception, because **an `=>` function cannot be used with the `new` keyword**.

**Conceptually and actually, an `=>` arrow function is not a function with a hard-bound `this` , it's a function that has no `this` at all. As such, `new` makes no sense against such a function, so JS just disallows it**.

---

<br><br>

---

## Various DOM query operations return lists of DOM elements that are not true arrays, how we can convert it to ad array?

There are different ways to do it but one of the easy ways is using **`Array.from(..)`**

```js
var arr = Array.from(arguments);
```

---

<br><br>

---

## What are the differences between Parsing Numeric Strings and the type conversion?

1. Parsing a numeric value out of a string is tolerant of non-numeric characters—it just stops parsing left-to-right when encountered— whereas coercion is not tolerant and fails, resulting in the NaN value.

2. If you pass a non-string, the value you pass will automatically be coerced to a string first, which would clearly be a kind of hidden implicit coercion.
