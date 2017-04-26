---
layout: post
title: CentOs 7 + i3wm Quickstart Guide
excerpt: |
    A simple, few-step guide to getting started with i3wm on CentOS 7 starting with installing CentOS 7.
categories:
    - linux
    - centos
    - i3wm
---

* TOC
{:toc}

> Most commands will need to be ran with super user permissions. Just prepend with `sudo` if any permissions errors
occur.

### Download CentOS 7

[Download](https://www.centos.org/download/){:target="_blank"} the preferred version of CentOS. I suggest the
"Everything" version for this tutorial.

### Install CentOS 7

Begin the installation process like any other Linux installation. Either through
[USB](https://wiki.centos.org/HowTos/InstallFromUSBkey){:target="_blank"} or
[Virtual Machine](https://wiki.centos.org/HowTos/Virtualization/VirtualBox){:target="_blank"}.

Installation is rather simple. I suggest the following configuration when reaching the `Software Selection` screen:

```
Basic Web Server > Backup Client
                   Debugging Tools
                   Directory Client
                   Language Support
                   Security Tools
                   Any preferred languages
```

![Cent OS Software Selction](/images/cent-os-software-selection.png)

> If one prefers not to have i3wm and would like to just use CentOS 7 with either KDE or Gnome from this point, they
should feel free to select `Development and Creative Workstation` instead of `Basic Web Server` in the first pane and
choose their preferred GUI/Window Manager now. Otherwise, continue on for a simple i3wm quickstart guide.

After choosing software, continue the installation as normal. Set a root password, setup a new administrator user, and
finish. Once rebooted and logged in as the new user, we can begin setting things up and installing i3wm.

### Setup network

This is necessary to communicated with the outside world, which we need to do in order to install software packages.
Follow instructions at [krizna](http://www.krizna.com/centos/setup-network-centos-7/){:target="_blank"}. The first
three steps under `GUI Mode` are all that are necessary.

### Add Required Repositories

Run these commands in order to install the two extra package repositories we need.

[EPEL 7](https://fedoraproject.org/wiki/EPEL){:target="_blank"} Repository:

```bash
$ su -c 'rpm -Uvh http://download.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm'
```

[coprs](https://copr.fedorainfracloud.org/coprs/admiralnemo/i3wm-el7/){:target="_blank"} Repository:

```bash
$ yum install -y dnf dnf-plugins-core
$ sudo dnf config-manager --add-repo https://copr.fedorainfracloud.org/coprs/admiralnemo/i3wm-el7/repo/epel-7/admiralnemo-i3wm-el7-epel-7.repo
```

### Install Xorg

Use `groupinstall` to ensure we get everything we need.

```bash
$ yum groupinstall "X Window System" "Desktop" "Desktop Platform"
```

### Install lightdm

`lightdm` is a lightweight package that will allow for a login screen

```bash
$ yum install lightdm xorg-x11-xinit-session
```

### Install i3wm

Install i3wm along with `i3status` and `lilyterm` just to get us started.
[LilyTerm](http://lilyterm.luna.com.tw/){:target="_blank"} is a lightweight terminal emulator. i3 does not come
with one.

```bash
$ yum install dejavu-sans-fonts dejavu-sans-mono-fonts dejavu-serif-fonts i3 i3status lilyterm
```

### Start GUI manually

Start the GUI manually from command line using the following command:

```bash
$ systemctl isolate graphical.target
```

You should be greeted with a login screen.

![CentOS Login Screen](/images/cent-os-login-screen.png)

You may need to manually select `i3` from the settings menu in the top right before logging in.

![CentOS i3 Selection](/images/cent-os-i3.png)

Once logged in, i3 should be ready to go!

### Start GUI automatically

All that needs to happen in order for the GUI to start automatically at startup is for this command to be ran:

```bash
$ systemctl set-default graphical.target
```

### Optional packages for customizing i3wm

For more customization, I suggest installing the following packages.

[py3status](https://py3status.readthedocs.io/en/3.5/intro.html){:target="_blank"} is used for customizing the status bar further.

```bash
$ yum install py3status
```

[feh](https://faq.i3wm.org/question/6/how-can-i-set-a-desktop-background-image-in-i3.1.html){:target="_blank"} is used to customize
the background for i3wm.

```bash
$ yum install feh
```
