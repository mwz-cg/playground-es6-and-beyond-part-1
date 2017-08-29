# The past

JavaScript *used to* be considered one of those "lesser" languages, kind of like PHP, that were deemed unsuitable for "real" programming. After all, yes, JavaScript is dynamically typed (the type of a variable is known/can vary at runtime), has implicit type conversion, and on top of that is indecently permissive/forgiving. Here are a few things that you can do without having runtime errors:
- add values of different types (such as numbers and strings),
- use a variable that has not been assigned before,
- reference non-existent properties of an object,
- call a function with less or more arguments than it expects.

This works because JavaScript has implicit type conversion, so it tries to make sense of what it is supposed to do when computing `1 + '2'` for instance, and also in addition to `null`, JavaScript has the `undefined` value that represents the result of an expression that is, well, not defined.

# The present

Anyway, so what changed since that time? Well, JavaScript caught up with other programming languages, especially with the release of the ECMAScript 6 standard (2015). This version introduced many changes and new features that make JavaScript more powerful and reduce the possibility for unintended behavior. In this playground I list the features that I have found to be the most useful on a daily basis.

## Use `let` not `var`

There are two problems with `var`. First, it has function scope even if it is declared in a nested block.

```javascript runnable
function problemWithVar() {
    for (var i of [0, 1, 2]) {
    }
    console.log(i);
```

This prints `2`. Change `var` to `let` and observe the difference.

The second problem is that within a function two declarations of a `var x` will in fact be the same variable. For instance:

```javascript runnable
function problemWithVar() {
    var x = 3;
    if (x > 2) {
        var x = 4;
    }
    console.log(x);
}
```

Try to use `let` and see what happens.

