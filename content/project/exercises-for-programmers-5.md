---
title: "Exercises for Programmers 5 - NODEJS"
date: 2018-01-20T17:18:29-07:00
draft: false
tags: ["programming exercises", "nodejs"]
summary: "Working throught exercise 10 of chapter 3 in NodeJS."
---
I had to take a break from working through the book to get a few other "life" things
done, but with those in the bag I was able to hop back into it today and finish up 
Chapter 3.

After the amount of learning I did with Exercise 10, the remaining three exercises
ended up being really easy and following basically the same pattern in all three
cases.

Repository for this is still
[on GitHub.](https://github.com/jeremy-j-ackso/exercises-for-programmers-nodejs)

Once again, I'm going to leave out the tests from this coverage since it's all
pretty straightforward and follows the same pattern each time.
I was expecting the test file for this chapter to be over 1,000 lines of code, but
somehow it only came out to 990.
It makes me wonder what test cases I overlooked as I was working on these
last three exercises.

I'm also going to leave out the code implementing the `readline` functionality.
Since the introduction of `Promise` chaining that I implemented in Exercise 10
I haven't had to do anything new and interesting there either, and once again
it looks like I've captured a good pattern for how to do that.

I started on these around 11 this morning.
It's now 5:30 PM and I've done a lot of driving and walking during that time in
addition to writing code, so I think I've got a total of maybe 2-3 hours of
actual coding time in today.
So I feel like I made a lot of progress in being able to wrap in the chapter in
a fairly efficient and concise manner.

I also think I've found a pretty good function pattern for checking that numeric
inputs are valid.
I should break that out into a utility module so that I stop re-creating it each
time.
That would help cut down on the number of tests I have to write, and also let me
write less code each time I start on a new exercise.
Breaking it out and updating all of the calls could take some time though.
For the moment I'm more excited to just plow through the exercises.

# Exercise 11
This exercise is a Euro to US Dollar currency converter.
I have no idea if the conversion rates are even correct.
I could probably go and find an API to supply this, but researching that could
have taken a bit more time than I was willing to commit to at this point.
Not that it would have been difficult once I found the API.
It would have actually been a good opportunity to do some practice with `Stream`.
Maybe I'll consider a refactor of that at some point.

```javascript
function checkInput(value) {
  if (typeof value !== 'string') {
    throw new Error('input amount must be delivered as a string')
  }
  if (Number.isNaN(parseFloat(value)) || /\d+\.\d+\.\d+/.test(value)) {
    throw new Error('input must be parseable as a number')
  }
  return true
}

function calculateConversion(amount_from, rate_from, rate_to) {
  if (typeof amount_from !== 'number' ||
    typeof rate_from !== 'number' ||
    typeof rate_to !== 'number') {
    throw new Error('all inputs must be numeric')
  }
  const output = (amount_from * rate_from) / rate_to
  return output.toFixed(2)
}

function buildConversion(amount_from, rate_from) {
  checkInput(amount_from)
  checkInput(rate_from)
  const parsed_from = parseFloat(amount_from)
  const parsed_rate = parseFloat(rate_from)
  const rate_to = 98.24
  const converted_value = calculateConversion(parsed_from, parsed_rate, rate_to)

  return `${amount_from} euros at an exchange rate of ${rate_from} is ${converted_value} US dollars`
}

module.exports = {
  calculateConversion,
  checkInput,
  buildConversion,
}
```

# Exercise 12
This one is a basic interest calculator, without doing any compounding.
That made the math super easy.

```javascript
function simpleInterest(principal, rate, period) {
  const output = principal + (principal * period * rate * 0.01)
  return output.toFixed(2)
}

function checkSIinputs(value) {
  if (typeof value !== 'string') {
    throw new Error('input amount must be delivered as a string')
  }
  if (Number.isNaN(parseFloat(value)) || /\d+\.\d+\.\d+/.test(value)) {
    throw new Error('input must be parseable as a number')
  }
  return true
}

function buildSIstring(principal, rate, period) {
  checkSIinputs(principal)
  checkSIinputs(rate)
  checkSIinputs(period)

  const parsed_principal = parseFloat(principal)
  const parsed_rate = parseFloat(rate)
  const parsed_period = parseFloat(period)
  const accrued = simpleInterest(parsed_principal, parsed_rate, parsed_period)

  return `After ${period} years at ${rate}%, the investment will be worth $${accrued}.`
}

module.exports = {
  simpleInterest,
  checkSIinputs,
  buildSIstring,
}
```

# Exercise 13
This one is for compounding interest.
A little bit more complicated mathematically, but again: not really a problem.

```javascript
function compound_interest(principal, rate, period, frequency) {
  const exponent = period * frequency
  const small_interest = 1 + ((0.01 * rate) / frequency)
  const output = principal * (small_interest ** exponent)
  return output.toFixed(2)
}

function checkCIinputs(value) {
  if (typeof value !== 'string') {
    throw new Error('input amount must be delivered as a string')
  }
  if (Number.isNaN(parseFloat(value)) || /\d+\.\d+\.\d+/.test(value)) {
    throw new Error('input must be parseable as a number')
  }
  return true
}

function buildCIstring(principal, rate, period, frequency) {
  checkCIinputs(principal)
  checkCIinputs(rate)
  checkCIinputs(period)
  checkCIinputs(frequency)
  const p_principal = parseFloat(principal)
  const p_rate = parseFloat(rate)
  const p_period = parseFloat(period)
  const p_frequency = parseFloat(frequency)

  const accrued = compound_interest(p_principal, p_rate, p_period, p_frequency)

  return `$${principal} invested at ${rate}% for ${period} years compounded ${frequency} times per year is $${accrued}`
}

module.exports = {
  compound_interest,
  checkCIinputs,
  buildCIstring,
}
```
