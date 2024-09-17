# You Don't Know JS : Types & Grammar <ins>**_Summary_**</ins> - 1st Edition

<br>

# Chapter 4: Coercion

<br><br>

## Converting Values

Converting a value from one type to another is often called “type casting” when done explicitly, and “coercion” when done implicitly (forced by the rules of how a value is used).

- It may not be obvious, but JavaScript coercions always result in one of the scalar primitive values, like `string`, `number`, or `boolean`. There is no coercion that results in a complex value like object or function.

However, in JavaScript, most people refer to all these types of conversions as coercion, so the way I prefer to distinguish is to say “implicit coercion” versus “explicit coercion.”

The difference should be obvious: “explicit coercion” is when it is obvious from looking at the code that a type conversion is intentionally occurring, whereas “implicit coercion” is when the type conversion will occur as a less obvious side effect of some other intentional operation.

```js
var a = 42;
var b = a + ""; // implicit coercion
var c = String(a); // explicit coercion
```

<br><br>

## Abstract Value Operations

<br>

### ToString

For regular objects, unless you specify your own, the default `toString()` (located in `Object.prototype.toString()`) will return the internal `[[Class]]` , like for instance "`[object Object]`".

Arrays have an overridden default `toString()` that stringifies as the (string) concatenation of all its values (each stringified themselves), with "," in between each value:

```js
var a = [1, 2, 3];
a.toString(); // "1,2,3"
```

<br>

#### JSON stringification
