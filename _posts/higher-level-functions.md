Array.forEach vs Array.map?

-- Why use array.forEach vs a for loop?
	-
-- Why use a for loop over array.forEach?
	- you can break out of a for loop
	- you can continue - break and continue are basically 'GOTO' statements under the hood
	- for loops, being low-level language constructs, have less overhead.
- When to use array.map?

- Side effect-driven functions
- map functions - being able to be used on a single value, and then a collection of values
- pure functions with no mutations - deterministic.

Javascript is a something language. It provides standard language-level constructs, such as the `for` loop, but it also provides higher-level functional constructs, such as `Array.prototype.forEach` and `Array.prototype.map`. At first glance, for iterating over the values in an array, these all seem identical. What gives?

We'll talk about three things today: the standard `for` loop, the `forEach` function, and then the `map` function.

# The `for` loop
The `for` loop, based in imperative programming, is one of the de facto methods for running a block of code multiple times based on a set of criteria. Essentially, a block of code is defined within the `for` scope, and that block is executed a number of times based on the criteria checked in the for loop.

# Array.prototype.forEach
ForEach is an example of a higher-level function; it takes a function as a parameter and calls that function for each element in a collection, passing the element and the array index to the function.

# Array.prototype.map
The fundamental idea behind the `map` function is this: *Given a function that can transform a single element, apply that function over a collection.*

Imagine a function that doubles a number:

function double (x) {
	return x * 2;
}

Then, imagine a function that squares a number:

function square (x) {
	return x * x;
}

This function will work for any single input that we provide. If I call `double(2)`, the result will be `4`. However, what if I want to apply this function to a collection of numbers? I could write the following:

function doubleArray (array) {
	for (var i = 0; i < array.length; i++) {
		array[i] = array[i] * 2;
	}

	return array;
}

This works. But now, I want to square the elements in my collection. This means that I have to write another function to square an array (squareArray). Later, we're asked to cube both a single element and all elements in a collection. Perhaps down the line, we're asked to transform an element to a hexadecimal string value, but we also need to do this for a collection.

We could do something like this:

function doubleArray (array) {
	for (var i = 0; i < array.length; i++) {
		array[i] = double(i);
	}
}

This is a little better. However, there's are a couple of problems here:
1. The Array function that we wrote (doubleArray) is now tightly coupled to the `double` function. It can't be used for anything else.
1. For each transformation we want to do, we still need to write an extra Array function for it.
1. Each Array function that we have that deals with a collection will have the same boilerplate code.

Why is this bad? It quickly becomes error-prone, because we have to repeat shared logic across multiple functions. We also end up with a lot of maintenance overhead; for every function we create that operates on a single element, we need to create one that operates on a collection. What if we could consolidate the boilerplate logic and make this work for any function?

function applyToArray (array, operationName) {
	for (var i = 0; i < array.length; i++) {
		switch (operationName) {
			case "double":
				array[i] = double(i);
				break;
			case "square":
				array[i] = square(i);
				break;
		}
	}
}

This is better - it's more general - but we still have maintenance complexity, and we still have tight coupling. Is there a better way?

JavaScript supports a paradigm known as first-class functions. This essentially means that functions can be treated like a number, a string, or any other data type; they can be stored in variables, and they can be passed around. (This may not seem like a huge deal, but not all programming languages work this way.)

Currently, our function declaration looks like this:

applyToArray(array, operationName)

If we can pass functions around as variables, our function declaration can look like this:

applyToArray(array, fn);

And the actual function would look something like this:

function applyToArray (array, fn) {
	for (var i = 0; i < array.length; i++) {
		fn(array[i]);
	}
}

> NOTE: In reality, because of the way that Javascript handles scope, it's more like this:

```
// context is the "this" argument that we want to pass to our function
function forEach (array, fn, context) {
	for (var i = 0; i < array.length; i++) {
		fn.prototype.call(context, array[i]);
	}
}

>

Initially, Array.prototype.map seems similar to Array.prototype.forEach. Both are higher-level functions, taking a function as an argument and calling the function for each element in a collection. They even pass the same arguments!

There are, however, some fundamental differences:

1. `forEach` takes a function as an argument, but doesn't use its return value. `map` uses the return values of the passed function.
1. `forEach` has no return value, while `map` returns a new array that has been transformed by the passed function.

Because map has a return value, it can be chained, like so:

var result = [1, 2, 3, 4, 5]
	.map((num) => num * 2)
	.filter((num) => num > 5);