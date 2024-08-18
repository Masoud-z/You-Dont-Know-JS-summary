# You Don't Know JS Yet: Get Started <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Appendix A: Exploring Further

<br>

- primitive values are always assigned/passed as value copies

- In JS, only object values (arrays, objects, functions, etc.) are treated as references.

JS chooses the **value-copy** vs. **reference-copy** behavior based on the value type. **Primitives are held by value**, **objects are held by reference**.

- All functions by default reference an empty object at a property named prototype.

<br>

## So Many Function Forms

Keep in mind that arrow function expressions are syntactically anonymous, meaning the syntax doesn’t provide a way to provide a direct name identifier for the function.
