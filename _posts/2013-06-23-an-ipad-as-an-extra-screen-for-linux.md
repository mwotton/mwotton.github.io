---
layout: post
title: "an iPad as an extra screen for Linux"
description: "or, digital Rube Goldberg machines for fun and profit"
category:
tags: [hacks ipad silliness]
---
{% include JB/setup %}

When I'm home, I am surrounded by monitors. It is like returning to a
Mission-Control-influenced womb. Still, while I am definitely a fan of
being gently irradiated by millions of pixels, I am also a fan of
travel, and there is a drastically limited number of 27" monitors that
will fit into your average backpack. Hence, I've been thinking for a
while about ways to use the lovely screen on my iPad as an extra
terminal. I usually take it anyway, so it's not even costing me extra
luggage space.

The harder question is how to do it. The most obvious method would be
to connect a VNC client to a given session on my laptop, then set up
Synergy to connect from that session to a local server. This setup
means I get a full X session on the iPad. The drawback is that VNC
tends to be horribly slow over wireless.

Another possible solution would be to use something like iSSH on the
iPad, and connect Synergy directly using a client on the iPad. This
runs into problems too: first, the only available Synergy client for
the iPad seems to require jailbreaking, and doesn't appear to work
reliably anyway. Foiled again!

Not all is yet lost for our hero, however. These two methods can be
combined to create a rickety construction of fearsome Goldbergian
beauty. We'll need one more component to make this work: tmux, which
will allow you to connect two terminals to the same shell session.

First, we set up a VNC server and client. "apt-get install vnc4server
xvnc4viewer" will do on Ubuntu. We start it up: "vncserver :1", and
connect with "xvnc4viewer".

Now, we can use Synergy (again, apt-get install synergy) to connect the :1
display to our local server. My conf looks a bit like this:

    section: screens
      left:
      rhino:
    end
    section: links
      rhino:
        left = left
      left:
        right = rhino
    end

I run "synergys -f" in a terminal on my normal desktop, then "synergyc
-n left localhost" in a terminal inside the vnc session. I can now
transfer the mouse between the two sessions.

The final part of the puzzle is tmux (apt-get install tmux). I can
then create a tmux session inside the VNC session with

    tmux -S /tmp/pair

(You might consider opening up the permissions on /tmp/pair if you
were sharing with another person, but we're only sharing with
ourselves so it should be fine as it stands.)

I then install iSSH on the iPad, and ssh into my box. I then run

    tmux -S /tmp/pair attach

And we are now done! When I move my mouse left off my main screen,
while I don't see the actual cursor, the shell in the window gets
keyboard focus, and I can run irc, compilation, or whatever other
textual glory I desire. All the other windows can be exiled to a
virtual desktop gulag I don't visit (or more professionally set up to
run headlessly somehow), and strangely, the whole setup seems to work
remarkably well.