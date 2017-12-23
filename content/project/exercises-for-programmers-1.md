---
title: "Exercises for Programmers 1 - NodeJS"
date: 2017-12-19T21:07:16-07:00
draft: false
tags: ["programming exercises", "nodejs"]
summary: "Just doing some programming exercises in NodeJS."
---
I decided to start working through exercises in
[Exercises for Programmers](https://pragprog.com/book/bhwb/exercises-for-programmers)
and sticking my solutions and thoughts here.
I definitely encourage you to buy directly from [The Pragmatic Bookshelf](https://pragprog.com).
I really love their books because they have great depth and they often cover a lot
of things that are of interest and value to the technical community, but aren't
just about tech.
A great example of this is
[Pragmatic Thinking and Learning](https://pragprog.com/book/ahptl/pragmatic-thinking-and-learning)
which I've gotten a great deal of value from.

Anyhow, on with the first exercise in the book!

# Chapter 1
The Chapter 1 exercise to kick things off is for creating a tip calculator.
The book wants it to be an interactive program, using `readlines`-type functionality.
I'm opting (for now at least) to just write functions that take the parameters as defined
in the problem, and outputting an object or something that can be fairly easily parsed.
This lets me easily test it.

## Testing
I'm using [Mocha](https://mochajs.org) with [Expect.js](https://github.com/Automattic/expect.js)
for my testing.
I started out with just a couple simple tests to verify that the outputs from known inputs
matched some reference values and used that as my only test until things started working.
From there I started adding in other test conditions, like what should happen if strings or
`undefined` or `null` values are enterred, and added conditionals in my function to check
for and handle these scenarios.

Here's my `ch1.test.js` file:

```javascript
/* eslint no-undef: "off" */

let expect = require('expect.js')
let tipCalculator = require('../ch1')

describe('tipCalculator', () => {
  let ref_tests = [
    { args: [1, 15], expected: { tip: 0.15, total: 1.15 } },
    { args: [15, 20], expected: { tip: 3.00, total: 18.00 } },
    { args: [15.30, 18.5], expected: { tip: 2.83, total: 18.13 } },
  ]
  it('correctly outputs values equal to reference', () => {
    ref_tests.forEach((test) => {
      expect(tipCalculator(test.args[0], test.args[1])).to.eql(test.expected)
    })
  })

  let err_tests = [
    { args: ['one', 15], throws: 'billAmount must be a number, either float or integer.' },
    { args: [1, 'fifteen'], throws: 'tipRate must be a number, either float or integer.' },
    { args: ['one', 'fifteen'], throws: 'billAmount must be a number, either float or integer.' },
    { args: [0, 15], throws: 'billAmount must be greater than 0' },
    { args: [-12, 15], throws: 'billAmount must be greater than 0' },
    { args: [12, -15], throws: 'tipRate must be greater than 0' },
    { args: [12, 0], throws: 'tipRate must be greater than 0' },
    { args: [undefined, 0], throws: 'billAmount must be greater than 0' },
    { args: [0, undefined], throws: 'tipRate must be greater than 0' },
    { args: [undefined, undefined], throws: 'billAmount must be greater than 0' },
    { args: [null, null], throws: 'billAmount must be greater than 0' },
  ]
  it('throw on invalid inputs', () => {
    err_tests.forEach((test) => {
      expect(tipCalculator).withArgs(test.args[0], test.args[1]).to.throwError(test.throws)
    })
  })
})
```

## `tipCalculator` Function
My `tipCalculator` function started out just doing the basic math required to calculate the
tip amount and the total bill amount based on the two inputs.
As it started passing the test showing that it was equal to reference and I added more tests
for bad inputs, it necessarily grew in order to handle those things.
Basically, none of that progression should come as a surprise.

Here's the function in all its glory:

```javascript
function tipCalculator(billAmount, tipRate) {
  let bill = billAmount
  let tipPerc = tipRate

  if (typeof bill !== 'number') {
    throw new Error('billAmount must be a number, either float or integer.')
  }

  if (typeof tipPerc !== 'number') {
    throw new Error('tipRate must be a number, either float or integer.')
  }

  if (bill <= 0) {
    throw new Error('billAmount must be greater than 0')
  }

  if (tipPerc <= 0) {
    throw new Error('tipRate must be greater than 0')
  }

  if (tipPerc < 1) {
    console.warn(
      'You entered a tipRate less than 1.\n',
      'The program will still function with this value, however you should know that it ',
      'expects the tipRate to be in the form of, for example 15%, so you would enter 15 ',
      'not 0.15.\n',
    )
  }

  tipPerc /= 100

  let tip = parseFloat((bill * tipPerc).toFixed(2))
  let total = parseFloat((bill + tip).toFixed(2))

  return { tip, total }
}

module.exports = tipCalculator
```

## Growing
After I was mostly finished I started reviewing my work and the documentation for Mocha and
realized that I could set Mocha to watch for changes in my files, which is super handy.
After adding that into my `package.json` I also realized that I hadn't seen a single `eslint`
message the whole time that I was writing it and realized I had failed to initialize it.
So I got that initialized along with some of my favored `eslint` settings and set about to
correcting my linting errors until I made it to what I've posted here.

# Aftermath
So this was good.
It's been a bit since I've set up a Node project from scratch, and I'm going to keep working
through these exercises and posting on my progress here.

I'm looking forward to working through Chapter 2.
It has 6 pretty basic exercises, so we'll see if I can get them all into one post, or if it
will take a few to get through.

Also, you can check out my repo of all of my code for this series of posts
[here](https://github.com/jeremy-j-ackso/exercises-for-programmers-nodejs).
