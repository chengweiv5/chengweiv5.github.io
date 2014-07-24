---
layout: post
title: Fail to log into docker container through ssh if SELinux enabled
tags:
  - selinux
  - docker
  - ssh
---

[Docker](https://docker.com/) now is standing at the core part of cloud
computing, and is deemed to be the next big technique of cloud computing, it's
changing the game.

When I recently start to experience docker and also try to use it as a
light-weight virtual machine though this isn't the recommended way to using
docker.
[Why?](http://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/).

I start my way to setup a docker image as below:

- With CentOS 6.4 as docker host OS
- With Docker version 1.0.0, build 63fe64c/1.0.0
- CentOS 6.4 as docker base image
- With [supervisor](https://docs.docker.com/articles/using_supervisord/)
  configured
- With [sshd](https://docs.docker.com/examples/running_ssh_service/) configured

This image has supervisor as its entry process which monitored by docker, and
supervisor will start sshd at container startup. So if everything goes right, I
can login to the container through ssh.

First, I start a container like below

{% highlight bash %}
# docker run -p 12345:22 docker-sshd
2014-07-24 23:18:43,721 CRIT Supervisor running as root (no user in config file)
Unlinking stale socket /var/tmp/supervisor.sock
2014-07-24 23:18:44,166 INFO /var/tmp/supervisor.sock:Medusa (V1.1.1.1) started
at Thu Jul 24 23:18:44 2014
        Hostname: <unix domain socket>
        Port:/var/tmp/supervisor.sock
2014-07-24 23:18:44,302 CRIT Running without any HTTP authentication checking
2014-07-24 23:18:44,304 INFO supervisord started with pid 1
2014-07-24 23:18:44,314 INFO spawned: 'sshd' with pid 6
2014-07-24 23:18:45,325 INFO success: sshd entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
{% endhighlight %}

As expected, I can login through ssh by connecting localhost:12345, as I can get
this port mapping by *docker ps* or *docker port*.

{% highlight bash %}
# docker ps
CONTAINER ID        IMAGE              COMMAND                CREATED             STATUS              PORTS                  NAMES
0cfbfffd018d        docker-sshd:1.2    /usr/bin/supervisord   2 minutes ago       Up 2 minutes        0.0.0.0:12345->22/tcp  jolly_pike

# docker port 0cfbfffd018d 22
0.0.0.0:12345
{% endhighlight %}

However, when I try to login from ssh, it failed like below.

{% highlight bash %}
# ssh localhost -p 12345
Connection to localhost closed by remote host.
{% endhighlight %}

So I restart a container with /bin/bash as its entry process to figure out what
caused this error.

{% highlight bash %}
# docker run -it -p 12345:22 docker-sshd /bin/bash
{% endhighlight %}

And then start sshd manually.

{% highlight bash %}
bash-4.1# /usr/sbin/sshd -De
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.
{% endhighlight %}

From another console, login again and I found below output from sshd in docker
container.

{% highlight bash %}
Accepted publickey for root from 172.17.42.1 port 19548 ssh2
ssh_selinux_getctxbyname: Failed to get default SELinux security context for root
ssh_selinux_setup_exec_context: security_getenforce() failed
{% endhighlight %}

At the first glance, it's related by SELinux, and I checked the docker host
SELinux status like below.

{% highlight bash %}
# sestatus 
SELinux status:                 enabled
SELinuxfs mount:                /selinux
Current mode:                   permissive
Mode from config file:          permissive
Policy version:                 24
Policy from config file:        targeted
{% endhighlight %}

Yes, it's enabled and in permissive state. So how about disable SELinux? The
answer is *yes*, I can login to docker container through ssh with SELinux
disabled on docker host.

So this is a bug about Docker and SELinux integration? Not really. There are a
lot of words says SELinux support for Docker is in the plan, so we can expected
it doesn't work in short coming.

As I dive into a little deep.

- why ssh login try to get user's SELinux context?
- why it doesn't figure out it's in docker container, has no SELinux runtime
	filesystem mounted, no SELinux context for files?

As I follow this clue and read some code of the specific libselinux version
installed in my docker image. That is libselinux-2.0.94-5.3.el6.x86_64. I
found below code is doing the runtime check of SELinux status.

{% highlight c %}
src/enabled.c

int is_selinux_enabled(void)
{
	char *buf=NULL;
	FILE *fp;
	ssize_t num;
	size_t len;
	int enabled = 0;
	security_context_t con;

	/* init_selinuxmnt() gets called before this function. We
	 * will assume that if a selinux file system is mounted, then
	 * selinux is enabled. */
	if (selinux_mnt) {

		/* Since a file system is mounted, we consider selinux
		 * enabled. If getcon_raw fails, selinux is still enabled.
		 * We only consider it disabled if no policy is loaded. */
		enabled = 1;
		if (getcon_raw(&con) == 0) {
			if (!strcmp(con, "kernel"))
				enabled = 0;
			freecon(con);
		}
		return enabled;
  }

	/* Drop back to detecting it the long way. */
	fp = fopen("/proc/filesystems", "r");
	if (!fp) 
		return -1;

	__fsetlocking(fp, FSETLOCKING_BYCALLER);
	while ((num = getline(&buf, &len, fp)) != -1) {
		if (strstr(buf, "selinuxfs")) {
			enabled = 1;
			break;
		}
	}

	if (num < 0)
		goto out;

	/* Since an selinux file system is available, we consider
	 * selinux enabled. If getcon_raw fails, selinux is still
	 * enabled. We only consider it disabled if no policy is loaded. */
	if (getcon_raw(&con) == 0) {
		if (!strcmp(con, "kernel"))
			enabled = 0;
		freecon(con);
	}

out:
	free(buf);
	fclose(fp);
	return enabled;
}
{% endhighlight %}

The aove code does loosely check, although neither /sys/fs/selinux nor /selinux
mounted, it fall through to only check the kernel part, since Docker is reuse
the host kernel, so it always get SELinux enabled. However, the whole docker
image filesystem has no SELinux context and no SELinux context rules loaded, so
a wrong check will cause many problems, like the above one.

The good news is this is fixed in the new libselinux, e.g. version 2.2.2, it
does below strict check.

{% highlight c %}
src/enabled.c

int is_selinux_enabled(void)
{
        int enabled = 0;
        security_context_t con;

        /* init_selinuxmnt() gets called before this function. We
         * will assume that if a selinux file system is mounted, then
         * selinux is enabled. */
        if (selinux_mnt) {

                /* Since a file system is mounted, we consider selinux
                 * enabled. If getcon_raw fails, selinux is still enabled.
                 * We only consider it disabled if no policy is loaded. */
                enabled = 1;
                if (getcon_raw(&con) == 0) {
                        if (!strcmp(con, "kernel"))
                                enabled = 0;
                        freecon(con);
                }
        }

        return enabled;
}
{% endhighlight %}

So I can deploy this version of selinux and its dependencies to my docker image
and get ssh works fine with SELinux enabled on docker host. Though it's a little
complicated compare with just disable SELinux on docker host.
