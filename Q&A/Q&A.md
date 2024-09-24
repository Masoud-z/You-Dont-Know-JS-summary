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

Parsing a numeric value out of a string is tolerant of non-numeric characters—it just stops parsing left-to-right when encountered— whereas coercion is not tolerant and fails, resulting in the NaN value.

---

<br><br>

---

## What is the difference between `a || b` || `a ? a : b` ?

`a || b` “**roughly equivalent**” to `a ? a : b` because **the outcome is identical**, _but there’s a nuanced difference_. In `a ? a : b`, if `a` was a more complex expression (_like for instance one that might have side effects like calling a function_, etc.), **then the `a` expression would possibly be evaluated twice (if the first evaluation was truthy)**. By contrast, **for `a || b`, the `a` expression is evaluated only once**, and that value is used both for the coercive test as well as the result value (if appropriate). The same nuance applies to the `a && b` and `a ? b : a` expressions.

---

<br><br>

---

## How does coercion work on loose equality when one side is a boolean? 

What is relevant is to understand how the `==` comparison algorithm behaves _with all the different type combinations_. As it regards a `boolean` value on either side of the `==`, **a `boolean` always coerces to a `number` first**.
