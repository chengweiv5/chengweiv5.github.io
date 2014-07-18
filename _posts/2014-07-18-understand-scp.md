---
layout: post
title: Understand SCP
tags:
  - pitfall
---

Suddenly I realized I don't understand [scp](http://en.wikipedia.org/wiki/Secure_copy)
though I have used Linux for about seven years, used scp frequently, shame of me.

The story start when I was trying to run an sshd in a [docker](https://docker.com/)
container, yes, this is the recommended way to using a docker container.

After I have start my docker container like below:

{% highlight bash %}
# docker run -p 12345:22 -it my_docker_image /bin/bash
{% endhighlight %}

After less than a second, I get the docker container shell prompt

{% highlight bash %}
bash-4.2#
{% endhighlight %}

And then I start sshd in docker like below

{% highlight bash %}
bash-4.2# /usr/sbin/sshd -De
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.
{% endhighlight %}

So the docker container is up and running, sshd is up and running and now I can
login to my container through the ssh tunnel like below.

{% highlight bash %}
# ssh localhost -p 12345
root@localhost's password: 
Last login: Fri Jul 18 13:45:49 2014 from 172.17.42.1
root@e6335ba07a7c ~# 
{% endhighlight %}

And I'd like to transfer some files into my docker container, so I use scp to
do that, yes, why not, I'm using scp day to day.

{% highlight bash %}
# scp -P 12345 test.txt localhost:/
root@localhost's password: 
bash: scp: command not found
lost connection
{% endhighlight %}

What?! **bash: scp: command not found**

That's did surprised me! As I see:

- I have sshd started in the docker container
- I can login into the docker container through ssh

Why scp failed?

Is there anything wrong with docker container?

How about this goes in real Linux? I didn't saw this error in my past seven
years, so I though this maybe a docker regression.

However, when I was not busy I study out why it fail and finally got my
answer at [wikipedia](http://en.wikipedia.org/wiki/Secure_copy).

>Normally, a client initiates an SSH connection to the remote host, and
>requests an SCP process to be started on the remote server. The remote
>SCP process can operate in one of two modes: source mode, which reads
>files (usually from disk) and sends them back to the client, or sink
>mode, which accepts the files sent by the client and writes them
>(usually to disk) on the remote host. For most SCP clients, source mode
>is generally triggered with the -f flag (from), while sink mode is
>triggered with -t (to).[2] These flags are used internally and are not
>documented outside the SCP source code.

Yes, sshd doesn't handle any read/write from/to file for scp but just
provides the secure tunnel, so a scp is necessary to start to handle file
copying on both the local end and remote end.
