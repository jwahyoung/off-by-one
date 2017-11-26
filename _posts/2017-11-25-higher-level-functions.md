---
uid: 4d36d722-2bf5-415e-ae54-17a37798b691
layout: post
title: Exploring Array.forEach and Array.map
description: ""
date: 2017-11-25
tags: javascript function map foreach array
published: true
---

JavaScript, in many ways, is a practical language. It provides standard imperative language-level constructs that are common to many languages, but it also provides higher-level functional constructs. For example, JavaScript provides `for` loops, `Array.prototype.forEach`, and `Array.prototype.map` for dealing with collections. But at first glance, for iterating over the values in an array, these all seem identical. What gives?

Today, we'll compare and contrast the following three operations, explore their implementations, and discuss how and when to use each one:

 1. The standard `for` loop.
 1. `Array.prototype.forEach`.
 1. `Array.prototype.map`.

## The `for` loop
The `for` loop, based in imperative programming, is a well-known method for running a block of code multiple times based on a set of criteria. The `for` loop and variants of it can be found in many programming languages. Essentially, a block of code is defined within the `for` scope, and that block is executed a number of times based on the criteria checked in the for loop. There are a few variations (loops that use iterators, such as `for...in` and `for...of`), but It generally looks something like this:

```javascript
var array = [1, 2, 3, 4];
var newArray = [];

for (var i = 0; i < array.length; i++) {
	newArray.push(array[i] * 2);
}

// newArray: [2, 4, 6, 8]
```

In this case, the for loop defines a start condition (`i = 0`), a termination condition (`i < array.length`), and an execution statement (`i++`) that runs on every iteration of the loop. When we use a basic for loop with arrays, we generally use a counter; we define our start condition and end conditions based on the indices of the array that we want to access, and we set our execution statement to allow us to step through elements of the array by altering the counter.

### When is a `for` loop appropriate?

For loops are lower-level constructs in JavaScript, and as such have many use cases. You can use a for loop when:

 1. You need to break out of the `for` loop early (this can be accomplished using the `break` statement).
 1. You need a custom iteration order (for example, looping through an array backwards, or only performing an operation for every other element).
 1. Performance is critical and you heavily need to minimize abstractions.

### When is a `for` loop inappropriate?

There aren't many cases where a basic `for` loop won't work. However, there exist abstractions to make code cleaner and more maintainable, such as `forEach`.

## Array.prototype.forEach
When we use `Array.prototype.forEach`, it looks like this:

```javascript
var array = [1, 2, 3, 4];
var newArray = [];

array.forEach(function (each) {
	newArray.push(each * 2);
});

// newArray: [2, 4, 6, 8]
```
If we have `for` loops, why would we need a `forEach` function?

Imagine that we have three atomic operations: one to double a number, one to square it, and one to log the numbers to the console.

```javascript
function double (x) {
	return x * 2;
}

function square (x) {
	return x * x;
}

function print (x) {
	console.log(x);
}
```

Each of these operations will work for any single input that we provide. If I call `double(2)`, the result will be `4`, and if I call `print(4)`, "4" will be logged to the console. However, what if I want to apply the double operation to a collection of numbers? The operation functions defined above don't work with arrays. As such, I could write the following:

```javascript
function double (x) {
	return x * 2;
}

function doubleArray (array) {
	for (var i = 0; i < array.length; i++) {
		array[i] = double(array[i]);
	}
}
```

This works. But now, if I want to square the elements in my collection, I have to write another square operation just for an array (perhaps called `squareArray`) - I also have to write another operation to print the elements of an array. Let's enumerate the problems with this approach.

 1. The array-specific operation that we wrote (`doubleArray`) is now tightly coupled to the `double` operation. It can't be used for anything else.
 1. Every new operation we add will need to have an associated version just for arrays.
 1. Each array-specific operation that we have will repeat the same boilerplate code.

Why is this bad? It quickly becomes error-prone, because we have to repeat shared logic across multiple operations. We also end up with a lot of maintenance overhead; for every operation we create to handle a  single element, we need to create one to handle a collection, doubling the number of methods we need to write! (To me, this is unacceptable, because I'm lazy and I don't like grunt work.) What if we could consolidate the boilerplate logic into a single function and make this work for any and all of our original operations? Perhaps something like this:

```javascript
function forEach (array, operationName) {
	for (var i = 0; i < array.length; i++) {
		switch (operationName) {
			case "double":
				array[i] = double(array[i]);
				break;
			case "square":
				array[i] = square(array[i]);
				break;
			case "print":
				array[i] = print(array[i]);
				break;
			// ...and so on
		}
	}
}
```

This is better - it's more general - but we still have maintenance complexity, and we still have tight coupling. Is there a better way?

As it turns out, there is. JavaScript supports a paradigm known as first-class functions. This essentially means that functions can be treated like a number, a string, or any other data type; functions can be assigned to variables, and they can be passed around and referenced just like other objects.

> NOTE: This may seem standard for many of today's languages. However, not all programming languages, both old and new, allow functions to be used in this manner. Some classical languages such as C and early versions of Java fall into this category.

Currently, our function declaration looks like this:

```javascript
/**
 * @param {Array} array,
 * @param {string} operationName
 */
forEach(array, operationName)
```

If we can pass functions around as variables, our function declaration can look like this:

```javascript
/**
 * @param {Array} array
 * @param {Function} fn
 */
forEach(array, fn)
```

And the actual function would look something like this:

```javascript
function forEach (array, fn) {
	for (var i = 0; i < array.length; i++) {
		fn(array[i]);
	}
}
```

ForEach is an example of a higher-level function; it takes a function as a parameter and calls that function for each element in a collection, passing the element as an argument to the function.

> NOTE: The function above is just a demonstration of the underlying logic. In reality, the function implementation might look more like this, passing more arguments to the function and handling `this` scope correctly.

> ```javascript
// context is the "this" argument that we want to pass to our function
function forEach (array, fn, context) {
	for (var i = 0; i < array.length; i++) {
		if (array.hasOwnProperty(i)) {
			fn.prototype.apply(context, [array[i], i, array]);
		}
	}
}
```

So now, with our `forEach` function, we can do this:

```
var array = [1, 2, 3, 4];

function print (x) {
	console.log(x * 2);
}

array.forEach(print);

// console: 2
// console: 4
// console: 6
// console: 8
```

### When is `forEach` appropriate?
It's worth noting that anything you can do with `forEach`, you can do with a `for` loop. However, I personally only tend to use `for` loops as a fallback strategy when I can't attack an issue in a functional way. Use `forEach` when:

 1. You want to code `for` loops in a more functional style.
 2. You need to iterate over a sparse array and only want to run your functions for populated elements.
 3. You need to dynamically choose an operation to apply over your collection at runtime.

### When is `forEach` inappropriate?

There are cases when you will need to fall back to the standard for loop:

  1. You need a custom iteration order. `forEach` will always traverse an array in ascending index order, from the first element to the last.
  1. You need to terminate the `forEach` early. `forEach` doesn't allow for this. Use a `for` loop instead.
  1. You only want to iterate over a portion of the array. Use a `for` loop.
  1. You're dealing with a sparse array, but you want to run operations for all indexes, even if they are empty. Use a `for` loop.


## Array.prototype.map

When we use `Array.prototype.map`, it looks like this:

```javascript
var array = [1, 2, 3, 4];

var newArray = array.map(function (each) {
	return each * 2;
});

// newArray: [2, 4, 6, 8]
```

Initially, `Array.prototype.map` seems similar to `Array.prototype.forEach`. Both are higher-level functions, taking a function as an argument and calling the function for each element in a collection. They even pass the same arguments!

There are, however, some core differences.

 1. `forEach` takes a function as an argument, but doesn't use its return value. `map` uses the return values of the passed function.
 1. `forEach` has no return value, while `map` returns a new array that has been transformed by the passed function.
 1. `map` does not mutate the original array.

The core idea driving `forEach` is this: *Given an operation, repeat that operation for each element in a collection.*

The fundamental idea behind the `map` function is more focused, and reads like this: *Given a function that can transform a single element, apply that transform function over a collection.*

Let's take our operations from above:

```javascript
function double (x) {
	return x * 2;
}

function square (x) {
	return x * x;
}

function print (x) {
	console.log(x);
}
```



Because `map` returns a new array with the transformations, it can be chained, like so:

```javascript
var result = [1, 2, 3, 4, 5]
	.map(double)
	.map(square);

// result: [4, 16, 36, 64, 100]
```

However, what happens if we use the print operation?

```
var result = [1, 2, 3, 4, 5]
	.map(double)
	.map(square)
	.map(print);

// result: [undefined, undefined, undefined]
```

What happened? `map` uses the return value of the operation to populate each element of the new array. Because the `print` function has no return value, each element ends up undefined.

In contrast to `forEach`, any operation passed to `map` must provide a return value. Ideally, these functions would be "pure functions", functions that don't contain any side effects.

### When is `map` appropriate?

1. You are transforming data, but you don't want to affect the original array.
1. You are transforming data as part of a functional pipeline.

### When is `map` inappropriate?

1. You want to iterate over an array to perform an operation with side effects, and you don't care about the return values. Use `forEach`.
1. You want to operate on the array in-place, mutating each element. Use `forEach` or a `for` loop.
3. You are concerned about space complexity. Because `map` creates a new array, it doubles the storage space needed. Consider using `forEach` or a `for` loop to operate on the array in-place.