JavaScript Inheritance Patterns
---

## Objectives

1. Explain how prototypal inheritance works in JavaScript
2. Practice inheriting from prototypes with `Object.create()`
3. Explain how to use `hasOwnProperty()`

## Introduction

There are two primary types of object-oriented (OO) languages: class-based
and prototype-based.

In a class-based OO language, such as Ruby, Java, C#, and C++, we must
first design the *Class*, or blueprint, of an object, and then create
objects from that blueprint in order to use them.

Imagine a set of instructions from IKEA as the *Class*, and your
assembled Fthugënbøller bookcase as the *object*.

JavaScript uses prototypal OO. Instead of creating a nonfunctional class
definition, we actually create the object, which is then used as a
prototype to create other objects.

In the real world, this would be like taking a bunch of laminated wood
and assembling a Fthügënbøller, and then using that as your guide, or
prototype, for creating other Fthügënbøllers, and Fthügënbøller-like objects.

**//Flat-fact:** I'm regretting my choice to use impossible-to-type "Fthügënbøller" as my
made up example, but I'm committed to the bit. Besides, we all know
everything about IKEA furniture is a nightmare.

![crib](http://i.giphy.com/ZvkHeRqClNgUE.gif)

So in both OO paradigms, we're defining the object, but in JavaScript,
that definition is a functioning prototype that we can use to build
other objects.

## Inheritance

Where this distinction between class-based and prototype-based OO really
comes to the forefront is in *inheritance*.

Inheritance is a part of object-oriented programming that concerns
itself with code reuse. Inheritance allows us to build more complex or
specific object types out of more simple, previously defined object
types.

### "is-a"

As a real-world example, consider a rectangle. A rectangle has some
rules to it, or *properties*. A rectangle has four sides. Opposite sides
are equal length and parallel. Adjacent sides are perpendicular.

Now consider the square. A square shares all of the properties of a
rectangle, and by definition *is* a rectangle. However, the square has
an additional important property that a rectangle does not: a square is
equilateral. All four sides are the same length.

So a square *is a* rectangle, but a rectangle *is not a* square. In
object-oriented programming, we would consider this *is-a* relationship
a place to use inheritance. That is, we could define a rectangle, then
create a square out of that rectangle and add the additional properties.
The inheritance allows us to reuse the rectangle's properties without
having to redefine them on the square.

Let's see this in action, first without inheritance:

```js
// Rectangle constructor
function Rectangle(sides, width, height) {
  this.sides = sides;
  this.width = width;
  this.height = height;
  this.area = function() {
    return this.width * this.height;
  }
  this.perimeter = function() {
    return (this.width + this.height) * 2;
  }
}

// Square constructor
function Square(sides, length) {
  this.sides = sides;
  this.width = length;
  this.height = length;
  this.area = function() {
    return this.width * this.height;
  },
  this.perimeter = function() {
    return (this.width + this.height) * 2;
  }
}

var rect = new Rectangle(4, 3, 5);
var square = new Square(4, 2);

rect.area();
square.area();
```

This works great, but we've created two objects that have basically the same properties and behaviors.
The only difference is that `width` and `height` are the same in a
`square`, so we only take one `length` argument and use it to set
`width` and `height` to the same thing.

Inheritance allows us to create a square from a rectangle and then alter
the square as necessary to make a new type of object, without having to
repeat everything we did when constructing the rectangle. So how can we
implement this using inheritance in JavaScript?

### Constructor-Based Inheritance

It isn't much of a stretch to see a simple way to make `Square`
"inherit" from `Rectangle` by just using the constructor and a `call` to
set `this` to a new square:

```js
// Rectangle constructor
function Rectangle(sides, width, height) {
  this.sides = sides;
  this.width = width;
  this.height = height;
  this.area = function() {
    return this.width * this.height;
  }
  this.perimeter = function() {
    return (this.width + this.height) * 2;
  }
}

// Square constructor
function Square(sides, length) {
  Rectangle.call(this, sides, length, length);
}

var rect = new Rectangle(4, 3, 5);
var square = new Square(4, 2);

rect.area();
square.area();
```

This will work, and it certainly reduces the repeated code. But under
the hood, there's a problem. Our `square` isn't truly inheriting from
the prototypical `Rectangle`. It is, in essence, a standalone object
that only copied what `Rectangle` had available at the time of
instantiation.

To illustrate, try adding this code:

```js
Rectangle.prototype.internalAngles = 90;
rect.internalAngles;
square.internalAngles;
```

The `rect` can access `internalAngles`, but `square` cannot. Why? The
answer is the `prototype` attribute.

### Prototypes

Every object in JavaScript has a `prototype`. This `prototype`
is essentially the "parent" object that the object was created from, and
it provides any number of methods and properties to that object.

Any object created with the object literal `{}` will inherit from the
`Object.prototype`, meaning it will get certain methods and properties
for free. Try it yourself:

```js
var o = {};
console.log(o.toString());
```

We didn't define a `toString()` function on `o` -- it came from
`Object.prototype`.

So the prototype is the parent object that this one was created from,
and when we try to access a property or method on our object that isn't
defined by our object, JavaScript will check our object's prototype, and
if it's not there, then *that* object's prototype, and so on, until it
finds what it's looking for or reaches the end of the prototype chain at
`Object.prototype`. This is a form of delegation - rather than force
every object to handle everything it inherits on its own, it can
delegate up the prototype chain.

This means that every object that inherits from another gains two
benefits from prototypal inheritance.

The first is efficiency. Not having to copy every method to
every inherited object keeps things light and simple.

The second is that this delegation allows already-instantiated objects
to reap the benefits of changes to their prototype at runtime.

Let's revisit the example above:

```js
// Rectangle constructor
function Rectangle(sides, width, height) {
  this.sides = sides;
  this.width = width;
  this.height = height;
  this.area = function() {
    return this.width * this.height;
  }
  this.perimeter = function() {
    return (this.width + this.height) * 2;
  }
}

// Square constructor
function Square(sides, length) {
  Rectangle.call(this, sides, length, length);
}

var rect = new Rectangle(4, 3, 5);
var square = new Square(4, 2);

rect.area();
square.area();

Rectangle.prototype.internalAngles = 90;
rect.internalAngles;
square.internalAngles;
```

If we just examined these two objects with `console.log(square)` and
`cosole.log(rect)`, we would see that neither explicitly has the
`internalAngles` property. This means that `rect` is *delegating* it to
the `Rectangle` prototype, and `square` is not.

They were both created from the `Rectangle` constructor though. What
gives?

### Problems With Constructor Functions and new

Using constructor functions with the `new` keyword is really just a way
to force a class-based OOP paradigm onto the prototypal nature of
JavsScript, obscuring, and sometimes interfering with, the benefits of
using JavaScript in a purely prototypal way.

One of the problems with the constructor function pattern is that it
requires the use of `new` to work properly. If you forget the `new`,
things will go very wrong when you try to use your objects. We use the
convention of capitalizing the first letter of the function, but that's
not foolproof.

However, the even bigger problem with constructor functions and `new`
shows up when we try to inherit from another object and take advantage
of the delegation features in the prototype system.

When an object is created, its prototype is set to the
object it was created from. In the case of a new object created with the
`{}` literal, that prototype will be `Object.prototype`.

But what do we know about functions in JavaScript? They are also
objects! So when we create an object from a constructor function, we are
really making that function the object's `prototype`!

![kramer mind blown](http://i.giphy.com/OK27wINdQS5YQ.gif)

Let's look at our example one more time.

```js
// Rectangle constructor
function Rectangle(sides, width, height) {
  this.sides = sides;
  this.width = width;
  this.height = height;
  this.area = function() {
    return this.width * this.height;
  }
  this.perimeter = function() {
    return (this.width + this.height) * 2;
  }
}

// Square constructor
function Square(sides, length) {
  Rectangle.call(this, sides, length, length);
}

// rect prototype becomes Rectangle()
var rect = new Rectangle(4, 3, 5);
// square prototype becomes Square()
var square = new Square(4, 2);

// this works because the rect prototype is still Rectangle
Rectangle.prototype.internalAngles = 90;
rect.internalAngles;

// but the square prototype is not Rectangle
square.internalAngles;
```

Even though we call the `Rectangle` constructor function inside the
`Square` constructor function, the function that actually *creates*
`square` is `new Square()`, so the prototype for the square is
`Square.prototype`, meaning that we haven't inherited from
`Rectangle` at all, we've just borrowed its constructor function to make our own
object, and made copies of its properties and methods, but haven't taken
advantage of prototyping.

### Object.create()

Fortunately, we have `Object.create()` to rescue us from these problems
with `new`. Rather than try to force class-style inheritance with `new`,
we'll use the pure object-to-object prototype system that JavaScript is
built on.

In order to create a `Square` that truly inherits from `Rectangle`, we'll need
to make a few changes to how we put our objects together.

We need to convert our constructor function to a simple object
definition, creating the "prototype" for all rectangles to come:

```js
var rectangle = {
  sides: 4,
  width: 5,
  height: 7,
  area: function() {
  	return this.width * this.height;
  },
  perimeter: function () {
  	return (this.width + this.height) * 2;
  }
}
```

We have created a perfectly functional `rectangle` object, and not only
can we use it, but we can use it to create other rectangle-like objects.
Let's try a few things:

```js
var rect = Object.create(rectangle);

console.log(rect);
// empty Object {}

console.log(rect.sides);
// 4

console.log(rect.__proto__);
/* rectangle { sides: 4, width: 5, height: 7,
              area: [Function: area],
              perimeter: [Function: perimeter] }
*/

rectangle.isPrototypeOf(rect);
// true
```

Let's examine each of these statements and what's happening.

First we create a new object called `rect` with
`Object.create(rectangle)`. JavaScript is giving us an instance of a new
object whose prototype is `rectangle`. This new `rect` object is not,
itself, a `rectangle`. It is created from the `rectangle` prototype, or
we can say it inherits from `rectangle`.

To illustrate the difference, if we `console.log(rect)`, we see that
it's an empty object. Just the `{}` object literal.

However, if we try to access `rect.sides`, we get `4`, which is the
value for `rectangle.sides`. Why? Because, as learned earlier,
JavaScript makes use of the prototype to *delegate* property and
function calls up the prototype chain. Here, when we call `rect.sides`,
JavaScript first looks for a property called `sides` in `rect`. But
`rect` is an empty object, so it looks at the prototype for `rect`, in
this case `rectangle`. It finds a `sides` property on `rectangle` and
returns that value.

We can see this prototype chain by accessing `rect.__proto__` and
examining the results, which is our `rectangle` and all its values.
Every object has a `__proto__` property, and that's how JavaScript knows
where to go next to look for things. If it didn't find `size` in
`rect.__proto__` it would have looked in `rect.__proto__.__proto__` and
so on until it reached the end of the inheritance chain at
`Object.prototype`.

Finally, we can prove that `rectangle` is the prototype for `rect` by
calling `rectangle.isPrototypeOf(rect)`.

If we set a property on `rect`, what happens?

```js
rect.width = 2;
console.log(rect);
// {width: 2}
console.log(rectangle);
/*  { sides: 4, width: 5, height: 7,
      area: [Function: area],
      perimeter: [Function: perimeter] }
*/
```

We already know that you can add properties to an object at runtime by
just setting them, so here we're adding the property `width` to our
`rect` oject, which means when we try to access it, JavaScript will find
it on `rect` and will no longer have to access `__proto__`, effectively
overriding the property for our `rect` object.

A rectangle is really just a special quadrilateral, right? Let's build out this example a little more and see some inheritance in action.

```js
var quadrilateral = {
  sides: 4,
  sideOneLength: 1,
  sideTwoLength: 2,
  sideThreeLength: 1,
  sideFourLength: 2,
  perimeter: function() {
    return this.sideOneLength + this.sideTwoLength + this.sideThreeLength + this.sideFourLength;
  },
  setSides: function(s1, s2, s3, s4) {
    this.sideOneLength = s1;
    this.sideTwoLength = s2;
    this.sideThreeLength = s3;
    this.sideFourLength = s4;
  }
}

var rectangle = Object.create(quadrilateral);
rectangle.width = function() {
  return this.sideOneLength;
}
rectangle.height = function() {
  return this.sideTwoLength;
}
rectangle.setWidth = function(width) {
  this.sideOneLength = width;
  this.sideThreeLength = width;
}
rectangle.setHeight = function(height) {
  this.sideTwoLength = height;
  this.sideFourLength = height;
}
rectangle.area = function() {
  return this.width() * this.height();
}

var square = Object.create(rectangle);
square.setSideLength = function(l) {
  this.setWidth(l);
  this.setHeight(l);
}

square.setSideLength(4);
square.area();
square.perimeter();
```

Now we have a base `quadrilateral` that we can use to build a
`rectangle`. We'll extend the rectangle to have functions to determine
more rectangular-specific things, like `width`, `height`, and `area`. It
wouldn't make sense to define `area()` at the `quadrilateral` level,
because it's a different formula depending on the shape. So we do that
on `rectangle`. But we preserve `perimeter()` by using the side length
properties of `quadrilateral`, so that even when we get down to
`square`, we can still use the prototype system to delegate our
perimeter formula up to `quadrilateral`.

Since a square is just an equilateral rectangle, our `square` can use
everything from the `rectangle` prototype, and just extend it to
specifically set one side-length value instead of a `width` and
`height`.

This is the basic pattern of an inheritance chain. A parent object is
more general than a child, which takes attributes from the parent and
adds more specific attributes of its own.

## hasOwnProperty()

One interesting challenge with objects and inheritance is that we
sometimes need to determine exactly what properties and methods are
available on an object, and whether or not they are defined on the
object itself or delegated to a prototype. Let's look at our `square`
from the last example, which was created from the `rectangle` prototype,
which was created from the `quadrilateral` prototype.

```js
var quadrilateral = {
  sides: 4,
  sideOneLength: 1,
  sideTwoLength: 2,
  sideThreeLength: 1,
  sideFourLength: 2,
  perimeter: function() {
    return this.sideOneLength + this.sideTwoLength + this.sideThreeLength + this.sideFourLength;
  },
  setSides: function(s1, s2, s3, s4) {
    this.sideOneLength = s1;
    this.sideTwoLength = s2;
    this.sideThreeLength = s3;
    this.sideFourLength = s4;
  }
}

var rectangle = Object.create(quadrilateral);
rectangle.width = function() {
  return this.sideOneLength;
}
rectangle.height = function() {
  return this.sideTwoLength;
}
rectangle.setWidth = function(width) {
  this.sideOneLength = width;
  this.sideThreeLength = width;
}
rectangle.setHeight = function(height) {
  this.sideTwoLength = height;
  this.sideFourLength = height;
}
rectangle.area = function() {
  return this.width() * this.height();
}

var square = Object.create(rectangle);
square.setSideLength = function(l) {
  this.length = l;
  this.setWidth(l);
  this.setHeight(l);
}
```

We can look at each property available to `square` by iterating over
them with `for...in`:

```js
for (var prop in square) {
  console.log("square." + prop + " = " + square[prop]);
}
```

Whoa! That outputs everything from `quadrilateral` through `rectangle`
and then to `square`! That's a lot. What if we just want to see if
something came from `square` and isn't delegated to a prototype?

We can use `hasOwnProperty()` for just this. We can check that
`setSideLength` comes from `square` like this:

```js
square.hasOwnProperty("setSideLength");
//true
```

This tells us that `setSideLength` was defined on `square` and isn't
going to be delegated to the prototype. What about `width()`?

```js
square.hasOwnProperty("width");
//false

rectangle.hasOwnProperty("width");
//true
```

Even though we can call `square.width()`, it's not defined by `square`,
it's defined by `rectangle`.

Why is this important? Consider the `square` implementation again:

```js
var square = Object.create(rectangle);
square.setSideLength = function(l) {
  this.length = l;
  this.setWidth(l);
  this.setHeight(l);
}
```

Here, if we call `square.setSideLength`, we'll not only call `setWidth`
and `setHeight`, but we'll also set a property called `length` on our
square. Before we call this function, let's check to see if `square` has
a `length` property:

```js
square.hasOwnProperty("length");
///false
```

Now we know that `length` has never been set on our square, meaning our
side-lengths are coming from who-knows-where up the prototype chain! But
once we call `setSideLength`, it will set that property, and we can
verify that our particular square has a length:

```js
square.hasOwnProperty("length");
///false
square.setSideLength(12);
square.hasOwnProperty("length");
//true
```

Now we know for sure that this instance of `square` has its own
`length`, and isn't delegating it to someone else, potentially causing
bugs.

**Top-tip:** You might think it's enough to check if `square.length ===
undefined` but all that tells us is whether or not `square.length`
evaluates to `undefined`, which could be true regardless of if `length`
exists as a property of `square`.

## Summary

We've talked about the difference between class-based and prototypal
inheritance models in object-oriented languages, and learned that every
object in JavaScript can serve as a prototype with which we can build
other objects.

We've seen how to emulate class-like inheritance with constructor
functions and the `new` keyword, and explored potential problems with
that approach.

We've seen how to create prototype objects and then inherit from them
with `Object.create()`, avoiding the pitfalls of `new`, and learned how
to use `hasOwnProperty()` to determine if a given property belongs to an
object or is delegated to its prototype.

## Resources

- [Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
- [for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in)
- [hasOwnProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)
