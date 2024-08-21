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
