---
title: "Why I Use Vagrant So Much"
date: 2017-12-17T17:03:13-07:00
draft: false
tags: ["vagrant", "virtualization", "simulation", "reproducibility"]
summary: "Vagrant can help you create consistency in how you build and deploy software. Here are some basics."
---
I use [Vagrant](https://vagrantup.com) for a bunch of different tasks.
If I'm forced to use a Windows-based workstation I like to use Vagrant so that I have a standard
Linux virtual machine available to me at all times so that I can do basic types of tasks
that I might need `bash`, `python`, `ruby`, or `gnu-utils` for.
I also use it for what it's mostly intended for: **having a development environment that mimics
my targetted deployment environment in as many ways as possible.**

Suppose that I'm working on a MEAN stack website.
I probably don't want to actually just build it on my machine, especially if I'm working with
someone else.
Yes, my personal machines all use Debian Stable, which should be just fine for development that
targets an Ubuntu VM in the cloud, or on a VPS, but my personal machines also have a bunch of
software installed that has nothing to do with the operation of a website.
I'm talking about things like window and desktop managers, web browsers, my time tracking
application, sqlite, and lots of other software that's out there.

Let's complicate this further and suppose that I'm collaborating on this with a teammate who's
on a Mac.
Just for the heck of it, let's say that we have third teammate helping us out who's on Windows.
They have all of their own stuff running on their machines that works in different and unique
ways that fail to be close in any way to the deployment target.
So in this situation, we need a way to have a common target that we can develop and test towards.
**This is where Vagrant comes to the rescue.**

{{% toc %}}

# About Vagrant
Vagrant ([github](https://github.com/hashicorp/vagrant)) is a tool built mostly in Ruby for
driving Virtual Machines across different platforms in a standardized way.
It integrates well with various configuration management tools like Chef, and is even
capable of driving your VM's in most, if not all, major cloud providers.

I (and my colleagues) predominantly use it to drive VirtualBox on our workstations,
though we're hoping that as our IT org moves more towards infrastructure-as-code we'll
be able to get them to give us standardized images as well as the ability to deploy these
directly to our cloud provider.

# Ways that Vagrant helps our work
So far I've only explained how Vagrant helps my team collaborate on development towards a common
deployment target, regardless of what kind of operating systems we're actually running on our
workstations.
For actually doing the work, Vagrant also helps in bunch of other ways.

We automatically suppose that our ultimate deployment target is just going to be a barebones
Linux VM, probably either CentOS or Ubuntu.
However, we have a bunch of software that we need to install in order to be able to get a running
website, analytics service, or database.
Vagrant helps here because we can create provisioning scripts in something like bash (or a
configuration management system if we were more advanced) and lets us provision, modify scripts,
and reprovision virtual machines on our workstations in very quick iterations.
That way, if something is messed up in the way we configured our server or software we can
quickly detect that, fix it in the script, and then reprovision the VM to test and check it out.
We can even include scripts to do things like configuring our `cron` jobs, setting up `systemd`
so that the process running our website can start on boot and recover from failures,
configure our webserver (like `nginx` or `apache`), and any number of other things that we
would like to happen during server provisioning.
The best part is that we haven't wasted a dime on our actual deployment server yet!

# The Vagrantfile
You configure your VM's using a `Vagrantfile`, which specifies the VM image you want to use,
the size of the VM (think CPU cores and memory), what kind of a networking situation needs to
happen to make it similar to the deployment target (like available ports, etc), and shared
folders (so that you can still just modify all of your code using your local text editor, and
not whatever is available once you login to the VM).
`Vagrantfile`s have a lot of different configuration options, but these are the ones that are
probably most important for a normal user, along with telling it what scripts you want it to
run as part of the provisioning process.
Since your `Vagrantfile` is just ruby code, you get the benefit of being able to include it in
your project's repository, so that it can be easily shared with all the members of your team.

So let's say that you want to use Vagrant for a project.
Doing a basic setup is extremely easy.
You just run `vagrant init` and it will create a barebones `Vagrantfile` for you to modify
as needed.

So, what do you put in your `Vagrantfile`?
Let's make a list about what we know about our website project:

* We need to have port 80 available.
* We plan on using `nginx` and `mysql`.
* We need NodeJS, version 8.x installed.
* Deployment target is Ubuntu 16.04.

When you open your Vagrantfile, it will have a bunch of boilerplate with some sensible defaults
and explanations about common configuration settings that you can have.
I'm going to exclude all of that here and just put in the Vagrant configs that we actually need
to accomplish what's in our list.

```ruby
Vagrant.configure("2") do |config|

  # The bento project provides high quality VM images for a number of Linux OS's.
  config.vm.box = "bento/ubuntu-16.04"

  # We need port 80 on our VM, but let's forward it port 8080 on the laptop.
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Here's our VirtualBox configurations
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = false
  
    # Customize the amount of memory on the VM:
    vb.memory = "1024"
    vb.cores = "2"
  end

  # We could modify this to point at a provision script, but we'll just do it inline.
  # Keep in mind that everything in this block is run as root at /home/vagrant.
  config.vm.provision "shell", inline: <<-SHELL
    # Copied from https://github.com/nodesource/distributions
    curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -

    apt-get update
    apt-get install -y nginx mysql git nodejs
  SHELL

end
```

# Launching Vagrant
Once the Vagrantfile is written we can launch the VM by running `vagrant up` and we'll soon
have a running VM.
Keep in mind that the first time you run `vagrant up` for any particular Virtual Machine Image
(`bento/ubuntu-16.04` in this case), Vagrant will first download the Virtual Machine Image
(called `boxes` in Vagrant parlance) into a default VM library, then create a copy of it to be
configured according to what you've specified.
Because of this, you definitely need a fast internet connection, and ideally should have an SSD
installed to your workstation.
Once you have that box downloaded once, you won't have to download it again, though Vagrant
may let you know when a new version of the box is available for download.

Vagrant will also print out a bunch of messages describing what it's doing at any given moment,
as well as everything that gets printed to `stdout` and `stderr` as a result of the commands
run in your `config.vm.provision` block.
When the provisioning process is complete, Vagrant will return you to your command prompt with
either a success message or a message that there was some kind of failure.
If it was a success, and even in some failure cases, you should now be able to log in to your
running VM by using `vagrant ssh`.
Then, once you're in you can check that everything is as you intended the configuration to be.

# Dealing with misconfiguration
If you do find a problem with your configuration you can start editing your `Vagrantfile` and
provisioning scripts to correct your mistakes.
Once you're done making your changes you reprovision the VM at a level appropriate for the change
you're making.

```bash
# Say you're just changing a minor configuration detail, you can just run:
vagrant reload --provision

# If you made some big changes and want to start with a fresh box:
vagrant destroy
vagrant up
```

Keep in mind that just doing `vagrant reload --provision` will also take less time than destroying
and starting totally fresh, so just be aware of what you're doing and not wasting more time than
you need to in getting your configurations correct.

# Wrapping up
So we've covered some good uses and reasons for using Vagrant, we did a quick demo showing off
how to configure a Vagrant box for use in a project, and how to deal with configuration problems.
Now it's up to you to find novel uses of this great piece of software and start building
good, reproducible software with it!

# Resources
* [Vagrant](https://vagrantup.com)
* [Vagrant on Github](https://github.com/hashicorp/vagrant)
* [VirtualBox](https://virtualbox.org)
* [HashiCorp, the makers of Vagrant](https://hashicorp.com)
* [The Bento Project, by Chef.io](https://github.com/chef/bento)
* [Chef](https://chef.io)
* [Discover other Vagrant Boxes here](https://app.hashicorp.com/boxes/search)
