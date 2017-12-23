---
title: "Exercises for Programmers 2 - NODEJS"
date: 2017-12-23T09:40:33-07:00
draft: false
tags: ["programming exercises", "nodejs"]
summary: "Just doing some programming exercises in NodeJS."
---
It took a couple days (because I was doing them in bed before going to sleep),
but I just finished up Chapter 2 of
[Exercises for Programmers](https://pragprog.com/book/bhwb/exercises-for-programmers).

{{% toc %}}

# Background and Thinking
This chapter is where you actually start in on working on the advertised "57 exercises".
It really does want you to be able to run these exercises as stand-alone command-line
programs (and challenges you to also implement in them in various ways, such as with a GUI),
so I decided to yield and get in on Node's `readline` package.
`readline` is stream-based, so you have a couple of options in how you get it working.
I opted for the callback-style rather than using `pipe`, but if I ever go back and refactor
this stuff I'll be implementing `pipe` so that the code can be cleaner.

In order to make testing easier I broke each exercise down into two pieces: a function that
takes the inputs and produces the desired outputs, and a script that implements `readlines`
to ask the user questions, calls the aforementioned function, and then `console.log()`s the
output.

I also made a single test file for the whole chapter to keep all these exercises together.
I'm still only testing the functions themselves, but I'm investigating methods for testing
the `readline` scripts as well so that I can also test the interactions between the functions
and their calling scripts.
So far I've only found a couple of blog posts that explain how to do it, but the methods
being implemented seem to all be pretty verbose and require some finnagling and/or refactoring
the `readline` into a `Promise`.
I'll probably look into how different packages that provide cli interactivity do this, like
`eslint`.

While reading up on `readline` I also found out about `npm link`, and I'm really excited for it.
`npm link` looks for the `bin` object in your `package.json` and then creates a symlink from
`/usr/bin/<key>` to `~/mydir/<value>.js`.
That way you can have your Node scripts run as if they were actual command line programs
that can be called by name rather than by running `node <file>.js --arguments ...`.
One thing to keep in mind is that `npm link` may break your legitimate functions in `/usr/bin`
by creating that symlink.
To avoid that, I added `-js` to the end of each of my keys in the `bin` object so that it
would never conflict with or break things.
This was the thing I was definitely most excited about learning during this whole chapter.

[My repo](https://github.com/jeremy-j-ackso/exercises-for-programmers-nodejs) for this
continues to grow. I think I'm going to skip branching on this until I start going back for
refactors.

# `package.json`
I'm just going to drop in my `bin` object to keep things short(er).

```javascript
"bin": {
  "hello-js": "./ch2/hello.js",
  "countChars-js": "./ch2/countChars.js",
  "quote-js": "./ch2/quote.js",
  "madlib-js": "./ch2/madlib.js",
  "simplemath-js": "./ch2/simplemath.js",
  "retirement-js": "./ch2/retirement.js"
}
```

# Tests
As I mentioned, I dropped everything from this chapter into a single test file so that they
would be grouped together in my mocha output.

```javascript
/* eslint no-undef: "off", no-sparse-arrays: "off", comma-spacing: "off", comma-dangle: "off" */

const expect = require('expect.js')

const hellof = require('../ch2/ex1.js')
const countChars = require('../ch2/ex2.js')
const quote = require('../ch2/ex3.js')
const madlib = require('../ch2/ex4.js')
const simplemath = require('../ch2/ex5.js')
const retirement = require('../ch2/ex6.js')

describe('ch2', () => {
  describe('ex1.js', () => {
    it('should return values equal to reference.', () => {
      const ref_test = [
        { args: 'Jeremy', expect: 'Hello, Jeremy, nice to meet you!' },
        { args: 'Sue', expect: 'Hello, Sue, nice to meet you!' },
        { args: 'Bob', expect: 'Hello, Bob, nice to meet you!' },
        { args: 'Ana', expect: 'Hello, Ana, nice to meet you!' },
        { args: 'Tom', expect: 'Hello, Tom, nice to meet you!' },
        { args: '5', expect: 'Hello, 5, nice to meet you!' },
      ]
      ref_test.forEach((test) => {
        expect(hellof(test.args)).to.eql(test.expect)
      })
    })

    it('should throw an error if not given a string', () => {
      const err_test = [
        { args: undefined },
        { args: null },
        { args: 5 },
      ]
      err_test.forEach((test) => {
        expect(hellof).withArgs(test.args).to.throwError('name should be a string')
      })
    })
  })

  describe('ex2.js', () => {
    it('should return values equal to reference', () => {
      const ref_test = [
        { args: 'Jeremy', expect: 6 },
        { args: 'bob', expect: 3 },
        { args: 'eleventy-one', expect: 12 },
      ]
      ref_test.forEach((test) => {
        expect(countChars(test.args)).to.equal(test.expect)
      })
    })

    it('should throw an error if not given a string', () => {
      const err_test = [
        { args: undefined },
        { args: null },
        { args: 5 },
      ]
      err_test.forEach((test) => {
        expect(countChars).withArgs(test.args).to.throwError('inputString should be a string')
      })
    })
  })

  describe('ex3.js', () => {
    it('should return values equal to reference', () => {
      const ref_test = [
        { args: ['bob', 'Hi.'], expect: 'bob says, "Hi."' },
        { args: ['Joe', 'Hello.'], expect: 'Joe says, "Hello."' },
        { args: ['R2D2', 'beep-bloop'], expect: 'R2D2 says, "beep-bloop"' },
      ]
      ref_test.forEach((test) => {
        expect(quote(test.args[0], test.args[1])).to.equal(test.expect)
      })
    })

    it('should throw an error if an argument is empty, undefined, or null', () => {
      const err_test = [
        { args: [,] },
        { args: [, 'hi'] },
        { args: ['hi',] },
        { args: [null, null] },
        { args: [null, 'hi'] },
        { args: ['hi', null] },
        { args: [undefined, undefined] },
        { args: [undefined, 'hi'] },
        { args: ['hi', undefined] },
      ]
      err_test.forEach((test) => {
        expect(quote).withArgs(test.args[0], test.args[1]).to.throwError('all arguments must be strings')
      })
    })

    it('should throw an error if an argument is not a string', () => {
      const err_test = [
        { args: [1, 'hi'] },
        { args: ['hi', 1] },
        { args: [1, 1] },
      ]
      err_test.forEach((test) => {
        expect(quote).withArgs(test.args[0], test.args[1]).to.throwError('all arguments must be strings')
      })
    })
  })

  describe('ex4.js', () => {
    it('should return values equal to reference', () => {
      const ref_test = [
        { args: ['dog', 'walk', 'blue', 'quickly'], expect: 'Do you walk your blue dog quickly? That\'s hilarious!' },
        { args: ['cat', 'jump', 'lithe', 'anxiously'], expect: 'Do you jump your lithe cat anxiously? That\'s hilarious!' },
        { args: ['noun', 'verb', 'adjective', 'adverb'], expect: 'Do you verb your adjective noun adverb? That\'s hilarious!' },
      ]
      ref_test.forEach((test) => {
        expect(madlib(test.args[0], test.args[1], test.args[2], test.args[3])).to.equal(test.expect)
      })
    })

    it('should throw an error if an argument is empty, undefined, or null', () => {
      const err_test = [
        { args: [,,,] },
        { args: [, 'hi',,] },
        { args: ['hi',,,] },
        { args: [null, null, null, null] },
        { args: [null, 'hi', null, null] },
        { args: ['hi', null, null, null] },
        { args: [undefined, undefined, undefined, undefined] },
        { args: [undefined, 'hi', undefined, undefined] },
        { args: ['hi', undefined, undefined, undefined] },
      ]
      err_test.forEach((test) => {
        expect(madlib).withArgs(test.args[0], test.args[1], test.args[2], test.args[3]).to.throwError('all arguments must be strings')
      })
    })

    it('should throw an error if an argument is not a string', () => {
      const err_test = [
        { args: [1, 'hi', 'hi', 'hi'] },
        { args: ['hi', 1, 'hi', 'hi'] },
        { args: [1, 1, 1, 1] },
      ]
      err_test.forEach((test) => {
        expect(madlib).withArgs(test.args[0], test.args[1]).to.throwError('all arguments must be strings')
      })
    })
  })

  describe('ex5.js', () => {
    it('should return values equal to reference', () => {
      const ref_test = [
        { args: ['1', '2'], expect: '1 + 2 = 3\n1 - 2 = -1\n1 * 2 = 2\n1 / 2 = 0.5' },
        { args: ['8', '3'], expect: '8 + 3 = 11\n8 - 3 = 5\n8 * 3 = 24\n8 / 3 = 2.67' },
        { args: ['19', '1'], expect: '19 + 1 = 20\n19 - 1 = 18\n19 * 1 = 19\n19 / 1 = 19' },
      ]
      ref_test.forEach((test) => {
        expect(simplemath(test.args[0], test.args[1])).to.equal(test.expect)
      })
    })

    it('should throw an error if the string input cannot be coerced to a number', () => {
      const err_test = [
        { args: ['one', '1'] },
        { args: ['1', 'one'] },
        { args: ['one', 'one'] },
      ]
      err_test.forEach((test) => {
        expect(simplemath).withArgs(test.args[0], test.args[1]).to.throwError('inputs must be digits')
      })
    })

    it('should throw an error if the input is actually a number and not a string', () => {
      const err_test = [
        { args: [1, 1] },
        { args: ['one', 1] },
        { args: [1, 'one'] },
      ]
      err_test.forEach((test) => {
        expect(simplemath).withArgs(test.args[0], test.args[1]).to.throwError('inputs must be digits')
      })
    })

    it('should throw an error if inputs are null or undefined', () => {
      const err_test = [
        { args: [,] },
        { args: [undefined,] },
        { args: [, undefined] },
        { args: [undefined, undefined] },
        { args: [null,] },
        { args: [, null] },
        { args: [null, null] },
      ]
      err_test.forEach((test) => {
        expect(simplemath).withArgs(test.args[0], test.args[1]).to.throwError('inputs must be digits')
      })
    })
  })

  describe('ex6.js', () => {
    it('should return values equal to reference', () => {
      const ref_test = [
        { args: ['20', '40'], expect: 'You have 20 years left until you can retire.\nIt\'s 2017, so you can retire in 2037.' },
        { args: ['20', '60'], expect: 'You have 40 years left until you can retire.\nIt\'s 2017, so you can retire in 2057.' },
      ]
      ref_test.forEach((test) => {
        expect(retirement(test.args[0], test.args[1])).to.equal(test.expect)
      })
    })

    it('should throw an error if the input is actually a number and not a string', () => {
      const err_test = [
        { args: [1, 1] },
        { args: ['one', 1] },
        { args: [1, 'one'] },
      ]
      err_test.forEach((test) => {
        expect(simplemath).withArgs(test.args[0], test.args[1]).to.throwError('inputs must be digits')
      })
    })

    it('should throw an error if inputs are null or undefined', () => {
      const err_test = [
        { args: [,] },
        { args: [undefined,] },
        { args: [, undefined] },
        { args: [undefined, undefined] },
        { args: [null,] },
        { args: [, null] },
        { args: [null, null] },
      ]
      err_test.forEach((test) => {
        expect(simplemath).withArgs(test.args[0], test.args[1]).to.throwError('inputs must be digits')
      })
    })
  })
})
```

# Exercise 1

Exercise 1 was just a simple "Hello Bob!" kind of thing.

Here's `ex1.js`:

```javascript
function hello(name) {
  if (typeof name !== 'string') throw new Error('name should be a string')
  return `Hello, ${name}, nice to meet you!`
}

module.exports = hello
```

Here's it's calling script, `hello.js`:

```javascript
#!/usr/bin/node

const readline = require('readline')
const hellof = require('./ex1.js')

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
})

rl.question('What is your name? ', (name) => {
  const response = hellof(name)
  console.log(response)
  rl.close()
})
```

# Exercise 2
Exercise 2 outputs the number of characters in a string.

Here's `ex2.js`:

```javascript
function countChars(inputString) {
  if (typeof inputString !== 'string') throw new Error('inputString should be a string')
  const chars = inputString.length
  return chars
}

module.exports = countChars
```

Here's it's calling script, `countChars.js`:

```javascript
#!/usr/bin/node

const readline = require('readline')
const countChars = require('./ex2.js')

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
})

rl.question('What is the input string? ', (inputString) => {
  const response = countChars(inputString)
  console.log(`${inputString}: ${response}`)
  rl.close()
})
```

# Exercise 3
This exercise asks two questions, which is where taking advantage of `pipe` with these
streams starts to show its usefulness. You give it a quote and who said it, and
it puts them together in a single string.

Here `ex3.js`
```javascript
/* eslint prefer-template: "off" */

// prefer-template is switched off because the book says to use concatenation.

function quote(author, qte) {
  if (typeof author !== 'string') throw new Error('all arguments must be strings')
  if (typeof qte !== 'string') throw new Error('all arguments must be strings')
  return author + ' says, "' + qte + '"'
}

module.exports = quote
```

Here's its calling script, `quote.js`:

```javascript
#!/usr/bin/node

const readline = require('readline')
const quote = require('./ex3.js')

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
})

rl.question('What is the quote? ', (inputQuote) => {
  rl.question('Who said it? ', (inputSpeaker) => {
    const response = quote(inputSpeaker, inputQuote)
    console.log(response)
    rl.close()
  })
})
```

# Exercise 4
This exercise does a basic madlibs-style of string creation. With 4 inputs, I'm definitely
seeing callback hell start to creep into the `readline` block. 

Here's `ex4.js`:
```javascript
function madlib(noun, verb, adjective, adverb) {
  if (typeof noun !== 'string' || typeof verb !== 'string' ||
    typeof adjective !== 'string' || typeof adverb !== 'string') {
    throw new Error('all arguments must be strings')
  }
  return `Do you ${verb} your ${adjective} ${noun} ${adverb}? That's hilarious!`
}

module.exports = madlib
```

Here's its calling script `madlib.js`:

```javascript
#!/usr/bin/node

const readline = require('readline')
const madlib = require('../ch2/ex4.js')


const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
})

let builder = {}
rl.question('Enter a noun: ', (noun) => {
  builder.noun = noun
  rl.question('Enter a verb: ', (verb) => {
    builder.verb = verb
    rl.question('Enter an adjective: ', (adjective) => {
      builder.adjective = adjective
      rl.question('Enter an adverb: ', (adverb) => {
        builder.adverb = adverb
        console.log(madlib(builder.noun, builder.verb, builder.adjective, builder.adverb))
        rl.close()
      })
    })
  })
})
```

# Exercise 5
This one does the basic four arithmetic functions on two input numbers.
I can probably refactor how I'm producing the output string to be a bit more elegant and shorter
(I'm thinking string concatenation with array destructuring), but this is fine for a
first stab at it.

Here's `ex5.js`:

```javascript
/* eslint no-restricted-globals: "off" */

function simplemath(firstNum, secondNum) {
  if (isNaN(Number(firstNum))) throw new Error('inputs must be digits')
  if (isNaN(Number(secondNum))) throw new Error('inputs must be digits')

  if (typeof firstNum !== 'string') throw new Error('inputs must be digits')
  if (typeof secondNum !== 'string') throw new Error('inputs must be digits')

  const fnum = Number(firstNum)
  const snum = Number(secondNum)

  let mathObj = {
    add: fnum + snum,
    subtract: fnum - snum,
    multiply: fnum * snum,
    divide: fnum / snum,
  }

  mathObj.divide = parseFloat(mathObj.divide.toFixed(2))

  return `${fnum} + ${snum} = ${mathObj.add}\n${fnum} - ${snum} = ${mathObj.subtract}\n${fnum} * ${snum} = ${mathObj.multiply}\n${fnum} / ${snum} = ${mathObj.divide}`
}

module.exports = simplemath
```

Here's its calling script `simplemath.js`:

```javascript
#!/usr/bin/node

const readline = require('readline')
const simplemath = require('./ex5.js')

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
})

let values = {}
rl.question('What is the first number? ', (firstNumber) => {
  values.firstNumber = firstNumber
  rl.question('What is the second number? ', (secondNumber) => {
    values.secondNumber = secondNumber
    console.log(simplemath(values.firstNumber, values.secondNumber))
    rl.close()
  })
})
```

# Exercise 6
This exercise starts incorporating `Date()` and doing math with that. Other than that, it's
pretty much the same as the others.

Here's `ex6.js`:

```javascript
/* eslint no-restricted-globals: "off" */

function retirement(currentAge, retirementAge) {
  if (isNaN(Number(currentAge))) throw new Error('inputs must be digits')
  if (isNaN(Number(retirementAge))) throw new Error('inputs must be digits')

  if (typeof currentAge !== 'string') throw new Error('inputs must be digits')
  if (typeof retirementAge !== 'string') throw new Error('inputs must be digits')

  const years = retirementAge - currentAge
  const curYear = new Date().getFullYear()
  const retYear = curYear + years
  return `You have ${years} years left until you can retire.\nIt's ${curYear}, so you can retire in ${retYear}.`
}

module.exports = retirement
```

Here's its calling script, `retirement.js`:

```javascript
#!/usr/bin/node

const readline = require('readline')
const retirement = require('./ex6.js')

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
})

let values = {}
rl.question('What is your current age? ', (firstNumber) => {
  values.firstNumber = firstNumber
  rl.question('At what age would you like to retire? ', (secondNumber) => {
    values.secondNumber = secondNumber
    console.log(retirement(values.firstNumber, values.secondNumber))
    rl.close()
  })
})
```
