# You Don't Know JS Yet: Scope & Closures <ins>**_Summary_**</ins> - 2nd Edition

<br>

# Appendix A: Exploring Further

<br>

## Implied Scopes

### Parameter Scope

```js
// outer/global scope: RED(1)
function getStudentName(studentID) {
  // function scope: BLUE(2)
  // ..
}
```

Here, `studentID` is a considered a “simple” parameter, so it does behave as a member of the BLUE(2) function scope. But if we change it to be a non-simple parameter, that’s no longer technically the case. Parameter forms considered non-simple include parameters with **default values**, **rest parameters** (using **`...`**), and **destructured parameters**.

```js
function getStudentName(studentID = maxID, maxID) {
  // ..
}
```

Assuming left-to-right operations, the default `= maxID` for the `studentID` parameter requires a `maxID` to already exist (and to have been initialized). This code produces a TDZ error. The reason is that `maxID` is declared in the parameter scope, but it’s not yet been initialized because of the order of parameters. If the parameter order is flipped, no TDZ error occurs:

```js
function getStudentName(maxID, studentID = maxID) {
  // ..
}
```

```js
function whatsTheDealHere(id, defaultID = () => id) {
  var id = 5;
  console.log(defaultID());
}
whatsTheDealHere(3);
// 3
```

The `var id = 5` is shadowing the `id` parameter, but the closure of the `defaultID()` function is over the parameter, not the shadowing variable in the function body.

<ins>This proves **there’s a scope bubble around the parameter list**.</ins>
