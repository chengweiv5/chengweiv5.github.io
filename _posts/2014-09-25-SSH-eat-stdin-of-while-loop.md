---
layout: post
title: SSH eats stdin of while loop
tags:
  - ssh
  - bash
---

Bash script is helpful if you want to do something automatically, especially in
batch mode. Recently I want to upgrade a package on several hosts so I write a
small script within it there is a while loop to read hosts from a file and ssh
to every one and run some commands. However, the weird thing is that commands
only done on the first host apparently and the while loop broke.

That did surprised me a little, see below simple scripts

{% highlight bash %}
#!/bin/bash

echo -e "host1\nhost2\nhost3" |
while read host; do
  echo "ssh to $host"
  ssh $host "echo hello"
done
{% endhighlight %}

The output as I expected was

{% highlight bash %}
ssh to host1
hello
ssh to host2
hello
ssh to host3
hello
{% endhighlight %}

However, the out was

{% highlight bash %}
ssh to host1
hello
{% endhighlight %}

Apparently something gone wrong, as I expected if comment the line *ssh $host
"echo hello"*, it works fine and print three *ssh to <host>* lines. So the magic
brought by **ssh**.

Though google is not always here (in China) but it's helpful and I found answer
[here](http://unix.stackexchange.com/questions/107800/using-while-loop-to-ssh-to-multiple-servers)
and [there](http://linux.spiney.org/help_ssh_eating_all_my_standard_input).

So here comes the short answer, ssh eat the stdin of the while loop, so
*host2\nhost3\n* was never sent to the while loop but eaten by ssh, we can
verify that like below.

{% highlight bash %}
#!/bin/bash

echo -e "host1\nhost2\nhost3" |
while read host; do
  echo "ssh to $host"
  ssh $host "cat"
done
{% endhighlight %}

We'll get the output like

{% highlight bash %}
ssh to host1
host2
host3
{% endhighlight %}

A simple solution is use *ssh* with **-n** option to redirect its stdin from
/dev/null like below.

{% highlight bash %}
#!/bin/bash

echo -e "host1\nhost2\nhost3" |
while read host; do
  echo "ssh to $host"
  ssh -n $host "echo hello"
done
{% endhighlight %}

It will works as we expected.

{% highlight bash %}
ssh to host1
hello
ssh to host2
hello
ssh to host3
hello
{% endhighlight %}
