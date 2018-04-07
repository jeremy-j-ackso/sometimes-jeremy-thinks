---
title: "First Steps With Apache Kafka"
date: 2018-04-07T16:35:17-06:00
draft: false
tags: ["kafka", "apache", "vagrant", "data", "data engineering"]
summary: "Apache Kafka is a skill I need to progress. Here are my first steps in setting it up."
---

Apache Kafka is the de-facto streaming platform in use by data engineers right now.
It comes integrated with many of the commercial Hadoop distributions, and is used
extensively at companies that are dealing with event-based data.

So, here's my first steps at messing around with it.

{{% toc %}}

# Yesterday, 2018-04-06
I got some good advice from an experienced data engineer about what I needed to get
into my toolset in order to be a strong data engineer.
His advice was to focus on getting a few examples of working with Apache Kafka and
Apache Spark out there.
So I'm going to do a few posts as I work through that.

# Last Night, 2018-04-06
I built out a `Vagrantfile` that installs and sets up Kafka.
This was just a modification to their [quickstart](http://kafka.apache.org/quickstart)
instructions, and it worked without a hitch.
If you want to check it out you can clone my repo, `git checkout` the `single-broker`
tag, `vagrant up`, login to the machine and pick up with quickstart instructions 
for Steps 3-5.

```
git clone https://gitlab.com/jeremy.jackson/kafka-experimenting.git
git checkout single-broker
vagrant up
```

And Steps 3-5 can be summarized as:

```
# Do these to get to the right spot for the following commands:
vagrant ssh
cd kafka_2.11-1.1.0

# Step 3:
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
bin/kafka-topics.sh --list --zookeeper localhost:2181

# Step 4:
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

# At this point you could optionally enter a few messages for the broker to store.

# Step 5 (You should do this in a second terminal since your first one will be waiting for input):
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

# You should get output from the previous messages, and any additional messages that you give
# to the broker should appear in the consumer's terminal in a very short amount of time.
```

# Today, 2018-04-07
Today, feeling motivated after last night's success, I decided to start working on
a multi-broker set up, still following the quickstart instructions.

I added some code to my `Vagrantfile` to do some configuration customization of each of the
brokers that would be running, did some troubleshooting on my `sed` scripts & skills,
and got that running as well.
Took me a total of about 2 hours because I was being dumb about `sed`, taking lots of breaks, etc.

That is stored at the `multi-broker` tag in git.
This one goes a little further than the `single-broker` tag does by also creating the topic
as part of the initial scripting.

You should be able to just SSH into this version of the vagrant box and immediately
start up the console producer and consumer.
Interesting thing to note about this one that isn't mentioned in the quickstart that I wasn't
too surprised worked well is that the producer and consumer don't need to be attached to the
same broker.
If you connect the producer to the broker on port 9092 you can connect your consumer
to the broker on port 9093 or 9094 (or all of them), and it will work fine.
Part of me wants to connect six terminals to producers and consumers for each of these
brokers and just mess around watching the messages propagate.

# Next
Dinner.

Then I'm considering if I want to head up to [The Alley Cat](http://www.alleycatcoffeehouse.com/)
to keep working into the night on kafka.

I'd like to get the multi-broker example working in a
[vagrant multi-machine](https://www.vagrantup.com/docs/multi-machine/)
so that each broker is running on one host.
I'm not 100% sure how that's going to work out, especially with Zookeeper, since I'm
not sure if I'll need to add some zookeeper configuration for multiple hosts.
I might be wanting to take this in a direction that wouldn't quickly accomplish the advice
I was given, so I might jump right into Spark instead.
Maybe I'll spend an hour or two on it, see how far I get and then decide if I want to bail
on that plan to work on Spark instead.
