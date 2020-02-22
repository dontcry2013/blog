---
title: ES6
date: 2016-11-24 14:40:02
categories: JavaScript #文章文类
tags: [ES6] #文章标签，多于一项时用这种格式
---
# Default Parameter of Function

Prior to ES6, you could not directly specify default values ​​for function parameters, only workarounds could be used.

``` js
function log(x, y) {
  y = y || 'World';
  console.log(x, y);
}
log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello World
```
<!--more-->
The disadvantage is that if the parameter y is false, the assignment has no effect.
In order to avoid this problem, it is usually necessary to first determine whether the parameter y is assigned, and if not, it is equal to the default value.
``` js
if (typeof y === 'undefined') {
  y = 'World';
}
```

ES6 allows you to set default values ​​for function parameters, that is, write directly after the parameter definition.

``` js
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
```

# rest parameter

The variable in the rest parameter represents an array, so array-specific methods can be used for this variable. The following is an example of rewriting the array push method using the rest parameter.

``` js
function push(array, ...items) {
  items.forEach(function(item) {
    array.push(item);
    console.log(item);
  });
}

var a = [];
push(a, 1, 2, 3)
```

Note that there can be no other parameters after the rest parameter (that is, only the last parameter), otherwise an error will be reported.
The length property of the function, excluding the rest parameter.

``` js
(function(a) {}).length  // 1
(function(...a) {}).length  // 0
(function(a, ...b) {}).length  // 1
```

# Extension operator

The spread operator is three dots (...). It is like the inverse of the rest parameter, turning an array into a sequence of parameters separated by commas.

``` js
function add(x, y) {
  return x + y;
}

var numbers = [4, 38];
add(...numbers) // 42
```

# Replace array's apply method

``` js
// ES5 notation
Math.max.apply(null, [14, 3, 77])

// ES6 notation
Math.max(...[14, 3, 77])

// equivalent to
Math.max(14, 3, 77);
```

``` js
> var a = [1,2]
undefined
> var b = [3,4]
undefined
> a.push(b)
3
> a
[ 1, 2, [ 3, 4 ] ]
> a.push(...b)
5
> a
[ 1, 2, [ 3, 4 ], 3, 4 ]


// ES5 notation
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
Array.prototype.push.apply(arr1, arr2);

// ES6 notation
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
arr1.push(...arr2);
```

# Application of extended operators

## (1)Merge array
``` js
// ES5
[1, 2].concat(more)
// ES6
[1, 2, ...more]
```

## (1)String

Expansion operator can also turn strings into real arrays.

``` js
[...'hello']
// [ "h", "e", "l", "l", "o" ]
```

# Other
```js
foo::bar;
// equivalent to
bar.bind(foo);
```