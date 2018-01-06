---
title: "Exercises for Programmers 4 - NODEJ"
date: 2018-01-05T21:14:35-07:00
draft: false
tags: ["programming exercises", "nodejs"]
summary: "Working throught exercise 10 of chapter 3 in NodeJS."
---
As I mentioned in the previous post, chapter 3 is getting a bit more complex.
I'm through Exercise 10, and that one ended up being pretty complex.
I ended up breaking out the functionality as lowly as I reasonably could
just to help control the complexity.
Part of the outcome of that is that I also had to write waaay more tests.
I'm right around 750 lines in my chapter 3 test file, and around 225 of those
lines are just for this one exercise.

{{% toc %}}

# Working up the functions and tests
I ended up writing 7 functions for this exercise just to produce outputs,
most of which are fairly short and highly specific to their purpose.
For instance, rather than calculating a subtotal for an item as `price * quantity`
I converted it into a function call that does the math, but also takes care
of decimal precision.
Doing this for as many feature aspects as possible helped me keep my main
function that produces the final output fairly clean.
It also increased testability of each component piece so that I could
more easily test for edge cases.
So even though I ended up writing more tests and test cases, reasoning
about the tests and what the functions were supposed to do became **much,
MUCH** easier.

I also decided to break out some additional pieces of functionality that I
could see a lot of people leaving in the main function. One is responsibile for
deciding if a user input is valid.
It performs several tests to either throw one of three errors or return `true`.
Breaking this out and calling it as part of the main function let me keep
my `readline` functions a bit cleaner and also minimized the complexity of
generating errors from bad inputs.

The other function that others might have left in is responsible for building
the output string.
It takes numeric values calculated by other functions, sticks them in an array
of template strings, and then returns those strings joined together with
newline `\n` characters.

# Dealing with user input and eliminating callback hell
The last big takeaway I got from working through this exercise was in dealing
with user interaction via `readline`.
I had mentioned in one of the earlier Exercises for Programmers posts that
I was seeing a very strong possibilty of having to deal with callback hell
in working with `readline.question()`.
That hell almost became realized in working on this exercise.
The author has you asking two questions for each of three items: "What's the
price?" and "What's the quantity?"
This is an obvious situation where code re-use shows some benefits as well.
So between code re-use and the threat of callback hell I decided to pay
special attention to how this would interact with users and how I could
streamline and simplify my `readline`.

I considered several different possible courses of action, noting that
`readline` produces Streams, but after doing some reading into Streams,
Promises, and Generators (the three most likely places a solution for this
would come from) I settled on Promises as the correct way to handle this.
Since I have two questions, I wrote two functions that each returns
a `new Promise()` to ask them.
Since Promises can be chained together using the `then()` method, and the
output of each link in the chain is passed to the next `then()` call I
conveniently end up with a nice way to build up an Object piece by piece
that can ultimately be passed into my main funciton that I've already written.

Each question also has to mention which "item number" it's asking about.
Since the book's requirements specify that we're only concerned about 3
items, and not a potentially infinite number, I decided that the most prudent
thing to do would be to explicitly chain all 6 questions together.
It also became simpler to just make the "item number" an argument to
the question functions.
This saved me from having to write code that would analyze the Object
passed into it in order to figure out what "item number" it would be
asking about.

# Things I would change for the future
I think that how I'm dealing with the `readline` functionality is really
good, but it could definitely be improved.
I currently don't have anything going on with the `reject` statements
in the Promises, mostly because I haven't given much thought about what
it would mean for possible future user input to be rejected.
In this case it may not be necessary.

I could also do more to clean up the input validation and error generation.
For instance, moving the input validation closer to the `readline`
statements would help in generating and recovering from errors earlier.
Currently it only has the possibility of generating errors after it has
already collected all of the information from the user.
I'd like to throw the error sooner and also create a way to recover from
those errors and re-ask an errored question with some guidance to the
user about valid inputs.

Lastly, `selfCheckout()` has some ugly spots.
It works, but I wouldn't call it the most well-written and clear function
I've ever written.
It could use a re-write, which could eventually turn into some of the
functionality built into it being pushed out to other functions.

# Self Checkout `readline`
So here's my `readline` module that is implemented with Promises so
that we get that nice chaining to forward the data on to subsequent
questions.

```javascript
#!/usr/bin/node

const readline = require('readline')
const { selfCheckout } = require('./ex10.js')

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
})

enterPrice({}, 1)
  .then(passedData => enterQty(passedData, 1))
  .then(passedData => enterPrice(passedData, 2))
  .then(passedData => enterQty(passedData, 2))
  .then(passedData => enterPrice(passedData, 3))
  .then(passedData => enterQty(passedData, 3))
  .then((passedData) => {
    const output = selfCheckout(passedData)
    console.log(output)
    rl.close()
  })

function enterPrice(passedData, itemNumber) {
  return new Promise((resolve, reject) => {
    rl.question(`Enter the price of item ${itemNumber}: `, (price) => {
      passedData[`item${itemNumber}`] = { price }
      resolve(passedData)
      reject()
    })
  })
}

function enterQty(passedData, itemNumber) {
  return new Promise((resolve, reject) => {
    rl.question(`Enter the quantity of item ${itemNumber}: `, (qty) => {
      passedData[`item${itemNumber}`].qty = qty
      resolve(passedData)
      reject()
    })
  })
}
```

# Self Checkout Fuctions
Here's all of the functions that do the calculations, validate inputs,
and generate outputs.
`selfCheckout()` is the function that brings all of them together and
gets called at the end of the `readline` Promise chain.

```javascript
function itemSubtotal(price, qty) {
  const output = price * qty
  const output_fixed = output.toFixed(2)
  return output_fixed
}

function billSubtotal(subtotals) {
  const output = subtotals.reduce((arr, cv) => {
    arr += cv
    return arr
  }, 0)
  const output_fixed = output.toFixed(2)
  return output_fixed
}

function tax(subtotal) {
  const taxrate = 0.055
  const output = subtotal * taxrate
  const output_fixed = output.toFixed(2)
  return output_fixed
}

function total(subtotal, tax_val) {
  const output = subtotal + tax_val
  const output_fixed = output.toFixed(2)
  return output_fixed
}

function isStringedNumber(input, type) {
  if (!['money', 'quantity'].includes(type)) {
    throw new Error('type must be one of [\'money\', \'quantity\']')
  }

  if ((type === 'money' && !/^[0-9]+(\.[0-9]{2})?$/.test(input)) || typeof input !== 'string') {
    throw new Error('price must be a string digit, either as an integer, or with two decimal places')
  }

  if ((type === 'quantity' && !/^[0-9]+$/.test(input)) || typeof input !== 'string') {
    throw new Error('quantity must be a string integer')
  }

  return true
}

function outputBuilder(subtotal, tax_val, total_val) {
  const output = [
    `Subtotal: $${subtotal}`,
    `Tax: $${tax_val}`,
    `Total: $${total_val}`,
  ]

  return output.join('\n')
}

function selfCheckout(input) {
  const input_keys = Object.keys(input)
  input_keys.forEach((key) => {
    if (!['item1', 'item2', 'item3'].includes(key)) {
      throw new Error('input must be an object with three properties such that Object.keys(input) === [\'item1\', \'item2\', \'item3\']')
    }
  })
  if (!input_keys.includes('item1') || !input_keys.includes('item2') || !input_keys.includes('item3')) {
    throw new Error('input must be an object with three properties such that Object.keys(input) === [\'item1\', \'item2\', \'item3\']')
  }

  const input_vals = Object.values(input)
  input_vals.forEach((val) => {
    const val_keys = Object.keys(val)
    val_keys.forEach((key) => {
      if (!['price', 'qty'].includes(key)) {
        throw new Error('each item must have a price and qty and no other properties')
      }
    })
    if (!val_keys.includes('price') || !val_keys.includes('qty')) {
      throw new Error('each item must have a price and qty and no other properties')
    }

    isStringedNumber(val.price, 'money')
    isStringedNumber(val.qty, 'quantity')
  })

  const subtotals = [
    parseFloat(itemSubtotal(parseFloat(input.item1.price), parseFloat(input.item1.qty))),
    parseFloat(itemSubtotal(parseFloat(input.item2.price), parseFloat(input.item2.qty))),
    parseFloat(itemSubtotal(parseFloat(input.item3.price), parseFloat(input.item3.qty))),
  ]

  const billSub = billSubtotal(subtotals)
  const taxes = tax(parseFloat(billSub))
  const billTotal = total(parseFloat(billSub), parseFloat(taxes))

  const output = outputBuilder(billSub, taxes, billTotal)

  return output
}

module.exports = {
  selfCheckout,
  itemSubtotal,
  billSubtotal,
  tax,
  total,
  isStringedNumber,
  outputBuilder,
}
```
