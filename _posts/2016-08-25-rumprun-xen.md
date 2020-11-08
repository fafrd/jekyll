---
layout: post
title:  "Running rumprun for Xen in Ubuntu 16.04"
date:   2016-08-25 20:57:23 -0700
tags: xen
---

Running rumprun under Xen isn't hard, but it's less documented than running it under KVM. This page is similar to [Rumprun's guide to building rumprun unikernels](https://github.com/rumpkernel/wiki/wiki/Tutorial:-Building-Rumprun-Unikernels) with a few Xen-specific changes.

## Build the rumprun platform
Install prerequisite xen headers and build tools:
{% highlight bash %}
sudo apt-get install build-essential libxen-dev
{% endhighlight %}
Clone their repo, cd, build:
{% highlight bash %}
git clone http://repo.rumpkernel.org/rumprun
cd rumprun
git submodule update --init
CC=cc ./build-rr.sh xen
{% endhighlight %}

## Add binaries to PATH
You've now build rumprun and the binaries necessary for building, baking, running are located in rumprun/bin. You'll want to these to your [PATH variable](https://en.wikipedia.org/wiki/PATH_(variable)) for convenient access:
{% highlight bash %}
export PATH="${PATH}:$(pwd)/rumprun/bin"
{% endhighlight %}
You can also add this to your ~/.bashrc to make these changes permanent.
{% highlight bash %}
vim ~/.bashrc
{% endhighlight %}
Append the following, where [location of rumprun] represents the directory containing rumprun:
{% highlight bash %}
export PATH="$PATH:[location of rumprun]/rumprun/bin"
{% endhighlight %}

## Building applications
Get some source code and use rumprun's version of gcc to compile it. (Follow the [rumprun tutorial](https://github.com/rumpkernel/wiki/wiki/Tutorial:-Building-Rumprun-Unikernels) for a more thorough explanation...)

Here, helloer.c is our source code and helloer-rumprun is the output binary.
{% highlight bash %}
x86_64-rumprun-netbsd-gcc -o helloer-rumprun helloer.c
{% endhighlight %}

## Baking applications
I was going to make a joke here but I can't think of anything clever right now. You need to bake it. That means running a command to add in all the kernel-y bits that makes rumprun ready for it.

Here, helloer-rumprun is the binary we just built and helloer-rumprun.bin is the the binary with the necessary rumprun pieces.
{% highlight bash %}
rumprun-bake xen_pv helloer-rumprun.bin helloer-rumprun
{% endhighlight %}

## Running applications
Here's the hard part. The rumprun command is a script that will create a Xen configuration file in /tmp and start up a Xen PV guest. For xen, it will look like this:
{% highlight bash %}
rumprun -S xen -id -g [Xen config options] -I [network interface] -W [more network options]
{% endhighlight %}
The -I and -W commands can be omitted if there is no need for networking. I have networking set up using a NAT, in which there exists a subnet local to the machine. Look at my [article on Xen networking](/setting-up-nat-networking-in-xen-using-virsh.html) to see how I set up networking within rumprun.
