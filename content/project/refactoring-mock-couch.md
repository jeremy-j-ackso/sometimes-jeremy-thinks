---
title: "Refactoring Mock Couch"
date: 2018-01-06T15:48:48-07:00
draft: false
tags: ["refactor", "nodejs", "javascript"]
summary: "I'm working on refactoring the mock-couch package. Here's how it's going."
---
So I'm working on a boilerplate and website that relies on an
[Apache CouchDB](https://couchdb.apache.org)
database.
The site boilerplate, site, and database all work pretty well, but I want to use
some CI/CD tools to run testing.

I have a couple of options open to me such as using configuration management to set
up the whole environment, using Docker for key pieces of the environment, or finding
some libraries that mock some of my required components, like CouchDB.
After looking into what I would have to learn in order to implement each of them, I
ended up deciding that for the near-term just going with mocking libraries would be
the easiest.

{{% toc %}}

# Why refactor this? Where did this idea come from?
Since the site is using [Redis](https://redis.io) for the cookie store, we're using
the excellent [redis-mock](https://www.npmjs.com/package/redis-mock) for that.
Almost entirely based on the hilarious logo for it (not actually true), we quickly
decided on [mock-couch](https://www.npmjs.com/package/mock-couch) to mock our CouchDB.
There was one problem though: the current version of mock-couch doesn't run with NodeJS 8.x.

So I decided to refactor it in order to bring the packages up-to-date, hopefully
get it working with current versions of NodeJS, and take advantage of newer
language features.

I forked it a few months ago and have been working on it on-and-off
[here, under the `update-packages` branch.](https://github.com/jeremy-j-ackso/mock-couch/tree/update-packages)
After I'm done I'm going to submit it back to the original author to see if he'll
accept my PR for this.
If he doesn't we'll see where this goes from there.
Maybe I'll continue development as a new package if the merge request is declined?

# Error Analysis
When I was initally trying to use this package, it always immediately threw a bunch
errors.
I was able to trace the errors back and found an [open issue](https://github.com/chris-l/mock-couch/issues/53)
that was related.
I think that it's mostly being caused by an older version of [`restify`](http://restify.com/) and the
[`spdy`](https://www.npmjs.com/package/spdy) package that it relies on.
A subsequent commenter on that issue discovered that this problem is found only
with NodeJS > 7.x, so that means my NodeJS 8.x project is impacted.
So far, that commmenter's pull request has not been merged in even though it's a
fairly minor change.

# Why not stop if somebody already made a Pull Request for your problem?
Because I think there are some other benefits to a deeper refactor in this case.

Case in point: the [`ramda`](http://ramdajs.com/) package has evolved very much since the version this
was written for and current versions of `ramda` actually break this package.
Sure, that older version works, but this package also has reliance on it in ways that don't
simplify the code, and in some cases make it more complex for developers unfamiliar
with `ramda`.
In fact, I found that in many cases the vanilla JavaScript solution was easier to
read and understand than the `ramda` implementation.
This holds most true for the uses of `ramda`'s `Compose()` function.

# Abuse of `ramda.Compose()`
`ramda.Compose()` lets you chain a bunch of function calls together, each one putting
its outputs into the next one's input.
Except it's inverted.
Which isn't intuitive **at all!**
You end up having to read UP the page in order to understand how the code block flows.

This might be acceptable if some very complex things were going on, but most of what
was going on could easily be solved with plain `Array` methods.

# Package Linting
The package as written uses `jslint`, which is great.
However I kind of prefer `eslint` and (in my opinion) the better tooling and
configuration that it comes with.
So `jslint` has been replaced with `eslint`.

# Things that I'm leaving alone... for now.
For the moment, this is using `jasmine` tests.
I somewhat prefer `mocha`, but I want to leave `jasmine` in place at least until I'm
once again passing all tests after the refactor.
Maybe at that point I'll replace it with `mocha`.

I'm also leaving `grunt` in place as the task runner.
It isn't a very complex `gruntfile` at this point, so it may ultimately not be needed.
I could probably replace grunt with just `npm scripts` with how simple they are.

# Working on the code
Since I really wanted this to be totally up-to-date I started off by doing an
`npm update`.
This updated all of the packages to their most current versions and this is where I
discovered the breaking changes that more modern versions of `ramda` introduce.
I left everything in place as it was for the moment.

`eslint --fix` did a ton of work in whipping all of this code into shape.
After it's initial pass to fix things that are pretty straightforward I then loaded
everything in `/lib` into my `vim` buffers and started working through the remaining
lint errors.

As I went along I also tried to update the broken `ramda` code, but time after time
I discovered that what was going on in the `ramda` calls was pretty basic stuff that
could quite easily be done using vanilla JavaScript, often with `Array` methods.
So after replacing a couple of `ramda` calls with vanilla equivalents I did a quick
peak at all of the other places using `ramda` and discovered that pretty much all of
them were trivial replacements, and there were only a couple of instances where I would
have to go a bit beyond "trivial" to convert things to vanilla JS.
So I came to the conclusion that `ramda` wasn't really needed and removed it from
the package dependencies.

With that out of the way I continued working through all of the buffered files, fixing
linting errors and replacing now-defunct `ramda` calls with vanilla JS.

# Working on the tests
Once again `eslint --fix` was a lifesaver here.
It fixed a bunch of formatting issues in the tests, and the remaining ones were pretty
simple to deal with.
Mostly things like using destructuring, having individual `let` and `const` assignments
instead of chaining assignments with commas, implementing arrow functions for unnamed
functions, stuff like that.

# Running the tests
This is where I'm at now.
It isn't as bad as I was thinking it would be.
Roughly 50% of the tests are passing, which means that I probably just made some
goofy errors in a couple of key places as I was converting away from `ramda`.
I'm about to get into that now.

# Wrapping up
Once I finish up getting all of the tests to pass I'll update with a new post,
sumbit the merge request and see where this goes after that.
At the very least I'll be able to make use of it for my purposes, so definitely looking
forward to that.
