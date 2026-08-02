---
layout: post
title:  "bunsenlabs tryout with live dvd/usb"
date:   2026-08-02
categories: linux
---

  * Table of Contents
  {:toc}


# bunsenlabs tryout with live dvd/usb

This doc is a step-by-step guide to using a bootable linux distro to try it
out, just to see if you like it (without installing anything on your computer).

This is not an installation guide (maybe an addendum will come later), which
would definitely would change your computer permanently.

This guide was created for my mom (who is totally unfamiliar with linux), but
I will try to make the instructions generally useful for anyone who wants to
squeeze some more life out of an older laptop.

## before you start

A few things you might need:

  * The name & password for your wireless access point, if using WLAN
  * Zoom login credentials, if installing Zoom

## why bunsenlabs?

There are a bunch of listicles out there, if you search "best distro for old
laptops", "user-friendly distro for users transitioning from windows", or
similar, bunsenlabs has a pretty great reputation among those kinds of lists,
so it seemed a good first choice to try out.

Personally, I still have a 12? 14? year old thinkpad x1 (gen2) with only 4 gb
ram, and know from experience it can't handle a gnome desktop, but it works
fine for my needs with fluxbox.

Still, debian + fluxbox requires a lot of hands-on setup & maintenance that I
don't want to subject my mom to, ideally I would like a distro that uses a
lightweight window manager (bunsenlabs uses openbox, not fluxbox, which does
seem to need a little less up-front menu maintenance and so on), but which
also provides winsows-like ui components.

bunsenlabs looks promising!

## prepare the install media

(mom, skip this, i already sent you the dvd)

 1. head over to https://www.bunsenlabs.org/installation.html & download the iso (also checksum, signature if you like)
 2. burn to the desired installation/live media
    * i happened to be burning a dvd-r from windows 11, right clicking on the iso in windows explorer brings up a "burn to disc" menu option (there was no "create a bootable disc" toggle like in days of yore, but it booted fine)
    * when creating a bootable usb stick, i like https://rufus.ie/en/

At this point, you can try it out, insert the dvd into the drive or the usb
stick into the port, and restart your computer.  unless you've done this
before, it is unlikely to work.  if it just boots you straight back into
windows, continue to the next section.

## restart into your bios

Most pc hardware ships configured to boot directly into windows, we want to
adjust it so that if a bootable dvd/usb is found on the system, it will choose
to boot into that instead.

The following screenshots were created on a thinkpad x1, your bios will look a
bit different, but in general, the process of looking for an option like "boot
order", modifying it, saving & restarting is the same across every bios i've
ever seen.

To access the bios on a thinkpad, you press the F1 key during restart, on my
old Dell I think it was F12, different manufacturers require different
keypresses to get into the bios during boot, you can refer to the article at:

https://www.tomshardware.com/reviews/bios-keys-to-access-your-firmware,5732.html

or your manufacturer's website, for example:

https://www.dell.com/support/kbdoc/en-us/000126121/access-uefi-bios-system-setup-from-windows-on-your-dell-system

### adjusting the boot order in bios

Once you boot into your bios there will be some kind of menu screen, as
mentioned, the high-level description of our objective is to find an option to
edit the boot order on startup, so let's start by entering the **Startup** submenu:<br />
![main screen from thinkpad bios](/assets/bunsenlabs/001-bios-main.png) <br />
(apologies for some of these cellphone-style screenshots, i did this in a bit of a hurry)

This example screen looks like it has an option to choose the boot device
interactively at boot time via the F12 key -- maybe a lot of modern BIOSes have
this option (it's been at least a decade since I regularly set up dual-boot
systems & such) -- still, I am targeting old hardware here, and let's assume we
have to do it the old-fashioned way.

At the very top of the **Startup** menu, I've circled a **Boot** option, it
might say "boot order", "boot devices", somehitng like that:<br />
![startup submenu from thinkpad bios](/assets/bunsenlabs/002-bios-startup.png) <br />

Finally the **Edit Boot Order** screen.  The keys/mechanisms to change the
order will not be the same for all BIOSes, but there should be a menu spelling
out the options at the bottom of the screen, for me I had to use the arrow
keys to highlight the entry I wanted to modify, and then use F5/F6 to move
that item up or down the list.

This laptop already dual-boots into Linux, so you probably won't have anything
like my "debian" option, and your USB/DVD drives will probably have different
names, but I recognize "USB CD TSSTcorp CDDVDW SE-etc.." as my external CD/DVD
drive, and "USB HDD" is clearly how a removeable USB stick would show up, so
we want to move those above "Windows Boot Manager": <br />
![boot order sub-sub-menu from thinkpad bios](/assets/bunsenlabs/003-bios-boot-order.png) <br />

Once the order looks good, we can save & reboot.

Make sure the bunsenlabs DVD is in the drive, or USB stick is in the port before continuing.

Again, there should be a menu at the bottom of the screen.  For me I think it
was F10 & the option was called "Save & Quit" (or "Save & Restart", or something like that).

If you are unsure that you didn't mess something up, it's probably better to
cancel & restart, and maybe ask for help (I intend to be on the phone with Mom
when she does this bit, for sure).  If you feel confident, go ahead and *Save & Restart*.

*Note, you don't have to change the boot order every time you want to use the
live-media disc/usbstick, only the first time*

## reboot into bunsenlabs

Particularly if you are booting from an external DVD drive like I am, the
restart may take awhile, your drive may make a lot of noise and spin like
crazy, give it time, it will hopefully get there.

It's a good time to note, the point of this exercise is very much just trying
out linux to decide if it's something you can live/work with, from a useabilty
perspective.  The performance while running from a live DVD or USB stick is
going to sometimes be a struggle, and you should not use this experience to
judge Linux is too slow or anything like that, I assure you this particular
version of linux will be faster than any version of Windows if installed to
disk.

What happens when running a live-from-media OS is: the OS (Linux), loads into
memory, and has to swap things in-and-out of memory, reading from very slow
media when it needs to read something.  Usually once it is up and running, you
can configure the internet connection, and use it almost like normal for
periods of time, but you will definitely notice a lot of disc-gnashing the
first time you open a web browser, or other major applications.  As an
experiment, we'll go ahead and install Zoom below, which will also install
directly into memmory.

## bunsenlabs welcome screen
Once the reboot-from-bios finally loads, and most of that disc-gnashing
settles down, you should be presented with a welcome screen, and a few options: <br />
![bunsenlabs Live System option on welcome screen](/assets/bunsenlabs/004-bunsenlabs-choose-live-system.png) <br />

**DO NOT CHOOSE "Start Installer"**


