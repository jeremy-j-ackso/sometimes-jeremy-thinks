---
title: "Experimenting With GitLab CI"
date: 2018-04-03T20:57:42-06:00
draft: false
tags: ["continuous integration", "git", "gitlab", "docker", "docker-compose"]
summary: "Setting up a NodeJS project to work with GitLab CI and docker-compose."
---

So last weekend I really buckled down on a project that I started a couple weeks
ago before I came down with a really bad cold and was laid-up for a couple days.
I was trying to figure out a bunch of different things: how to use Docker in my
web development workflow, how to make use of GitLab CI in that workflow, and
more things in the vein of converting my workflow to be more amenable to Continuous
Integration.
This last weekend was the first in many that I was fully feeling myself, so I
decided to finish that up and really figure out what I was failing to grasp before.

{{% toc %}}

# What I wasn't grokking about Docker.
I'm very familiar with setting up Vagrant boxes and using them to prototype my
deployment environments.
I've even dabbled a little bit into Chef for configuration management, though
I'm not totally grokking that either (there may be an upcoming post on Ansible
because I think I might be able to grok that over a couple of weekends).
What I was having trouble with in Docker was how to make all the containers
work together and how they should talk to each other.

My web development projects thus far have all been small-scale.
Each one can happily reside on a modest server and perform very well, and that
includes the whole OS, webserver, database, and application server.
Because of that, everything is talking to each over `localhost`, which makes
networking and most security issues extremely easy to deal with.
However, all of my previous attempts to dockerize even one part of this stack
have totally failed because of networking issues.
I just couldn't figure out how to make my locally-running application talk to
a dockerized database.
Because of those troubles I figured that trying to dockerize each or most of
those pieces would end up totally failling.
I was probably right at the time, but mostly due to my then (and still) very
limited knowledge of container networking and orchestration.

# What I wasn't grokking about GitLab CI.
I knew at some level that GitLab CI was also using Docker to achieve things,
and because of my previously-mentioned problems in Docker networking and
orchestration and not knowing how to build my applications to work for Dockerized
applications and databases, I just kept failing and failing to get GitLab CI
to work.
This was before I discovered that you could just run arbitrary bash in many of
these containers.

# What happened last weekend.
Last weekend, I decided to fix all of this and I knew that I had to start with
Docker.
I've been following Docker on Twitter and seeing a lot about `docker-compose`.
So I read the [Getting Started guide](https://docs.docker.com/compose/gettingstarted/)
and it immediately seemed to make sense.
I decided to convert the repo I had been working on to use `docker-compose`.

## First Steps
My first step was to just grab a generic ubuntu docker image, run all the commands
to install Node, install my dependencies, and get the Express server running.
I was able to get a minimal example that didn't rely on a database or any other
services running in under an hour, and when I succeeded I was over the moon
with joy.
I also suspected that I may have had enough to make running a minimal test suite
in GitLab CI possible.
So I looked at the documentation for that, watched a couple of webinars put on
by the GitLab sales staff, and immediately saw the similarity between how you
set up `docker-compose` and how you set up GitLab CI.
I got GitLab CI running with passing jobs on my next push and got that rush
of joy again.
Why didn't things fall into place like this last time?

## Adding a database
I immediately started reading up on services in `docker-compose`, again noticing
the similarity to the GitLab CI documentation on how services work.
I was able to get a `couchdb:2.1.1` container to talk to my ubuntu container
very quickly, just testing it with `curl`, and based on that messing around I
came to the conclusion that I could set up some Express routes to talk to this
datbase.
So I did.

I wrote tests for it as well, and figured out how I could run arbitrary commands
on my containers so that I could do local runs of my test suite.
This also solved another problem that had been vexxing me for a long time: I no
longer needed to mock a database for my test suite.
I could just set up an empty docker container with my database all set up to use
for testing against, which would be more like real life anyhow.
Sure it would take a little longer to get the containers running than using a
database mocking library, but we're talking about seconds, so it's not really
that big of a deal.

I also duplicated this service configuration over to my `.gitlab-ci.yml` file,
since it had so many similarities to my `docker-compose.yml`.
I crossed my fingers, pushed to my repo, and was totally awed when I saw the
correct couchdb image spinning up in the CI job.
I think that particular job failed, but it was probably because of a failing
test rather than the containers not correctly talking to each other.
It was a fixable problem.

## Getting those dope-ass badges
I've been in awe of badges for build pipelines, code coverage, etc., that I've
seen on other projects for a couple of years now and I noticed that the project
settings in GitLab had some info on them, so I decided it was time to read up
on that.
I was able to grab the pipeline badge immediately.
The code coverage badge needed a little bit of work though.

GitLab suggested using `tap`.
I was already using `mocha` for testing, and didn't really want to switch.
I didn't want to rewrite my tests and I use `mocha` at work as well, so I
decided I had to find another way.
I discovered that `tap` uses `istanbul` via `nyc` for code coverage, which
explained the regex that GitLab suggested.
I was able to very quickly get `nyc` stuck into my `test` npm script, and
with some experimenting with the different reporters, was able to get the
kind of output that would fit the regex GitLab was suggesting for use with
`tap` (it's the `text-summary` reporter, btw).
A little bit of configuration on the GitLab side later and I had a code
coverage badge gracing my `README.md` alongside my `build` badge.

## Adding on Code Climate
While doing research on code coverage their was also a lot of talk about
Code Climate.
I knew they had their paid tooling, but didn't totally get what was being
done in the GitLab webinars and documentation to make it work.
I also didn't really know how to interpret it (I'm still fuzzy on this for
the back end case, but it's clearer for the front end case).
Anyways, not finding any GitLab documentation for how to customize it,
I just copy/pasted GitLab's suggested Code Climate `.gitlab-ci.yml` configuration
and it somehow worked in my project.

Like I mentioned, GitLab's built-in tooling for interpreting the `codeclimate.json`
artifact is really great for front end projects, but this being totally a
back end project for the moment, it was just really hard to read and interpret
the JSON.
Maybe that will be corrected in the future.

## Going even further
I decided that running the full installs of everything each time I had to build
my containers or run my CI jobs was taking way too long, so I thought I should
probably do at least a minimum of research to see if there was a pre-existing
Docker container that could run my application server after only doing an
`npm install`.
Lo and behold, three whole minutes of searching yielded the `node:8` container.
I implemented that, which made my `Dockerfile`, `docker-compose.yml`, and
`.gitlab-ci.yml` files all smaller, and life immediately got better.

# Where to find it.
The repo is public, as are all of the runs of the pipeline.
I also put a bunch more detail into the `README.md`, talking more about the
"why" of CI, and the instructions for `docker-compose` and GitLab CI.
You can find it [here](https://gitlab.com/jeremy.jackson/experimenting-with-cicd).

Feel free to clone it, open issues for my abundant and obvious errors,
or even contribute a fix.

# What's next?
I have three more things I intend to accomplish with this project.

1. Incorporate a front end that also gets tested, code quality and coverage reports,
and gets built/minified into an artifact.
2. Set up a dedicated GitLab CI runner. I'm considering a Raspberry Pi for this, though I do
have a couple of older machines that I've picked up from friends on the cheap that I
could convert for this purpose as well.
3. Set up Continuous Deployment. Automating deployment would be tight "AEE EFF" (as the
kids say).
