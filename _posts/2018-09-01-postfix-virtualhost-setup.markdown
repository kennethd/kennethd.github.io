---
layout: post
title:  "Mail Server Setup"
date:   2018-09-01 14:30:00 -4000
categories: mail server
---

This mail server setup will provide a virtual host setup using Postfix for
SMTP, Courier IMAP, Postgres, Spamassassin, and Clamav, deployed to a
[Funtoo.org](https://www.funtoo.org/Funtoo_Containers) hosted instance,
and with a RoundCube WebMail UI.

This document is intended as a personal reference for myself about the setup of
my particular system, and not so much as a HOWTO for others.  I owe thanks to
many resources on the internet, but particularly [SwifT's HOWTO for
Gentoo](https://wiki.gentoo.org/wiki/User:SwifT/Wikified_but_not_merged_documents/Virtual_mail_HOWTO)

## Getting Started

My USE flags from `/etc/portage/make.conf`:

{% highlight ini %}
USE="-X -alsa -gnome -gtk -kde -mbox -qt4 -vda apache2 authdaemond bzip2 clamdtop crypt geoip imap ipv6 maildir mysql postgres sasl smtp spamassassin spell ssl urandom vhosts"
{% endhighlight %}

Installed versions of software:

{% highlight shell-session %}
kennethd ~ # equery list courier* postgres* postfix* spamassassin* clamav* roundcube* apache* 
 * Searching for courier* ...
[I-O] [  ] net-libs/courier-authlib-0.68.0-r1:0
[I-O] [  ] net-libs/courier-unicode-2.0:0
[I-O] [  ] net-mail/courier-imap-4.18.2:0

 * Searching for postgres* ...
[I-O] [  ] dev-db/postgresql-9.6.9:9.6
[I-O] [  ] dev-db/postgresql-10.4:10

 * Searching for postfix* ...
[IP-] [  ] mail-mta/postfix-3.2.4:0
[I-O] [  ] www-apps/postfixadmin-3.1:3.1

 * Searching for spamassassin* ...
[I-O] [  ] mail-filter/spamassassin-3.4.1-r21:0

 * Searching for clamav* ...
[I-O] [  ] app-antivirus/clamav-0.100.1:0

 * Searching for roundcube* ...
[I-O] [  ] mail-client/roundcube-1.3.7:1.3.7

 * Searching for apache* ...
[I-O] [  ] app-admin/apache-tools-2.4.34:0
[I-O] [  ] www-servers/apache-2.4.34-r2:2
{% endhighlight %}

## Postgres Configuration

You'll notice there are two versions of Postgres on the system.  I think the
9.6 one is from when I last began this setup, a few months ago, and 10.4 was
installed when emerging @world in preparation for this attempt.  Gentoo &
Funtoo allow for these parallel installations for upgrades and convenience of
maintenance.

Initial configuration will be read from `/etc/conf.d/postgresql-$VERSION`,
which defines a couple of important directories, `PGDATA` is where the postgres
config files will be found, and `DATA_DIR` where logs and database data will be
stored:

{% highlight ini %}
PGDATA="/etc/postgresql-10/"
DATA_DIR="/var/lib/postgresql/10/data"
{% endhighlight %}

Follow the instructions on the [Gentoo Postgres Quickstart](https://wiki.gentoo.org/wiki/PostgreSQL/QuickStart)
to set the superuser account password and add the postgres service to the default runlevel.

{% highlight shell-session %}
kennethd  # rc-update add postgresql-6.3 default
{% endhighlight %}

## Postfix Configuration

The only changes to `master.cf` were the uncommenting of the second and
services listed below (`smtp/inet` and `smtpd/pass`)

{% highlight text %}
# ==========================================================================
# service type  private unpriv  chroot  wakeup  maxproc command + args
#               (yes)   (yes)   (no)    (never) (100)
# ==========================================================================
smtp      unix  n       -       y       -       -       smtpd
smtp      inet  n       -       n       -       1       postscreen
smtpd     pass  -       -       n       -       -       smtpd
{% endhighlight %}

I've reduced my `main.cf` to include only options that vary from the default:

{% highlight ini %}
# default compatibility_level=0 logs messages about settings which have changed
# across versions. see http://www.postfix.org/COMPATIBILITY_README.html
compatibility_level = 2
# added due to presence of `maildir` USE flag, only used for system users
home_mailbox = .maildir/
# default is /usr/local/man, but that directory does not exist
manpage_directory = /usr/share/man

mydomain = localhost
myhostname = kennethd.host.funtoo.org
mynetworks_style = host
recipient_delimiter = +

smtp_tls_loglevel = 1
smtp_tls_security_level = may
smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_scache
smtpd_tls_cert_file = /etc/postfix/cert-20160808-193637.pem
smtpd_tls_key_file = /etc/postfix/key-20160808-193637.pem
smtpd_tls_loglevel = 1
smtpd_tls_received_header = yes
smtpd_tls_security_level = may
tls_random_source = dev:/dev/urandom

# user:group 5000 matches `vmail` user in /etc/passwd, and owns /var/vmail
# vmail:x:5000:5000::/var/vmail:/bin/false
virtual_alias_maps = pgsql:/etc/postfix/pgsql/virtual_alias_maps.cf
virtual_gid_maps = static:5000
virtual_mailbox_base = /var/vmail
virtual_mailbox_domains = pgsql:/etc/postfix/pgsql/virtual_mailbox_domains.cf
virtual_mailbox_maps = pgsql:/etc/postfix/pgsql/virtual_mailbox_maps.cf
virtual_uid_maps = static:5000

# http://www.postfix.org/POSTSCREEN_README.html
postscreen_dnsbl_action = enforce
postscreen_greet_action = enforce

{% endhighlight %}

# Create the postgres schema

The virtual host database is queried via the following SQL statements:

{% highlight ini %}
# virtual_alias_maps.cf
query           = SELECT goto FROM alias WHERE address='%s' AND active='1';

# virtual_mailbox_domains.cf
query           = SELECT description FROM domain WHERE domain = '%s' AND backupmx = '0' AND active = '1';

# virtual_mailbox_maps.cf
query           = SELECT maildir FROM mailbox WHERE local_part='%u' AND domain='%d' AND active='1';
{% endhighlight %}

# Alias root@

Edit `/etc/mail/aliases` to ensure root@ forwards to your email address.

{% highlight shell-session %}
kennethd ~ # newaliases 
kennethd ~ # /etc/init.d/postfix reload
{% endhighlight %}


# Requiring SSL

# Creating Certs

# DKIM

# SMTP-time SpamAssassin & ClamAV Configuration


## Courier Configuration



