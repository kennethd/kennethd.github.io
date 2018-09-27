---
layout: post
title:  "Git Server Setup"
date:   2018-09-18 13:30:00 -4000
categories: git server
---

These are my notes regarding setting up
[GitWeb](https://git-scm.com/book/en/v2/Git-on-the-Server-GitWeb) and 
[gitolite](http://gitolite.com/gitolite/index.html) to host public and private
git repos on my [Funtoo.org](https://www.funtoo.org/Funtoo_Containers) hosted
instance.

My requirements for a git server include:

  * Both public and private repositories
  * User groups, and whitelist access to repos based on them
  * Restrict write access to master
  * Restrict write access to other branches to owner/group

I am migrating from an older system which used
[gitosis](https://git-scm.com/book/en/v1/Git-on-the-Server-Gitosis) for the
same purpose.  `gitolite` has for the most part superceded `gitosis`, and is
the only one of the two available in portage.

## Install Gitolite

{% highlight shell-session %}
kenneth@kennethd:~$ sudo emerge dev-vcs/gitolite
{% endhighlight %}

#### Copy bare repositories into place

Having already used `rsync` to copy the **repositories** directory from the
old gitosis server to the new gitolite user, move it to the **git** user's
`$HOME` directory & verify ownership

Note this bit is being run as `root`:
{% highlight shell-session %}
kennethd ~ # rsync -r ~kenneth/tmp/repositories ~git/
kennethd ~ # chown -R git:git ~git/repositories 
kennethd ~ # ls -l ~git
total 12
drwxr-xr-x 75 git git 76 Sep 18 21:13 repositories
{% endhighlight %}

[Gitolite's docs](http://gitolite.com/gitolite/basic-admin/#appendix-1-bringing-existing-repos-into-gitolite) warn:

***Warning!***

***Gitolite will clobber any existing update hook in your repos when you do this.
Please see either the cookbook or the non-core page for information on how to
make your existing update hook work with gitolite.***

***Gitolite may clobber any existing "git-daemon-export-ok" file in your repo;
see the page on allowing access to gitweb and git-daemon for how to enable
that via gitolite.***

#### Run setup

{% highlight shell_session %}
kenneth@kennethd:~$ sudo cp ./.ssh/id_rsa.pub ~git/kenneth.pub
kenneth@kennethd:~$ sudo su - git
git@kennethd ~ $ mkdir -p .gitolite/logs
git@kennethd ~ $ gitolite setup -pk kenneth.pub
{% endhighlight %}

Setup will create bare repository `~git/repositories/testing.git` and add it
to `~git/projects.list`

## Daemonize git-daemon

`git-daemon` is used to provide anonymous read access to public archives.

There are a lot of options available for creating a service for `git-daemon`, a partial list:

  * **[supervisord](http://supervisord.org/)** which is what I plan to use on this server, and will set up below
  * **[runit](http://smarden.org/runit/)** my old gitosis server's runit config is documented [here](http://wiki.ylayali.net/doku.php?id=git:gitosis#anonymous_access)
  * **[sysvinit](https://packages.debian.org/sid/git-daemon-sysvinit)** -- link goes to debian's packaged config
  * **[systemd](https://git-scm.com/book/en/v2/Git-on-the-Server-Git-Daemon)** config from official git docs
  * Ubuntu's **[upstart](https://git-scm.com/book/en/v2/Git-on-the-Server-Git-Daemon)** config from official git docs
  * **[xinetd]()** there's an old HOWTO [here](https://cryptkcoding.com/blog/2011/09/04/setting-up-a-git-server-with-xinetd-gitolite-and-cgit-the-right-way/)

#### Install supervisord

{% highlight shell_session %}
kenneth@kennethd:~$ sudo emerge app-admin/supervisor
{% endhighlight %}

If you want your regular user to be able to run `supervisorctl`, it needs to be added to the group `supervisor`:
{% highlight shell_session %}
kenneth@kennethd:~$ sudo gpasswd -a kenneth supervisor
{% endhighlight %}

`sudo su - $LOGNAME` to obtain a shell with the newly added group activated

#### Create gitdaemon user for read-only access to the repositories

This user will own the `git-daemon` process
{% highlight shell_session %}
kenneth@kennethd:~$ sudo useradd --system --no-user-group --home-dir /nonexistent --no-create-home --shell /bin/false gitdaemon 
kenneth@kennethd:~$ grep git /etc/passwd
git:x:105:103:added by portage for gitolite:/var/lib/gitolite:/bin/sh
gitdaemon:x:999:100::/nonexistent:/bin/false
{% endhighlight %}

#### Create git-daemon supervisor config




## Configure repositories



