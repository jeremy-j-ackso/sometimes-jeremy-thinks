---
title: "XMPP With Node.js"
date: 2018-07-08T00:21:20-06:00
draft: false
tags: ["vagrant", "xmpp", "node", "nodejs", "javascript", "jabber"]
summary: "Creating a simple experimentation XMPP client."
---

XMPP has been around for a long time at this point.  Libraries for most popular
languages exist and work quite well.  There's even an [XMPP standards
body](https://xmpp.org/about/xmpp-standards-foundation.html) that promotes the
official libraries that they suggest based on quality.

The [XMPP.js](https://xmppjs.org) library for JavaScript turns out to be really
good, but somehow really poorly documented. There are a couple of half-baked
examples, but I found that it still took time to get something running that
touched on some of the major features of XMPP.

I've also been working through the 2009
[XMPP](http://shop.oreilly.com/product/9780596521271.do) book from O'Reilly,
which is mostly oriented around clients rather than technical implementations,
but is still good for understanding the theory and operations. My only wish is
that this book would also have contained instructions on setting up a Jabber
server.  It would have come in handy.

Anyhow, below I'm going to do a quick walk-through of what I did to get an
[ejabberd](https://ejabberd.im) server running in
[Vagrant](https://vagrantup.com), connect to it from my laptop using
[Pidgin](https://pidgin.im), and run the "Echo Bot" example from the XMPP book,
but using Node.js instead of Python. The Echo Bot just repeats everything you IM
to it. Nothing too fancy.

The GitLab repo for this project is
[here](https://gitlab.com/jeremy.jackson/use-xmpp) if you don't want to take the
time to implement this yourself.

{{% toc %}}

# ejabberd in Vagrant
I ended up just setting up a quick and easy Ubuntu 16.04 server in Vagrant. I
used the `ejabberd` that was in the repo and everything went great.
With some minimal configuration and user adding in the setup script everything
runs very well.

## Here's my `Vagrantfile`
Take special note of the forwarded ports. Port 5280 is for the web admin panel,
and port 5222 is the TCP port that ejabberd talks over. There's several
different protocols that you can use, like UDP, based on your own configuration.

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"

  config.vm.network "forwarded_port", guest: 5280, host: 5280, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 5222, host: 5222, host_ip: "127.0.0.1"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = "1"
  end

  config.vm.provision "shell", path: "setup.sh"
end
```

## Here's my `setup.sh`
Nothing too crazy here. The `sed` lines are just modifying some relevant
configs in `ejabberd`'s config file.

```bash
apt-get update

apt-get install -y ejabberd

# It defaults to IPv6, but I want IPv4 just to keep things simple here.
sed -i 's/    ip: "::"/    ip: "0.0.0.0"/g' /etc/ejabberd/ejabberd.yml

# This is just setting up the admin user.
sed -i 's/         - "": "localhost"/         - "admin": "localhost"/' /etc/ejabberd/ejabberd.yml

systemctl restart ejabberd

# I was surprised that the restart took long enough that the ejabberdctl
# commands below were failing.
sleep 5

ejabberdctl register admin localhost password
ejabberdctl register echo_bot localhost password
```

# Installing Pidgin
Pidgin is a Jabber-compliant Instant Messaging client for Linux, MacOS, and
Windows. They also provide source code. If you're using MacOS, they do suggest
using [Adium](https://adium.io) instead.

One of the best features of Pidgin (and probably Adium) is that it has an XMPP
Console that you can enable in the Plugins section of the application. That way
you can see all of the XMPP stanzas being passed between the client and server,
and you can even use it to enter raw XMPP stanzas if you so choose. It's
pretty rad.

I'm on Debian 9, so I just used the one in the repo's.

```bash
sudo apt-get install -y pidgin
```

# Node.js Echo Bot
This simple bot will let you subscribe to its status, and in chat will repeat
everything you say. See the comments for some more details.

```javascript
// Client is the actual Client class, xml is a convenience function for building
// valid XML.
const { Client, xml } = require('xmpp.js');

const client = new Client();

// Node doesn't like self-signed certificates. You could also pass this as an
// environment variable.
process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';

// Here we get into the event-based actions.
// I'm logging everything so that you can see what's getting passed through at
// each of these stages. It gets pretty interesting and looking at this helped me a
// ton when doing some debugging.
client.on('error', err => console.log('ERROR:', err.toString));

client.on('status', status => console.log('STATUS:', status));

client.on('input', input => console.log('INPUT:', input));

client.on('output', output => console.log('OUTPUT:', output));

// Most of the magic happens here. You can set up all your conditions for the
// various XMPP stanzas that you receive. You can be creative with this, for
// instance you could have the bot running on a Raspberry Pi and change
// some status LED's based on the message.
client.on('stanza', stanza => {
  console.log('STANZA:', JSON.stringify(stanza.toJSON()));

  // This acts on requests from other clients to watch the status of this bot
  // so that the other client can see whether or not this bot is online.
  if (stanza.is('presence') && stanza.attrs.type === 'subscribe') {
    client.send(
      xml('presence', { to: stanza.attrs.from, type: 'subscribed' })
    );
  }

  // This is doing the echoing.
  if (stanza.is('message') && stanza.attrs.from !== client.jid) {
    stanza.children.forEach(child => {
      if (child.name === 'body') {
        const response = child.children.join('\n');
        client.send(
          xml('message', { to: stanza.attrs.from, type: 'chat' },
            xml('body', {}, response)
          )
        );
      }
    });
  }
});

// When the bot comes online it updates its status to let you know that it's
// ready to talk back to you.
client.on('online', jid => {
  console.log('ONLINE:', jid.toString());
  client.send(
    xml('presence', {}, 
      xml('show', {}, 'chat'),
      xml('status', {}, 'I say everything you do!'),
    )
  );
});

// This is just handling the client authentication.
client.handle('authenticate', authenticate => {
  return authenticate('echo_bot', 'password');
});

// This actually launches the server.
client
  .start('xmpp://localhost:5222')
  .catch(err => console.error('start failed', err.message));
```

# Running it all
To get this whole thing running you need to first launch the vagrant box, then
create a new account in Pidgin for `admin@localhost` using the fake `password`,
and then launch the Echo Bot (`node index.js`). You'll see some console output from the bot
registering with the `ejabberd` server, and then it will just wait. Once it's
waiting, you can "Add a Buddy" in Pidgin, by looking for `echo_bot@localhost`.
The Echo Bot will pick that up so that you can then see its status in Pidgin.
Lastly, you can go ahead and open up a chat with your new `echo_bot` buddy, and
it will repeat everything back to you.

# Conclusion
This library has a couple of other components as well, but what they do and how
to use them I have no idea because the documentation is basically non-existent.
I'll be sure to update here if/when I get to those other features.
