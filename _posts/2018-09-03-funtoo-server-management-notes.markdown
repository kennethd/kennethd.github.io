---
layout: post
title:  "Funtoo Server Maintenance Notes"
date:   2018-09-03 15:30:00 -4000
categories: mail server
---

Being a reference of useful commands for managing my funtoo instance



# System update procedure
{% highlight shell-session %}
 $ sudo emerge -uavDN --with-bdeps=y @world
 $ sudo emerge -av --depclean
 $ sudo revdep-rebuild -v -- --ask
 $ sudo dispatch-conf
{% endhighlight %}


# Package Management

## equery

https://wiki.gentoo.org/wiki/Equery

### view USE flags active (and inactive) for a specific package
{% highlight shell-session %}
kennethd /etc/portage # equery uses  dev-db/postgresql:9.6
{% endhighlight %}



## eix 

### list all slots/versions available for a given package
{% highlight shell-session %}
kennethd /etc/portage # eix -e python
[?] dev-lang/python
     Available versions:  
     (2.7)  2.7.10-r1 (~)2.7.11-r2 (~)2.7.12
     (3.3)  3.3.5-r3 (~)3.3.5-r8(3.3/3.3m)
     (3.4)  3.4.3-r1 (~)3.4.3-r7(3.4/3.4m) (~)3.4.4(3.4/3.4m) (~)3.4.5(3.4/3.4m)
     (3.5)  (~)3.5.0-r2 (~)3.5.1-r2(3.5/3.5m) (~)3.5.1-r3(3.5/3.5m) (~)3.5.2(3.5/3.5m)
       {-berkdb build doc examples gdbm hardened ipv6 libressl +ncurses +readline sqlite +ssl +threads tk +wide-unicode wininst +xml ELIBC="uclibc"}
     Installed versions:  2.7.15(2.7)[?](02:52:09 PM 11/24/2018)(gdbm ipv6 ncurses readline sqlite ssl threads wide-unicode xml -berkdb -bluetooth -build -doc -examples -hardened -libressl -tk -wininst ELIBC="-uclibc")
                          3.4.6-r1(3.4/3.4m)[?](06:20:56 AM 09/01/2018)(gdbm ipv6 ncurses readline sqlite ssl threads xml -build -examples -hardened -libressl -tk -wininst ELIBC="-uclibc")
                          3.5.3-r1(3.5/3.5m)[?](06:42:02 AM 09/01/2018)(gdbm ipv6 ncurses readline sqlite ssl threads xml -build -examples -hardened -libressl -tk -wininst ELIBC="-uclibc")
     Homepage:            http://www.python.org/
     Description:         An interpreted, interactive, object-oriented programming language
{% endhighlight %}




# Virtual Hosts

## Create a new virtual host

  * create vmail records in postgres for domain
  * create apache vhost configs
  * create let's encrypt keys & config
  * create DNS A & MX records
  * create SPF, DKIM, & DMARC record


# Let's Encrypt!



# Postfix

The domain to address system users is `highball.org`

**SMTP Session**

Some problems are easiest diagnosed by communicating directly with postfix

```
telnet mail.highball.org 25
HELO local.ylayali.net
MAIL FROM: kenneth@ylayali.net
RCPT TO: kenneth@highball.org

Subject: Test from telnet

Test

.
QUIT
```

**Getting help on #postfix @ irc.freenode.org**

```
20:44 < kennethd> !getting_help
20:44 < knoba> kennethd: "getting_help" : before asking your question, read the !relevant_logs and !showconfig factoids, and prepare a single pastebin containing all of 
               that data. if you don't understand what this means, or if you need help doing this, please let us know. also see !pastebin
20:44 < kennethd> !relevant_logs
20:44 < knoba> kennethd: "relevant_logs" : mail.* syslog Postfix log messages (NOT verbose, see !no_verbose) which show ONLY the entire handling of a single mail which 
               illustrates the issue with which you want help. Random selections from your mail log are not adequate. IMAP/POP3 daemons and external delivery agents often 
               log to the same syslog facility and should not be shown. Also see http://rob0.nodns4.us/postfix-logging
20:44 < kennethd> !showconfig
20:44 < knoba> kennethd: "showconfig" : when asked to provide your config, please provide a SINGLE pastebin (see !pastebin) with postconf -nf and postconf -Mf. if your 
               version is too old for those commands to work (< 2.9), you should upgrade, but see !showconfig_old
```

# Postgres




# Maintaining these notes with Jekyll

**Clone**

```
 $ git clone gitosis@git.ylayali.net:kennethd.github.io.git
```

**Configure**

```

```

**Bundle**

```
 $ bundle install --verbose --path .bundle
```

**Serve**

```
 $ bundle exec jekyll serve
```

**Publish**

```
```

