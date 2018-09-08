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

Installed versions of some relevant software:

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


I will use `ylayali.net` as the authoritative (or "non-virtual") domain for
the server, to allow mailing system users (and so my personal account can
continue to use procmail).  It needs to be added to `/etc/hosts` on its own line:

```
# Auto-generated hostname. Please do not remove this comment.
127.0.0.1       kennethd.host.funtoo.org kennethd localhost localhost.localdomain
::1             kennethd.host.funtoo.org kennethd localhost localhost.localdomain
172.97.103.107  ylayali.net mail.ylayali.net www.ylayali.net media.ylayali.net rt.ylayali.net
```

The prototype domain for testing virtual host features will be `highball.org`.


## Create vmail User

{% highlight shell-session %}
kennethd ~ # groupadd -g 5000 vmail
kennethd ~ # useradd -m -d /var/vmail -s /bin/false -u 5000 -g vmail vmail
kennethd ~ # ls -ld /var/vmail/
drwxr-xr-x 3 vmail vmail 4096 Nov 11 22:39 /var/vmail/
kennethd ~ # chown vmail:vmail /var/vmail/
kennethd ~ # chmod 2770 /var/vmail/
kennethd ~ # ls -ld /var/vmail
drwxrws--- 3 vmail vmail 4096 Nov 11 22:39 /var/vmail
{% endhighlight %}


## Let's Encrypt!

The [Let's Encrypt!](https://letsencrypt.org/getting-started/)
SSL Certs will be used for Apache & Postfix.

{% highlight shell-session %}
kennethd ~ # emerge --ask app-crypt/certbot
{% endhighlight %}


[The Gentoo Wiki](https://wiki.gentoo.org/wiki/Let%27s_Encrypt) mentions an
optional package, [acme-tiny](https://github.com/diafygi/acme-tiny/), which
promises to bypass some of the "bloat" of the official client.  It requires
setting up an [overlay](https://www.funtoo.org/Local_Overlay), which are
managed via [layman](https://wiki.gentoo.org/wiki/Layman).


## Postgres Configuration

There are two versions of Postgres on the system, 9.6 and 10.  I think the
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

Following along with the instructions on the [Gentoo Postgres Quickstart](https://wiki.gentoo.org/wiki/PostgreSQL/QuickStart):

{% highlight shell-session %}
kennethd ~ # emerge --config dev-db/postgresql:10
{% endhighlight %}

**/etc/postgresql-10/pg_hba.conf**
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     password
# IPv4 local connections:
host    all             all             127.0.0.1/32            password
# IPv6 local connections:
host    all             all             ::1/128                 password
# Allow replication connections from localhost, by a user with the replication privilege.
local   replication     all                                     trust
host    replication     all             127.0.0.1/32            trust
host    replication     all             ::1/128                 trust
```

{% highlight shell-session %}
kennethd ~ # /etc/init.d/postgresql-10 reload
kennethd ~ # rc-update add postgresql-10 default
{% endhighlight %}

Try it out: `psql -U postgres postgres`
{% highlight sql %}
Password for user postgres:
psql (10.4)
Type "help" for help.

postgres=# CREATE ROLE kenneth WITH LOGIN;
CREATE ROLE
postgres=# \password kenneth
Enter new password:
Enter it again:
postgres=# CREATE DATABASE testdb WITH OWNER kenneth;
CREATE DATABASE
postgres=# \l
                                List of databases
   Name    |  Owner   | Encoding | Collate |    Ctype    |   Access privileges   
-----------+----------+----------+---------+-------------+-----------------------
 postgres  | postgres | UTF8     | C       | en_US.UTF-8 | 
 template0 | postgres | UTF8     | C       | en_US.UTF-8 | =c/postgres          +
           |          |          |         |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | C       | en_US.UTF-8 | =c/postgres          +
           |          |          |         |             | postgres=CTc/postgres
 testdb    | kenneth  | UTF8     | C       | en_US.UTF-8 | 
(4 rows)
{% endhighlight %}


## Postfix Configuration

I have the following package-specific USE flags:
{% highlight shell-session %}
kennethd ~ # echo "mail-mta/postfix cdb hardened postgres sasl ssl" >>  /etc/portage/package.use/mail-mta
{% endhighlight %}

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

mydomain = ylayali.net
myhostname = mail.ylayali.net
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

Create the `postfix` user:
{% highlight shell-session %}
kennethd ~ # createuser -U postgres -D -P -R -S postfix
{% endhighlight %}


# Alias root@

Edit `/etc/mail/aliases` to ensure root@ forwards to your email address.

{% highlight shell-session %}
kennethd ~ # newaliases 
kennethd ~ # /etc/init.d/postfix reload
{% endhighlight %}


# SPF and DKIM

[SPF (Sender Policy Framework)](https://en.wikipedia.org/wiki/Sender_Policy_Framework)
is a protocol intended to prevent email spoofing by publising (via DNS) a list of IPs
authorized to originate email for a domain.


# SMTP-time SpamAssassin & ClamAV Configuration


## Courier-imap Configuration

Edit both `/etc/courier-imap/{imapd.cnf,pop3d.cnf}` to look similar to:

{% highlight ini %}
[ req_dn ]
C=US
ST=NY
L=New York
O=Courier Mail Server
OU=Automatically-generated IMAP SSL key
CN=localhost
emailAddress=postmaster@ylayali.net
{% endhighlight %}

And run:

{% highlight shell-session %}
kennethd ~ # cd /etc/courier-imap/
kennethd /etc/courier-imap # mkpop3dcert 
kennethd /etc/courier-imap # mkimapdcert 
kennethd /etc/courier-imap # /etc/init.d/courier-imapd start 
kennethd /etc/courier-imap # /etc/init.d/courier-imapd-ssl start 
kennethd /etc/courier-imap # /etc/init.d/courier-pop3d start 
kennethd /etc/courier-imap # /etc/init.d/courier-pop3d-ssl start 
{% endhighlight %}

# Cyrus-sasl Configuration

Cyrus-sasl accepts login info for Courier-imap, which Courier then uses to
authenticate against postgres

**/etc/sasl2/smtpd.conf**
```
mech_list: PLAIN LOGIN
pwcheck_method: saslauthd
```

**/etc/conf.d/saslauthd**
```
SASLAUTHD_OPTS="${SASLAUTH_MECH} -a rimap -r" SASLAUTHD_OPTS="${SASLAUTHD_OPTS} -O localhost"
```

And run:
{% highlight shell-session %}
kennethd /etc/courier-imap # /etc/init.d/saslauthd start
{% endhighlight %}

# Add Courier-imap services to default runlevel

{% highlight shell-session %}
kennethd /etc/courier-imap # rc-update add courier-authlib default
kennethd /etc/courier-imap # rc-update add courier-imapd default
kennethd /etc/courier-imap # rc-update add courier-pop3d default
kennethd /etc/courier-imap # rc-update add courier-imapd-ssl default
kennethd /etc/courier-imap # rc-update add courier-pop3d-ssl default
kennethd /etc/courier-imap # rc-update add saslauthd default
{% endhighlight %}


## Apache Configuration

Verify the value of `APACHE2_OPTS` in `/etc/conf.d/apache2`
{% highlight ini %}
APACHE2_OPTS="-D DEFAULT_VHOST -D INFO -D SSL -D SSL_DEFAULT_VHOST -D LANGUAGE -D SECURITY -D PHP"
{% endhighlight %}

```
kennethd ~ # /etc/init.d/apache2 start
kennethd ~ # rc-update add apache2 default
```



