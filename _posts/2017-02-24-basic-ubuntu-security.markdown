---
title: "Basic Ubuntu Security"
layout: post
tags:
- Ubuntu
- Security
category: [Linux, Security]
---

Especially in 2017 everyone should be concerned about security. You don't need to be a genius or completely paranoid in order to avoid most potentially compromising situations. Follow these instructions and you'll have a basic understanding of what it is to secure your brand new, vanilla Ubuntu server. That said, if you use the web or have a publicly accessible service then there is always a chance you will be compromised. You cannot predict application patches with vulnerabilities or the colorful attempts a hacker that is specifically targeting you may employ. You may simply prep with your best effort. <!--more-->Let's begin!

<!--more-->

## Add your first user

{% highlight shell %}
sudo -s
adduser <new_user>
usermod -aG sudo <new_user>
{% endhighlight %}

## Modify the SSHD config

`vim /etc/ssh/sshd_config`

Modify the following line:

{% highlight shell %}
PermitRootLogin = yes
PermitRootLogin = no
{% endhighlight %}

Some blogs may advise you to change the port to a non-standard number (ie: 8022, 2222). I don't believe this is a good idea:
**A.** There is no such thing as security through obscurity
**B.** This opens the door for password interception, assuming you're using plaintext passwords instead of keys. A lot of amateur blogs will use the word non-standard port to reference anything above 1024, however, the proper terminology is a non-privileged port (meaning root does not have to be used to begin listening, any user can listen).

**Ex.** If you use port 2222 I can now create a script to mimic SSH and begin stealing your passwords like the troublesome child I am.

## Setup Uncomplicated Firewall (UFW)
{% highlight shell %}
apt install ufw
ufw allow 22/tcp
{% endhighlight %}

We’ve installed UFW and opened TCP port 22 for ssh connections.

## Configure Automatic Security Updates
{% highlight shell %}
apt install unattended-upgrades
dpkg-reconfigure unattended-upgrades
{% endhighlight %}

You’ll want to select the description:

This package can download and install security upgrades automatically and unattended, taking care to only install packages from the configured APT source, and checking for dpkg prompts about configuration file changes.

## Other considerations

In future articles I’ll probably write some more on this subject. Here’s some more topics worth looking into in security:</h2>

* AppArmor
* Keep in mind Linode does not support AppArmor, you will need to recompile the kernel
* Key based authentication
* Swap file security
* File ownership
* Principle of Least Privilege
