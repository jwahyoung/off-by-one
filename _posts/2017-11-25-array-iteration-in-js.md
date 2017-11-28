---
uid: 4d36d722-2bf5-415e-ae54-17a37798b691
layout: post
title: Exploring Array iteration in JavaScript
description: "JavaScript provides 'for' loops, 'Array.prototype.forEach', and 'Array.prototype.map' for dealing with collections. But at first glance, for iterating over the values in an array, these all seem identical. What gives?"
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
*"Given a block of code, repeat that code block until the termination condition is satisfied."*

The `for` loop, based in imperative programming, is a well-known method for running a block of code multiple times based on a set of criteria. The `for` loop and variants of it can be found in many programming languages. Essentially, a block of code is defined within the `for` scope, and that block is executed a number of times based on the criteria checked in the loop. It generally looks something like this:

```javascript
var array = [1, 2, 3, 4];
var newArray = [];

for (var i = 0; i < array.length; i++) {
	newArray.push(array[i] * 2);
}

// newArray: [2, 4, 6, 8]
```

In this case, the for loop defines a start condition (`i = 0`), a termination condition (`i < array.length`), and an execution statement (`i++`) that runs on every iteration of the loop. When we use a basic `for` loop with arrays, we generally use a counter; we define our start condition and end conditions based on the indices of the array that we want to access, and we set our execution statement to allow us to step through elements of the array by altering the counter.

> *What about [for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) and [for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)?*
>
> There are a few variations of the basic `for` loop that use iterators, such as the `for...in` loop and the `for...of` loop. While `for...in` and `for...of` can be used with arrays, they are much more powerful when used with JavaScript objects as they iterate over properties, not just array indices. For the scope of this blog post, we'll focus on iterating over arrays and their indices only.

`for` loops allow control flow via two keywords: `break`, and `continue`. `break` will exit the loop early, while `continue` will immediately begin the next loop cycle. These keywords are generally used with conditional statements. `continue` can be used like so:

```javascript
var array = [1, 2, 3, 4];
var newArray = [];

for (var i = 0; i < array.length; i++) {
    if (array[i] % 2 === 0) {
        newArray.push(array[i] / 2);
        continue;
    }

    newArray.push(array[i] * 2);
}

// newArray: [2, 1, 6, 2]
```

...while `break` can be used in the following fashion:

```javascript
var array = [1, 2, 3, 4];
var search = 3;

for (var i = 0; i < array.length; i++) {
    if (array[i] === search) {
        console.log(array[i]);
        break;
    }
}
```

For loops are simple and powerful. However, there exist abstractions to make collection iteration easier, cleaner, and more maintainable - such as `forEach`

## Array.prototype.forEach
*"Given an operation, repeat that operation for each element in a collection."*

Here's an example of `forEach` in action:

```javascript
var array = [1, 2, 3, 4];
var newArray = [];

array.forEach(function (each) {
	newArray.push(each * 2);
});

// newArray: [2, 4, 6, 8]
```

If we have `for` loops, why would we need a `forEach` function?

### The thought process behind `forEach`

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

Why is this bad? It quickly becomes error-prone, because we have to repeat shared logic across multiple operations. We also end up with a lot of maintenance overhead; for every operation we create to handle a  single element, we need to create one to handle a collection, doubling the number of methods we need to write! Can we fix this?

### Designing a basic implementation of `forEach`

What if we could consolidate the boilerplate logic into a single function and make this work for any and all of our original operations? Perhaps something like this:

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

This is better - it's more general. (Instead of switch statements, we could use function maps in a lookup object). But we still have maintenance complexity even if we refactor this, and we still have tight coupling. Is there a better way?

As it turns out, there is. JavaScript supports a paradigm known as [first-class functions](https://en.wikipedia.org/wiki/First-class_function). This essentially means that functions can be treated like a number, a string, or any other data type; functions can be assigned to variables, and they can be passed around and referenced just like other objects.

> NOTE: This may seem standard for many of today's languages. However, not all programming languages, both old and new, allow functions to be used in this manner. Some classical languages such as C and early versions of Java fall into this category.

Currently, our function declaration looks like this:

```javascript
/**
 * @param {Array} array
 * @param {string} operationName
 */
forEach(array, operationName)
```

If we can pass functions around as variables, our function declaration can look like this:

```javascript
/**
 * @param {Array} array
 * @param {Function} operation
 */
forEach(array, operation)
```

And the actual function would look something like this:

```javascript
function forEach (array, operation) {
	for (var i = 0; i < array.length; i++) {
		operation(array[i]);
	}
}
```

`forEach` is an example of a [higher-order function](https://en.wikipedia.org/wiki/Higher-order_function); it takes a function as a parameter and calls that function for each element in a collection, passing the element as an argument to the function.

So now, with our `forEach` function, we can do this:

```javascript
var array = [1, 2, 3, 4];

function print (x) {
	console.log(x * 2);
}

forEach(array, print);

// console: 2
// console: 4
// console: 6
// console: 8
```

### The actual implementation of `Array.prototype.forEach`

The `forEach` function we defined above is just a demonstration of the underlying logic. In practice, we'd run into some issues. The actual implementation is built differently to handle more use cases - such as providing a "this" context to the operation, and skipping operations for empty buckets in sparse arrays - but the underlying premise is still the same. The following code is closer to a production implementation:

```javascript
/**
 * @param {Function} operation The operation to run on each element of the array.
 * @param {Object} [context] The value of "this" in the operation.
 */
Array.prototype.forEach = function (operation, context) {
	for (var i = 0; i < this.length; i++) {
		if (this.hasOwnProperty(i)) {
			operation.prototype.apply(context, [this[i], i, this]);
		}
	}
}
```

> *What's a sparse array?*
>
> An array's length is always one more than its largest index. However, that doesn't always mean that an array is storing that number of values. Sparse arrays are arrays that have more than zero empty elements. (Null or undefined is not empty.) For example, the array literal `[1, ,3]` has length 3, but only stores two values - one element is empty. The array literal `[ , , , ]` has length 4, but it's completely empty! The `forEach` function above would skip these elements.

## Array.prototype.map
*"Given a function that can transform a single element, apply that transform function over a collection."*

Here's an example of `map` in action:

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

### The thought process behind `map`

Let's take our double operation from above, along with our `forEach` function from above:

```javascript
function double (x) {
	return x * 2;
}

var array = [1, 2, 3, 4];
var newArray = [];
forEach(array, double);

// array: [1, 2, 3, 4]
// newArray: []
```

Uh-oh. What's happening here?

`forEach` runs our `double` operation on each element of the array. Our operation returns the correct value, but due to the way `forEach` works the return value isn't used. This means that our operation would have to have a side effect to work:

```javascript
function double (x) {
	newArray.push(x * 2);
}

var array = [1, 2, 3, 4];
var newArray = [];
forEach(array, double);

// array: [1, 2, 3, 4]
// newArray: [2, 4, 6, 8]
```

This, however, is still problematic. We've introduced side effects and our double operation is now tightly coupled to `newArray`; we can work around this in various ways (we could retain the original `double` operation, and define a `doubleArray` operation), but we've essentially recreated the problems we had before.

It'd be better if we could do something like the following:

```javascript
function double (x) {
	return x * 2;
}

var array = [1, 2, 3, 4];
var newArray = forEach(array, double);
```

...but the output isn't what we expect:

```javascript
// array: [1, 2, 3, 4]
// newArray: undefined
```

`forEach` isn't designed to return a value. Is there another solution?

### Designing a basic implementation of `map`

Let's define a new function, `map`, that follows the same principles as `forEach`, but that uses the return value of the passed operation and returns the result of transforming the collection.

```javascript
function map (array, operation) {
	var mapped = [];

	for (var i = 0; i < array.length; i++) {
		mapped.push(operation(array[i]));
	}

	return mapped;
}
```

Now, we can use our function to set our variable:

```javascript
function double (x) {
	return x * 2;
}

var array = [1, 2, 3, 4];
var newArray = map(array, double);

// array: [1, 2, 3, 4]
// newArray: [2, 4, 6, 8]
```

What happened? `map` uses the return value of the operation to populate each element of the new array. In contrast to `forEach`, any operation passed to `map` must provide a return value. Ideally, these functions would be [pure functions](https://en.wikipedia.org/wiki/Pure_function), functions that don't contain any side effects.

### The actual implementation of `Array.prototype.map`

Again, the `map` function above is just a demonstration of the underlying logic. The actual implementation is an Array.prototype function that handles sparse arrays, context, and chaining, but the underlying premise is still the same. The following code is closer to a production implementation:

```javascript
/**
 * @param {Function} operation The operation to run on each element of the array.
 * @param {Object} [context] The value of "this" in the operation.
 */
Array.prototype.map = function (operation, context) {
	var map = this.slice();
	for (var i = 0; i < this.length; i++) {
		if (this.hasOwnProperty(i)) {
			map[i] = operation.prototype.apply(context, [this[i], i, this]);
		}
	}
	return map;
}
```

Because `map` returns a new array with the transformations, and because it's defined on `Array.prototype`, it can be chained, like so:

```javascript
var result = [1, 2, 3, 4, 5]
	.map(double)
	.map(square);
// result: [4, 16, 36, 64, 100]
```

Because this can be chained, it can be used in a pipeline with other array functions, such as `filter`, `reduce`, and `join`.

## Use Cases

Now that we've talked a bit about the thought processes behind these constructs and explored possible implementations, it's time to talk about how and when each should be used.

### When is a `for` loop appropriate?

For loops are lower-level constructs in JavaScript, and as such have many use cases. You can use a for loop when:

 1. You need to break out of the `for` loop early (this can be accomplished using the `break` statement).
 1. You need a custom iteration order (for example, looping through an array backwards, or only performing an operation for every other element).
 1. Performance is critical and you heavily need to minimize abstractions.

### When is a `for` loop inappropriate?

There aren't many cases where a basic `for` loop won't work. That said, if it's cleaner to use a high-level abstraction and you don't have any of the constraints defined above, consider using the functional abstractions.

### When is `forEach` appropriate?
It's worth noting that anything you can do with `forEach`, you can do with a `for` loop. However, I personally only tend to use `for` loops as a fallback strategy when I can't attack an issue in a functional way. Use `forEach` when:

 1. You want to code `for` loops in a more functional style.
 2. You need to iterate over a sparse array and only want to run your functions for populated elements.
 3. You need to dynamically choose an operation to apply over your collection at runtime.

These use cases can definitely be handled with a `for` loop and some manual work, but sometimes `forEach` is simply more elegant.

### When is `forEach` inappropriate?

There are cases when you will need to fall back to the standard for loop:

  1. You need a custom iteration order. `forEach` will always traverse an array in ascending index order, from the first element to the last.
  1. You need to terminate the `forEach` early. `forEach` doesn't allow for this. Use a `for` loop instead.
  1. You only want to iterate over a subset of the array. Use a `for` loop.
  1. You're dealing with a sparse array, but you want to run operations for all indices, even if they are empty. Use a `for` loop.

### When is `map` appropriate?

1. You are transforming data as a set, but you don't want to mutate the original array.
1. You are transforming data as part of a functional pipeline.
1. Your transformation operation is atomic and has no state based on the array or any side effects.

### When is `map` inappropriate?

1. You want to iterate over an array to perform an operation with side effects, and you don't care about the return values. Use `forEach`.
1. You want to operate on the array in-place, mutating each element. Use `forEach` or a `for` loop.
1. You want to only map a subset of an array to a transformation. Use a `for` loop, or remove the values you don't want in your collection before you call `map` on it.
3. You are concerned about space complexity. Because `map` creates a new array, it doubles the storage space needed. Consider using `forEach` or a `for` loop to operate on the array in-place.

## Wrap-up

Today, we've covered:

1. The standard `for` loop.
1. `Array.prototype.forEach`.
1. `Array.prototype.map`.

We took a look at a couple of the functional abstractions for looping in JavaScript, we've explored their implementations, and we've talked about when they're useful and when they might be the wrong tool to use.

JavaScript, despite its faults, can be a very elegant language with some nice tools. At times, some tools seem similar, but they have their differences, use cases, and nuances. Hopefully, this has helped to clear up the differences between (and, more importantly, the reasons behind) `for` loops, `Array.prototype.forEach`, and `Array.prototype.map`. Check out some more resources below!

## More Resources

- [for loops on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for)
- [Array.prototype.map on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map), and the recommended polyfill implementation: [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map#Polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map#Polyfill)

- [Array.prototype.forEach on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach), and the recommended polyfill implementation: [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach#Polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach#Polyfill)

- [Lodash](https://lodash.com/)
    - Lodash's map implementation: [https://github.com/lodash/lodash/blob/master/map.js](https://github.com/lodash/lodash/blob/master/map.js)
    - Lodash's foreach implementation:
    [https://github.com/lodash/lodash/blob/master/forEach.js](https://github.com/lodash/lodash/blob/master/forEach.js)