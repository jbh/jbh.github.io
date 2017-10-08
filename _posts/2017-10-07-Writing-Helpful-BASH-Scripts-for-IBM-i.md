---
layout: post
title: Writing Helpful BASH Scripts for IBM i
excerpt: Helpful tips for writing BASH scripts, as well as an example apachectl BASH script.
categories:
    - ibm i
    - bash
---

* TOC
{:toc}

I feel like BASH is sometimes an overlooked tool these days. Writing scripts is simple, and a script can create
efficient command line interfaces to achieve tasks that would normally take a lot longer. A great example of this is
starting, stopping, and restarting Apache on the IBM i. For those of us not accustomed to the green screen, there's the
HTTPAdmin Web GUI where the user can go to manage Apache servers. While HTTPAdmin is a nice tool, it can also be
cumbersome, and it adds time to development when the developer has to leave the command line environment and use a GUI
to restart a server.

In this article I'll be discussing how to write `apachectl` as a BASH script for the IBM i as an example of how
writing BASH scripts can greatly simplify tasks.

##### Picking a Location for Scripts

Scripts can be placed anywhere as long as the directory they're in is added to `PATH`. I suggest keeping custom scripts
inside the user's home directory and inside a `bin` folder. So, for example, the place I would store all my custom
scripts is `/home/josh/bin`.

To enable these scripts, add their bin to the PATH. In the `.bashrc` within the user's home directory on the IBM i, just
add something like this:

```bash
PATH=$PATH:~/bin
```

This appends the user's home folder bin to `PATH`. Now any scripts within that directory will be usable no matter what
directory the user is in.

##### Writing apachectl.sh

> Although one could just name the file `apachectl` and use it as such, I like to keep `.sh` on my custom user bin
scripts so it is easy to differentiate and doesn't override any other scripts.

Create the folder and file in whatever manner is easiest. The command line way:

```bash
$ mkdir /home/<user>/bin
$ touch /home/<user>/bin/apachectl.sh
```

The contents of my `apachectl.sh` currently contains:

```bash
#!/bin/bash

cmd=`basename "$0"`

print_use() {
  echo "Use: $cmd start|stop|restart <server>"
  echo "e.g.: $cmd start zendsvr6"
  exit 1
}

if [ $# -eq 2 ]
then
  action=$1
  server=${2^^}

  case "$action" in
    start)
      system "STRTCPSVR SERVER(*HTTP) HTTPSVR($server)"
      ;;
    stop)
      system "ENDTCPSVR SERVER(*HTTP) HTTPSVR($server)"
      ;;
    restart)
      system "ENDTCPSVR SERVER(*HTTP) HTTPSVR($server)"
      system "STRTCPSVR SERVER(*HTTP) HTTPSVR($server)"
      ;;
    *)
      print_use
  esac
else
  print_use
fi
```

##### Enabling and Using apachectl.sh

Sometimes it is necessary to enable a bash script. If there are issues running the script, try
`chmod +x /home/<user>/bin/apachectl.sh`.

If `PATH` has been properly set, and `apachectl.sh` is written and ready, then the user should be able to use the script
from anywhere while in BASH.

```bash
$ apachectl.sh stop zendsvr6
$ apachectl.sh start zendsvr6
$ apachectl.sh restart zendsvr6
```

Since it's a command in `PATH`, the user will have the convenience of tab completion as well when writing the command.

Here it is in action:

![apachectl.sh example](/images/writing-helpful-bash-scripts-for-ibm-i/apachectl-example.jpg)
