---
layout: post
title:  "Funtoo Server Maintenance Notes"
date:   2018-09-03 15:30:00 -4000
categories: mail server
---

Being a reference of useful commands for managing my funtoo instance

# Package Management

**System update procedure**

```
 $ sudo emerge -uavDN --with-bdeps=y @world
 $ sudo emerge -av --depclean
 $ sudo revdep-rebuild -v -- --ask
 $ sudo dispatch-conf
```


# Virtual Hosts

**Create a new virtual host**

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

