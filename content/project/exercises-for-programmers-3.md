---
title: "Exercises for Programmers 3 - NODEJS"
date: 2018-01-04T21:41:19-07:00
draft: false
tags: ["programming exercises", "nodejs"]
summary: "Working throught part of chapter 3 in NodeJS."
---
I'm in the midst of working through Chapter 3 of
[Exercises for Programmers](https://pragprog.com/book/bhwb/exercises-for-programmers)
and decided to write an update on my progress because some of these
individual exercises are starting to get a bit long and convoluted,
so I feel like some of them might benefit from being stand-alone posts.
That will certainly be true for Exercise 10, which I finished up last
night, so for this post I'm just going to talk about Exercises 7-9.

Also, [here's my Github repo for this project.](https://github.com/jeremy-j-ackso/exercises-for-programmers-nodejs)

{{% toc %}}

# Background
This chapter is all about calculations.
All of the exercises have you taking some input, doing some calculations,
and then writing the output.
As you might expect you end up getting cozy with your basic 4 functions,
`Math` and `Number` methods like rounding and defining decimal precision, and validating
that user inputs are parseable digits.
The complexity that is getting introduced in this chapter also really
forces the issue of making sure that you break out your functionality
into smaller component functions, and you can find some rewarding opportunities
for using arguments to slightly modify your function outputs, for instance
when you're converting between imperial and metric units.

As you might expect, with more component functions comes more tests.
Like I mentioned, last night I got through Exercise 10, and already my test file
for this chapter is around 750 lines long.
I have three more exercises to go for this chapter, so I'm figuring it will grow
to be over 1000 lines before this chapter ends.
That being the case, I'm not going to post my tests in here because they're
getting to be rather boilerplate-y and I personally wouldn't be interested in scrolling
for ages through similar-looking mocha tests.
However you can definitely go check the tests out
[here](https://github.com/jeremy-j-ackso/exercises-for-programmers-nodejs/blob/master/test/ch3.test.js).

My scripts that are implementing the `readline` functionality for each of the exercises
are also pretty standard for the three exercises here, so I'll just be linking back to
those on Github rather than showing them here.

The most interesting thing I learned from doing these exercises is that JavaScript
parses [semantic versioning numbers](https://semver.org/) as Floats (or Integers)
of the Major version, the first dot as the decimal point, and the Minor version.
It ignores the second dot and the Patch version.
This means that:

```
parseFloat('1.2.3') // returns 1.2
```

I had to write tests for this and that also inspired some of my later usage of regular
expressions to validate proper numeric input provided as strings.
That's also something I'm going to have to refactor for if I ever go back and do that.
The only reason I discovered this is that when I was writing up my tests I realized
that it should cause the functions to throw an error and I had no idea if it would
or not.

This was just something that popped into my head and I'm glad I wrote that test because
JavaScript is perfectly happy to pretend that these are numbers and will do its
best to accomodate them as such.

I also tested this in Python and R, and both of them generated errors when attempting
to convert the semver string to some type of numeric, which is what I would have
expected from JavaScript as well.

# Exercise 7 - Area of a Rectangular Room
The most fancy thing here is the use of the `*=` infix operator to convert
from imperial to metric in the case that the `units` required is metric.

[Here's the `readline` function calling it.](https://github.com/jeremy-j-ackso/exercises-for-programmers-nodejs/blob/master/ch3/area-of-rectangle-room.js)

```javascript
/* eslint comma-dangle: "off", comma-spacing: "off" */

function sqft(length, width) {
  const invalids = [null, undefined,]
  if (invalids.includes(length) || invalids.includes(width)) {
    throw new Error('inputs must not be null or undefined')
  }

  if (!parseFloat(length) || !parseFloat(width)) {
    throw new Error('length and width must be digits')
  }

  return `${areaOfRectangle(length, width, 'feet')} square feet`
}

function sqmeters(length, width) {
  const invalids = [null, undefined,]
  if (invalids.includes(length) || invalids.includes(width)) {
    throw new Error('inputs must not be null or undefined')
  }

  if (!parseFloat(length) || !parseFloat(width)) {
    throw new Error('length and width must be digits')
  }

  return `${areaOfRectangle(length, width, 'meters')} square meters`
}

function dimensions(length, width) {
  const invalids = [null, undefined,]
  if (invalids.includes(length) || invalids.includes(width)) {
    throw new Error('inputs must not be null or undefined')
  }

  if (!parseFloat(length) || !parseFloat(width)) {
    throw new Error('length and width must be digits')
  }

  return `You entered dimensions of ${length} feet by ${width} feet.`
}

function areaOfRectangle(length, width, unit) {
  const invalids = [null, undefined,]
  if (invalids.includes(length) || invalids.includes(width) || invalids.includes(unit)) {
    throw new Error('inputs must not be null or undefined')
  }

  if (!['feet', 'meters'].includes(unit)) {
    throw new Error('units must be either \'feet\' or \'meters\'')
  }

  if (!parseFloat(length) || !parseFloat(width)) {
    throw new Error('length and width must be digits')
  }

  let area = length * width
  if (unit === 'meters') area *= 0.09290304
  return area.toFixed(3)
}

module.exports = {
  dimensions,
  sqft,
  sqmeters,
  areaOfRectangle,
}
```

# Exercise 8 - Pizza Party
This one has you doing both integer division and grabbing remainders!
I also like that I had to write the `pluralizer()` function to correctly
decide what word to use based on the number values.
Also, `aboutTheParty()` has some extreme string templating going on.

[Here's the `readline` function calling it.](https://github.com/jeremy-j-ackso/exercises-for-programmers-nodejs/blob/master/ch3/pizza-party.js)

```javascript
function remainingPieces(people, pizzas, pieces) {
  if (typeof people !== 'string' || typeof pizzas !== 'string' || typeof pieces !== 'string') {
    throw new Error('arguments to this function must be provided as strings')
  }

  if (!Number.isInteger(parseFloat(people)) ||
    !Number.isInteger(parseFloat(pizzas)) ||
    !Number.isInteger(parseFloat(pieces))) {
    throw new Error('arguments must be parseable integer digits')
  }

  const total_pieces = pizzas * pieces
  const pieces_remain = total_pieces % people
  if (pieces_remain === 1) {
    return `There is ${pieces_remain} leftover piece.`
  }
  return `There are ${pieces_remain} leftover pieces.`
}

function piecesPerPerson(people, pizzas, pieces) {
  if (typeof people !== 'string' || typeof pizzas !== 'string' || typeof pieces !== 'string') {
    throw new Error('arguments to this function must be provided as strings')
  }

  if (!Number.isInteger(parseFloat(people)) ||
    !Number.isInteger(parseFloat(pizzas)) ||
    !Number.isInteger(parseFloat(pieces))) {
    throw new Error('arguments must be parseable integer digits')
  }

  const total_pieces = pizzas * pieces
  const pieces_each = Math.floor(total_pieces / people)
  return `Each person gets ${pieces_each} pieces of pizza.`
}

function aboutTheParty(people, pizzas, pieces) {
  if (typeof people !== 'string' || typeof pizzas !== 'string' || typeof pieces !== 'string') {
    throw new Error('arguments to this function must be provided as strings')
  }

  if (!Number.isInteger(parseFloat(people)) ||
    !Number.isInteger(parseFloat(pizzas)) ||
    !Number.isInteger(parseFloat(pieces))) {
    throw new Error('arguments must be parseable integer digits')
  }

  return `${people} ${pluralizer('people', people)} with ${pizzas} ${pluralizer('pizzas', pizzas)}, each pizza having ${pieces} ${pluralizer('pieces', pieces)}`
}

function pluralizer(type, val) {
  const plurals = {
    people: ['person', 'people'],
    pizzas: ['pizza', 'pizzas'],
    pieces: ['piece', 'pieces'],
  }

  if (!Object.keys(plurals).includes(type)) {
    throw new Error('type must be one of [\'person\', \'pizzas\', \'pieces\']')
  }

  if (typeof val !== 'string' || !Number.isInteger(parseFloat(val))) {
    throw new Error('val must be a parseable integer digit supplied as a string')
  }

  let lookup_val = val - 1
  if (lookup_val > 1) lookup_val = 1
  return plurals[type][lookup_val]
}

module.exports = {
  remainingPieces,
  piecesPerPerson,
  aboutTheParty,
  pluralizer,
}
```

# Exercise 9 - Paint Calculator
This is another "area of a rectangle" calculator since it's just focused on the
ceiling of a room.
It does impose some nice rounding requirements though.

[Here's the `readline` function calling it.](https://github.com/jeremy-j-ackso/exercises-for-programmers-nodejs/blob/master/ch3/paintCalc.js)

```javascript
function paintCalculator(length, width) {
  if (!/^[0-9]+\.?[0-9]*$/.test(length) || typeof length !== 'string') {
    throw new Error('length must be a parseable number provided as a string')
  }

  if (!/^[0-9]+\.?[0-9]*$/.test(width) || typeof width !== 'string') {
    throw new Error('width must be a parseable number provided as a string')
  }

  const galPerSqFt = 1 / 350
  const sqft_num = dimensions_paint(length, width)

  return Math.ceil(sqft_num * galPerSqFt)
}

function dimensions_paint(length, width) {
  if (!/^[0-9]+\.?[0-9]*$/.test(length) || typeof length !== 'string') {
    throw new Error('length must be a parseable number provided as a string')
  }

  if (!/^[0-9]+\.?[0-9]*$/.test(width) || typeof width !== 'string') {
    throw new Error('width must be a parseable number provided as a string')
  }

  const length_num = parseFloat(length)
  const width_num = parseFloat(width)
  return length_num * width_num
}

module.exports = {
  paintCalculator,
  dimensions_paint,
}
```
