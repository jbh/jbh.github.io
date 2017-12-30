---
layout: post
title: BASH on IBM i
excerpt: Creating a welcoming environment on the IBM i for Linux geeks going through BASH withdrawals.
categories:
    - ibm i
    - bash
    - git
---

* TOC
{:toc}

> **10/07/2017** As of the release of 5733-OPS, this is outdated. Please refer to
[IBM i Open Source Tools](/IBM-i-Open-Source-Tools) for more information.

> **Disclamer**: I am not an IBM i System Administrator. I am a Linux geek that wanted a more familiar development environment on the IBM i. This article is an attempt to document my experiences with getting a familiar BASH environment going on the IBM i.
> *I have only tested these techniques on versions [7.1](http://www-01.ibm.com/support/knowledgecenter/ssw_ibm_i_71/rzahg/icmain.htm "Documentation for IBM i v7r1"){:target="_blank"} and [7.2](http://www-01.ibm.com/support/knowledgecenter/ssw_ibm_i_72/rzahg/ic-homepage.htm "Documentation for IBM i v7r2"){:target="_blank"} of IBM i.*

This article will cover getting BASH running smoothly on the IBM i, as well as some tips and tricks of how to take advantage of this to build a comfortable BASH environment.

<!-- #### Assumptions

To keep this article relatively slim, I will make the following assumptions:

* The reader has a basic knowledge of [SSH](https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process){:target="_blank"} -->

### Why? What's the point?

I get asked this question quite a bit when I bring up this subject. While I appreciate the benefits of using the "Green Screen" (prompting commands for parameters and help, speedy displays, easy to follow menus), there are also plenty of benefits to having a clean BASH environment.

#### BASH vs BSH

The IBM i has the [Bourne Shell (BSH)](https://en.wikipedia.org/wiki/Bourne_shell "Bourne Shell Wikipedia Article"){:target="_blank"} by default when you SSH in. In fact, IBM i ships with the [Korn, Bourne, and C Shells](http://www-01.ibm.com/support/knowledgecenter/ssw_ibm_i_71/rzalf/rzalfpase.htm?cp=ssw_ibm_i_71&lang=en){:target="blank"}. These are a great starting point, and while BSH works fine for normal tasks and is extendable to do pretty much anything BASH can, BASH offers some added benefits right out of the box that help create a faster and more efficient workflow. Some examples would be tab completion[^1] and command history as well as access to command scrolling with the up and down arrow keys[^2] to name a couple.

### YiPs & Open Source Binaries

I have found [YiPs](http://yips.idevcloud.com/){:target="_blank"} to be a wonderful resource. The [YiPs Open Source Binaries](http://yips.idevcloud.com/wiki/index.php/PASE/OpenSourceBinaries){:target="_blank"} page is a great source for getting started with OSS on the IBM i. From there, one can read instructions for installing the GCC compiler, Git, PostgreSQL, and a few other tested RPMs. Since there are detailed posts on YiPs for installing these, I will try to focus more on tips and tricks and have [summarized step-by-step guides](#summarized-step-by-step-guides) for these installs toward the end of this post.

### BASH in all its glory

#### BASH on SSH login

There are (at least) a couple of different ways to achieve having BASH on login.

##### sshd_confg - Globally (not recommended)

> This will make it so anyone that SSHs in will have BASH at login. More information can be found [here](http://www.itjungle.com/fhg/fhg091714-story01.html){:target="_blank"}

Edit `sshd_config` (usually found at `/QOpenSys/QIBM/UserData/SC1/OpenSSH/openssh-4.7p1/etc/sshd_config`) to have the following:

{% highlight bash %}
# $OpenBSD: sshd_config,v 1.75 2007/03/19 01:01:29 djm Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/QOpenSys/usr/bin:/usr/ccs/bin:/QOpenSys/usr/bin/X11:/usr/sbin:.:/usr/bin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options change a
# default value.

ibmpaseforishell=/path/to/bash # Defines shell for pase. Can be any path to bash you like. For example: /bin/bash
{% endhighlight %}

##### .profile - Locally (recommended)

> This is recommended as it will only change BASH settings on a per-user basis.

Put a `.profile` file in your home directory `/home/USERNAME` with the following contents:

{% highlight bash %}
#!/bin/bash
# .profile
# vim:syntax=sh

# detect if we're in a PASE shell
/QSYS.LIB/QSHELL.LIB/UNAME.PGM > /dev/null 2>&1

if [ $? != 0 -a "$SHELL" != "/QOpenSys/usr/bin/bash" ]
then
  exec /QOpenSys/usr/bin/bash
fi
{% endhighlight %}

#### dotfiles

`dotfiles` are configuration files that are prepended with a dot, or ., so they are "hidden" from view. Some example dotfiles would be `.bashrc`, `.bash_profile`, `.bash_history`, `.vim`, `.weechat`, etc. [dotfiles on Github](https://dotfiles.github.io/){:target="_blank"} is a good place to get started. It has many repositories of dotfiles listed for examples and even ones that have automated installation. For example, [my dotfiles](https://github.com/jbh/dotfiles){:target="_blank"} can be downloaded and then installed by simply running `./script/bootstrap`. Keep in mind that most, if not all, dotfiles you run across will be for Linux or Mac, so make sure to consider compatibility on the IBM i before just copying and pasting someone else's dotfiles. Also, keep in mind that when using Github, your configs are public unless otherwise put into a private repository. Remember to make example config files for files with sensitive data. These can be copied and edited accordingly when the repo is cloned locally.

#### BASH ProTips

##### Shortcuts

One of my favorite conveniences of BASH are the shortcuts and commands it provides. These are not exclusive to BASH. BSH does have *some* of these conveniences.

`esc + .` - Escape-dot. This shortcut will pick the last argument from the previous command. `mkdir /path/to/new/dir` and then `cd [esc + .]` will populate `cd /path/to/new/dir`. You can continue pressing `esc + .` to cycle through previous last arguments.

##### Helpful Commands/Operators

`history` - Lists all previous commands ran by the current user. This is handy, especially when learning BASH.

`grep` - This command is used to search plain-text data sets for lines matching a regular expression. This tool should be in every BASHers toolbox. It can be used to search files, but more importantly to search through piped commands. For example: `history | grep "git"` will show all previous commands that have the word git in it somewhere.

`Bang!` - The bang (`!`) character is very important in BASH. `!!` will echo the last command ran. This is handy if one forgets to prepend `sudo` to a command. Just do `sudo !!` and it will run the last command as sudo. This can also be combined with the `history` command. `!n` will run the command that numbered. `!893` will run the 893rd command in the BASH history.

##### Chaining

Chaining commands is where BASH gets fun. Just by simply using `&&`, `||`, `|`, and other characters, one can chain commands together to create complex commands. Keep in mind that the following operators are not conditionals.

- `&&` - This is the `AND` operator. It can be used to chain commands to gether, and the next command will only execute if and only if the previous command succeeded.
- `||` - This is the `OR` operator, and can be thought of like an `else`. The command following this operator will only run if the previous command fails.
- `&` - This can be used to run commands in the background. This isn't recommended unless you are comfortable enough with the command line to not need to see what's happening while the command runs.
- `;` - This is commonly used to chain several commands together to make them run sequentially. The commands do not rely on one another. Think about it like an ending statement, because that's exactly what it is.
- `|` - The pipe operator. This operator is one of the most helpful, because it pipes the return of the previous command and feeds it into the next. This can be powerful when combining commands like `history | grep "git"`

These operators are key to wielding the command line effectively. Below are a few examples of how these can be used.

{% highlight bash %}
user@host:~# history | grep git #<== Run the history command and search for "git"
  865  git status
  894  git pull
  897  git status
  898  git pull
user@host:~# !898               #<== Run `git pull`
{% endhighlight %}

{% highlight bash %}
user@host:~# mkdir test || cowsay moo               #<== Make the `test` directory, else `cowsay moo`. It passed, so cow-no-say moo
user@host:~# mkdir test || cowsay moo
mkdir: cannot create directory ‘test’: File exists  #<== It failed this time because it already exists. cow-do-say moo
 _____
< moo >
 -----
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
{% endhighlight %}

You can even chain a bunch of commands together, and use a prompt to let you know what it is all done. I do this with [cowsay](https://github.com/schacon/cowsay){:target="_blank"}. This does not come with BASH. One could just use `echo "All done!"`, or whatever is preferred for an alert.

{% highlight bash %}
user@host:~# mkdir Sites && cd Sites && mkdir .conf && git clone git@gitlab.sobo.red:mine/rnp.git && ln -s ~/Sites/rnp/conf/local.conf ~/Sites/.conf/rnp.conf && cowsay "Done dudeski!" || cowsay "Oops :( Something went wrong boss."
Cloning into 'rnp'...
remote: Counting objects: 87, done.
remote: Compressing objects: 100% (77/77), done.
remote: Total 87 (delta 24), reused 0 (delta 0)
Receiving objects: 100% (87/87), 1.78 MiB | 2.34 MiB/s, done.
Resolving deltas: 100% (24/24), done.
Checking connectivity... done.
 _______________
< Done dudeski! >
 ---------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
{% endhighlight %}

##### Aliases

Aliases are very helpful tools on the command line. They allow you to create aliases for commands, chained commands, functions, etc. Think of it as command line shorthand. Below is my `.aliases` file as an example.

{% highlight bash %}
# .aliases
# vim:syntax=sh

# Reload bash aliases
alias reload="source ~/.bash_profile"

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Directory Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias .....='cd ../../../..'
alias ......='cd ../../../../..'

# Add an "alert" alias for long running commands.  Use like so:
#   sleep 10; alert
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'
{% endhighlight %}

##### Functions

I like to think of functions as verbose aliases. They can define much larger sets of commands and aliases. Below is my `.functions` file as an example.

{% highlight bash %}
# .functions
# vim:syntax=sh

#
# Functions on home path
#

function h { cd ~/$1; } # Use: `h Documents` will translate to `cd ~/Documents`
function d { cd ~/Development/$1; } # Use: `d ProjectDir` will translate to `cd ~/Development/ProjectDir`

# Make directory and move to it
mkcdr() { mkdir -p $1 && cd $1; }

# Extract - My personal favorite
extract() {
    if [ -f $1 ] ; then
        case $1 in
            *.tar.bz2)   tar xjf $1 ;;
            *.tar.gz)    tar xzf $1 ;;
            *.bz2)       bunzip2 $1 ;;
            *.rar)       rar x $1 ;;
            *.gz)        gunzip $1 ;;
            *.tar)       tar xf $1 ;;
            *.tbz2)      tar xjf $1 ;;
            *.tgz)       tar xzf $1 ;;
            *.zip)       unzip $1 ;;
            *.Z)         uncompress $1 ;;
            *.7z)        7z x $1 ;;
            *)           echo "'$1' cannot be extracted via extract()" ;;
        esac
    else
        echo "'$1' is not a valid file"
    fi
}
{% endhighlight %}

##### Putting it all together

### Summarized step-by-step guides

#### [Open Source Binaries](http://yips.idevcloud.com/wiki/index.php/PASE/OpenSourceBinaries){:target="_blank"}

1. Download [download-2.0.tar.zip](http://yips.idevcloud.com/wiki/uploads/PASE/download-2.0.tar.zip){:target="_blank"} and extract its contents locally.
2. Go ahead and add the replacement files  [setup2.sh](http://yips.idevcloud.com/wiki/uploads/PASE/setup2.sh){:target="_blank"} and [wwwperzl.sh](http://yips.idevcloud.com/wiki/uploads/PASE/wwwperzl.sh){:target="_blank"} to the unzipped folder.
3. FTP the folder to `/QOpenSys` on your IBM i. I set it up like the following:
    <pre>
        QOpenSys/
            downloads/          <== Can be named anything
                setup2.sh       <== Replacement for setup.sh
                wwwperzl.sh     <== Replacement file for wwwperzl.sh
    </pre>
4. I keep the name `downloads` for the folder because this is where `wwwinstall.sh` will store the rpm files during installation. Think of it like a downloads folder for your OSS.
5. Open `qp2term` on the IBM i, or `ssh` into the IBM i via a terminal on Linux and Mac, or a terminal emulator on Windows and do the following steps in order:
{% highlight bash %}
cd /QOpenSys/downloads
./setup2.sh             <== Installs rpm.rte, wget, untar, perzl deplists, etc. This can take awhile.
./wwwperzl.sh help      <== This ensures it was setup correctly and also gives decent instructions on use.
{% endhighlight %}

#### [Git](http://yips.idevcloud.com/wiki/index.php/PASE/Git){:target="_blank"}

> **Note**: Installing Git will install its many dependencies, including BASH. So keep that in mind and read the `wwwinstallgit.sh` file before running it.

1. Download [wwwinstallgit.sh](http://yips.idevcloud.com/wiki/uploads/PASE/wwwinstallgit.sh){:target="_blank"}.
2. FTP `wwwinstallgit.sh` to `/QOpenSys/downloads` on the IBM i.
3. Open `qp2term` on the IBM i, or `ssh` into the IBM i via a terminal on Linux and Mac, or a terminal emulator on Windows and do the following steps in order:
{% highlight bash %}
cd /QOpenSys/downloads
./wwwinstallgit.sh --all    <== Installs git along with many dependencies. This can take awhile.
git config --global user.name "Your Name"
git config --global user.email youremail@example.com
{% endhighlight %}

#### Using wwwinstall.sh

> **Note**: Use with caution. Always check what is going to be installed, and keep in mind how those might affect the machine. In most cases there's nothing to worry about, but better safe than sorry.

Shell scripts are pretty straight forward. To see what function will download what RPMs, just open `wwwinstall.sh` in your favorite editor, and each RPM is clearly shown. Also, running `wwwinstall.sh help` will give you a list of functions and what they do. For the sake of simplicity, I'm just going to demonstrate the `GetBase` option.

{% highlight bash %}
cd /QOpenSys/downloads
./wwwinstall.sh GetBase
{% endhighlight %}

It's that simple. This file will basically install packages of RPMs for you, so use with caution.

> **ProTip**: Running these commands with their `clean` options will remove all the `.rpm` files when they are done being installed.
{:class="note-protip"}

### Good references

* [BASH Cheat Sheet](http://www.johnstowers.co.nz/blog/pages/bash-cheat-sheet.html){:target="_blank"}
* [BASH to Z-Shell: Conquering the Command Line](http://www.bash2zsh.com/){:target="_blank"}

---

#### Footnotes

[^1]: Tab completion allows the user to type a few characters, press tab, and the shell will try to interpret what the user wants. Double-tapping tab will present the user with multiple options if there are more than one.
[^2]: In BASH, typing `history` will show the history of commands done by the current user. This is handy when piped to grep: `history | grep "[search-term]"`. Pressing up and down on the arrow keys will go to the previous and "next" (if there is one) command in the history list while on the command line.
