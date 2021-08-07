---
title: "New Laptop Review"
date: 2021-08-07T13:28:43-04:00
draft: false
---

# Dell XPS13 Dev Edition

I got [a new laptop](https://www.zdnet.com/article/dell-xps-13-linux-developer-edition-2020-hands-on-a-great-laptop-for-hard-working-developers/)!  It's a linux-default machine through Dell, the XPS13.  Her name is Friday, and she looks like this:

![Output of Neofetch](/img/xps13/Friday.png)

## TLDR

Mostly, I am very pleased with this laptop.  Dell has gone the extra mile to make things "just work" out of the box, and added extra safety with a recovery partition.  When plugged into a proper keyboard it's a high-powered computer with modern amenities running Ubuntu flawlessly.  However, the keyboard and touchpad can be really hard on my hands; I developed nerve tingling and sensitivity after only a few days of using the onboard peripherals, and need to use hand braces when using it directly for any period of time.  Part of this is my poor posture, but part of it is a trackpad that is very hard and keys with a very shallow range of motion.  I would recommend trying before you buy, or intending to hook up a real keyboard and mouse.  Note that the XPS 13 has **only two ports, both USB C**, so you really need something to help leverage them.  If you find a better solution here, such as something that passes through all the power, let me know!

**Rating**: 10/10 with peripherals, 5/10 without.

## Setup

For anyone else going down this road, or potentially me in the future, here are the setup tasks I performed to get to my happy place.

### Upgrading to 20.04

The laptop shipped with an earlier version of Ubuntu, so step 1 was fixing that.  As part of the process I had to add the `focal` repos.  In case you need them, here are the repositories I ended up with:

![Repos](/img/xps13/Repos.png)


### Connect my google account

Connecting Drive locally is especially nice, and this will be used for setting backups.

### Update software

`apt update` and `apt upgrade`.

### Restoration Media

Safety is important, and a couple minutes and some prep to get restoration media can save a lot of heartache in the long run. **You will need a USB C memory stick**.  I got [this one](https://www.amazon.com/dp/B01EZ0X034?psc=1&ref=ppx_yo2_dt_b_product_details) for $10.  Follow the system prompts and sleep easily; with restoration media and a solid backup system, the only real problems are hardware problems.

### Password Manager

I use [1password](https://1password.com/).

### Configure Backups

With Deja Dup and the Google Drive integration.  As for _what_ to back up, I specifically **include**:

* `~/Documents`
* `~/Pictures`
* `~/Music`
* `~/Videos`
* `~/Diagnostics`

And specifically **exclude** `~/Downloads`.  My basic theory is that backups should be fast and small, restricted to *data*.  If it's software I can reinstall, it doesn't belong in a backup.  So I *can* reinstall things, I have a cronjob that writes the installed packages to `~/Diagnostics`.  See [here](https://unix.stackexchange.com/questions/838/weekly-cron-job-to-save-list-of-installed-packages).   

### Enable restricted extras

[See here](https://askubuntu.com/questions/56446/how-do-i-install-the-ubuntu-restricted-extras-package).

### Livepatch

Set up in settings, this lets you do system updates while things are running.

### Night Mode

For screens that are easier on the eyes.

### Monitoring conveniences

I like bashtop (or bpytop) and neofetch (for the image above).

### Utilities

 * flatpak
 * git
 * vscode
 * vlc

### Thunderbird

I love the snappy shortcut-oriented workflow this provides.  I also use these extensions:

* Provider for Google Calendar
* Thunderbird Conversations

Which enable integration of my calendar and tasks.

### Launcher

[Ulauncher](https://ulauncher.io/) is just like the mac launcher; `ctrl+space` and name a thing to open it.

### Firefox and Mozilla VPN

For a more privacy-oriented browsing experience.  Moz VPN must be manually started and costs money, but I am happy to have another way to support Mozilla in developing a browser for *me*, not ad companies.

### Summary

That's the important stuff!  Now we just tweak aesthetics.

## Lifestyle Stuff

### Linux Dynamic Wallpapers

I just like the way this looks.  [Guide here.](https://www.omgubuntu.co.uk/2018/06/macos-mojave-dynamic-background-linux)

### Gnome Tweaks

Gnome is so customizable!  Use [a guide like this](https://itsfoss.com/gnome-shell-extensions/).

Here are the extensions I use:

![Tweaks](/img/xps13/Tweaks.png)

Also look at changing themes via a store like [this one](https://www.gnome-look.org/browse/).  I use:

![Theme](/img/xps13/Theme.png)


### Peripherals

As discussed above, the built in keyboard is **really** hard on my hands; an express bus to nerve pain in a debilitating place.  To compensate, I bought:

1. [This keyboard](https://www.amazon.com/dp/B07CHL1N46?psc=1&ref=ppx_yo2_dt_b_product_details)
2. [These](https://www.amazon.com/gp/product/B003BEECT2/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1) [braces](https://www.amazon.com/gp/product/B003BEECTC/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1).

The keyboard came well reviewed for range of motion, low noise, and compatibility with linux.  It has a fancy-color mode and a boring-lights mode, but the fancy-color mode is not customizable on linux as far as I can tell.  I have not experimented on other OSes yet.

Also, **the XPS13 only has two ports**.  This makes using all the peripherals I want tricky.  I bought [this USB hub](https://www.amazon.com/dp/B098RTGV47?psc=1&ref=ppx_yo2_dt_b_product_details) to make life a little better, but it has some caveats.  Plugging system power through the hub results in a warning screen about lower power than expected, so I want to plug power in directly.  This leaves a single USB C port for... everything else.  I am experimenting with how much I can plug through the hub; using a DVI<->USB C converter for my (old) monitor doesn't pass through correctly, so I need to try an HDMI converter.  

## Overall

I like this laptop **a lot**.  It's very powerful, looks great, and feels lovely as an extension of my brain.  The hardware interface could be easier on my hands, but you can't have everything.  Strong recommend!