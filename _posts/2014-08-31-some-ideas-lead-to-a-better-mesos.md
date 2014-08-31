---
layout: post
title: Some ideas that lead to a Better Mesos?
tags:
  - mesos
---

[Apache Mesos](http://mesos.apache.org) is a cluster manager that simplifies
the complexity of running applications on a shared pool of servers. There are
more and more applications/frameworks ported to mesos, for example: Hadoop
mapreduce, Storm, Spark, Chronos and so on. However, from my humble opinion,
there are still some ideas to lead to a better mesos.

I'm not going to describe how mesos looks like, how it works and architecture
and every piece of mesos. It has great document to do that on its website. I
will explain some ideas from my point of view that maybe lead to a better mesos.

##Resource Management

There are two kinds of resources in mesos: reserved resources and non-reserved
resources. To reserve any resource for a framework, a framework must has a role,
a role is used to category frameworks. So there maybe more than one framework in
a single role.

###Reserve resources in a pool way

Currently, mesos support reserve resources for any role in a per-slave way like
below.

{% highlight bash %}
export MESOS_resources="cpus(roleA):4;mem(roleA):2048;disk(roleA):10240;cpus(roleB):2;mem(roleB):2048;cpus(roleC):3;mem(roleC):1024;disk(roleC):5120
{% endhighlight %}

To reserve resources for a role, we generally need configure like above on many
mesos slave nodes given that

- A single slave generally can not provides enough resources for a framework
- we may like tasks of a job distributed to many slaves

Reserve resources is a cool feature, but somehow a little low-level, it shows
mesos internal to user and a little tricky to configuration. It's fine if we have
same configuration on all mesos slaves, otherwise, it's a nightmare of
maintenance.

In another way, mesos is trying to be a great cluster manager, provides
computing pool to on-top frameworks, so it's reasonable that if we can reserve
resources in a pool way, for example, user/framework asks mesos to reserve
resources in below way:

{% highlight bash %}
export MESOS_reserved_resources="cpus(roleA):40;mem(roleA):20480;disk(roleA):102400;cpus(roleB):20;mem(roleB):20480;cpus(roleC):30;mem(roleC):10240;disk(roleC):51200"
{% endhighlight %}

Um? Nothing changed except change a name and 10 times resources? Yes, from the
above line, but the change under the hood is that this is not a per-slave
configuration but for the mesos cluster, in another way, for mesos master. So
the user do not care we have reserve resources on which slave, but just want to
reserve some computing resources, why should I care these resources comes from
which slave in the computing pool? Should I?

This is generally much easy to configure, easy to understand, in a nature
resource pool way.

###Handle resource fragment

As we know that mesos has two kinds of resources, reserved resources and
non-reserved resources.

For a framework has a role, for example, roleA, it will receives two kinds of
resource offers from mesos master, reserved resource always comes after
non-reserved resource in an offer. And a good citizen always should uses
reserved resource first and then non-reserved resource in an offer. However,
this is not mandatory.

For example: frameworkA has a role roleA so it well receive offer contains two
kinds of resource like offer:

{% highlight bash %}
cpus(*):8;mem(*):4096;disk(*):102400;port(*):31000-32000;cpus(roleA):6;mem(roleA):3096
{% endhighlight %}

If the framework schedules a task with resource

{% highlight bash %}
cpus(roleA):4;mem(roleA):1024
{% endhighlight %}

So the updated offer becomes to

{% highlight bash %}
cpus(*):8;mem(*):4096;disk(*):102400;port(*):31000-32000;cpus(roleA):2;mem(roleA):2048
{% endhighlight %}

In the next time, frameworkA schedules a task with resource

{% highlight bash %}
cpus:4;mem:3072
{% endhighlight %}

It will uses non-reserved resource rather than use all reserved resource first
and then get some from non-reserved resources.

What's problem? Resource fragment!

There are some possible solutions to solve resources fragment like the above.

- A possible solution is reserving resource on mesos slaves in times of resource
  task may use, however, this will limit the framework flexibility how to
  schedule task.
- Another possible solution is *every* framework take resource fragment into
	mind and has its own code to resolve this problem.

However, both of them are not good enough, if we can move this decision to mesos
and we'll get:

- framework can not use non-reserved resource first
- no resource fragment like above
- transparent to frameworks

We'll see the answer in below section.

###Mandatory use reserved resource first

As described in the above section , we know that mesos
always append reserved resource after non-reserved resource in an offer, so a
good citizen should always use reserved resource first because the others can
not use a framework's reserved resources. However, this is and only is a
recommend way, one may break this way if it like. And we also know that differ
resources into reserved resource and non-reserved resource may cause resource
fragment.

###The answer

It's better if we provides a way that on-top framework can not escape and to use
non-reserved resource first. Um? It's better? And it make the reserve resource
architecture more simple. Take below points:

- Reserve resources in a pool way, no need to configure reserve resources on
  every single mesos slave, just let mesos-master know
- There is only one kind of resource, that is resource in the pool, every
  framework get this resource in an offer, so a framework can not say: I'm
  willing to use non-reserved resource first
- Mesos master record how many resource a framework used and take resources
  used exceeds reserved resources part to calculate DRF resource share
- No resource fragment and nothing changed in framework, mesos does all for
  them
- It's much simple, transparent, stable, effective, especially, All or most of
  changes go into mesos, but not every on-top framework

##Support max share of resources

So far (mesos-0.20.0), there isn't such a feature to limit the max share of
resource of a role. So in theory and in practical a framework can take all
resources from mesos and if it's not a good citizen, it may not release any
resources even it's a good citizen, it may do so, like Storm, which topology
is a generally a long-runing job. In another way, To prevent us from such
situation, we can reserve some resources per role, for example, if there are
3 role in our cluster, we may reserve some resources for every role like below.

{% highlight bash %}
export MESOS_resources="cpus(roleA):4;mem(roleA):2048;disk(roleA):10240;cpus(roleB):2;mem(roleB):2048;cpus(roleC):3;mem(roleC):1024;disk(roleC):5120
{% endhighlight %}

We may do that on very mesos slaves because it's a per-slave configuration not a
cluster-wide configuration.

Reserve resources for roles works fine if all roles have framework instance
running at the moment, however, if some of them have no framework instance
running, it's a waste of computing resource because the running frameworks can
not ask to use reserved resources for other roles. So in general, there are
some problems:

- Without resource reserve for roles, a framework may take all resources in a
  mesos cluster and the new coming framework may wait for resources for a long
  time
- With resource reserve for roles, it may waste reserved resources if no such
  framework is running
- Reserved resources is a mesos-slave configuration which can not change on the
	fly

So how a max share of resources mitigate these problems?

Max share of resources provides a way to not use all the resources in mesos
cluster; Max share of resources provides a way to "reserve" some resources for
common roles to preempt, so not reserved for every of them.

##Support in-framework task preempt

Mesos is a cluster manager which manage cluster computing resources and provides
computing resources as offer to on-top schedulers. Since there are several kinds
of resources: cpu, memory, disk, network port, it's quite hard to schedule all
kinds of resources among different frameworks. DRF(Dominant Resource Fairness)
allocator is the default resource allocator in Mesos and works quite well. In
simple, it calculate the dominant resource share of a framework and always
allocates offer to the one which has smallest DRF share.

For example: If the computing pool has resources: 100 cpu, 200 GB memory, 300 TB
disk. There are two frameworks running, FA and FB. At a time point, FA consumes
20 cpu, 50 GB memory, 10 TB disk while FB consumes 30 cpu, 20 GB memory, 10 TB
disk. Then we can get their resource share are:

{% highlight bash %}
Framework   cpu share   memory share   disk share   Dominant resource share
FA          20 / 100    50 / 200       10 / 300     memory (25%)
FB          30 / 100    20 / 200       10 / 300     cpu (30%)
{% endhighlight %}

If at the same time, both FA and FB request computing resources from Mesos DRF
allocator, mesos will first offer to FA because it has an smaller (25% &lt; 30%)
dominant resource share.

Whenever a framework finished its task, the resource attached to the task will
be return back to mesos computing pool and free to request by all frameworks.
And there is another kind of resource named reserved resource which only
reserved to a specified role. Every role can configured to has some reserved
resources to avoid hungry.

However, mesos doesn't support in-framework task preempt well. Thinking about
why a framework may needs preempt its job with a more important one? Maybe in
below situations.

- No reserved resource for it
- It consumes a lot of resources and is now has the biggest dominant resource
  share

So to preempt a job, it has to do:

- Kill the old job
- Schedule a new one with resources (hopefully)

However, as we know, once a framework finished its job, the resources will
return back to mesos computing pool, so generally there isn't any guarantee that
this framework will get resources again. Especially if it has the biggest
dominant resource share.

One may agree that a framework needs preempt its job but may argue if a
framework do bad thing in this way. Yes, it can if mesos provides a mechanism to
support in-framework task preempt. For example, a framework may always use this
way to keep resource. But from another point. Mesos doesn't do mandatory check
in many ways, do not check if the task request more than resources than it can
use, do not check if it use reserved resource first and so on. It basically
trust the user and the framework in some ways.

To support in-framework task preempt, we may re-use the reserved resource
design, in below way:

- A hook to preempt a task, this hook will kill the task and release attached
  resources not to the computing pool but to the reserved resource pool for
  the framework
- Framework request resource and schedule new task, just as before, but will
  always received reserved resource released by the previous task, cool?

However, there are some require conditions, the framework must has a role and
it is the only framework with that role.

The above are just my humble words maybe lead to a better mesos, there are no
such code to implement so far but I have plan to proposal these ideas to mesos
community and like to implement all of them, maybe within the coming months
because I do have a little time to contribute to upstream project.
