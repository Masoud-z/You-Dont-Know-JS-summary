# You Don't Know JS Yet: Objects & Classes <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Chapter 4: This Works

<br>

The determination of what value (usually, object) `this` points at is not made at author time, but rather determined at runtime.

In fact, a single `this` -aware function can be invoked at least four different ways, and any of those approaches will end up assigning a different `this` for that particular function invocation.

<br>

## This Aware

If a function does not have `this` in it anywhere, then the rules of how `this` behaves don't affect that function in any way. But if it does have even a single `this` in it, then you absolutely cannot determine how the function will behave without figuring out, for each invocation of the function, what `this` will point to.

<br>

### So What Is This?
