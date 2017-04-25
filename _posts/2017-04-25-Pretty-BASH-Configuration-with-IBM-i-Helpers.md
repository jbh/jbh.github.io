---
layout: post
title: Pretty BASH Configuration with IBM i Helpers
excerpt: |
    A simple BASH configuration example on the IBM i with helper functions specific to the platform. These examples
    can be used to make more IBM i specific BASH methods.
categories:
    - ibm i
    - bash
---

* TOC
{:toc}

### Handy Variables, Aliases, and Functions

I suggest starting everything with some color variables, since they usually end up repeated a lot when creating a decent
looking bash environment.

```bash
RESET="\033[0m"
BOLD="\033[1m"
BLACK="\033[30m"
RED="\033[31m"
GREEN="\033[32m"
YELLOW="\033[33m"
BLUE="\033[34m"
MAGENTA="\033[35m"
CYAN="\033[36m"
WHITE="\033[37m"
BG_BLACK="\033[40m"
BG_RED="\033[41m"
BG_GREEN="\033[42m"
BG_YELLOW="\033[43m"
BG_BLUE="\033[44m"
BG_MAGENTA="\033[45m"
BG_CYAN="\033[46m"
BG_WHITE="\033[47m"
```

That's usually followed with prettification of PS1 and the most basic aliases.

> In order for this particular PS1 configuration to work
[git-prompt.sh](https://github.com/git/git/blob/master/contrib/completion/git-prompt.sh){:target="_blank"} must be used.

```bash
# Prettify the bash prompt. This will give us something like
# <folder> (<git-branch>) →
source "${HOME}/git-prompt.sh"
export PS1="${CYAN}\W${RESET} $(__git_ps1 "(${GREEN}%s${RESET}) ")${RED}→${RESET} "

# Reload bash configuration
alias reload="source ${HOME}/.bashrc"

# helpful ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
```

Then comes all the fun stuff.

```bash
# This is helpful for writing bash scripts.
# Use: if ask "Continue?" Y;
function ask() {
    # http://djm.me/ask
    while true; do

        if [ "${2:-}" = "Y" ]; then
            prompt="Y/n"
            default=Y
        elif [ "${2:-}" = "N" ]; then
            prompt="y/N"
            default=N
        else
            prompt="y/n"
            default=
        fi

        # Ask the question - use /dev/tty in case stdin is redirected from somewhere else
        echo -en "$1 $GREEN($prompt)$RESET "
        read REPLY </dev/tty

        # Default?
        if [ -z "$REPLY" ]; then
            REPLY=$default
        fi

        # Check if the reply is valid
        case "$REPLY" in
            Y*|y*) return 0 ;;
            N*|n*) return 1 ;;
        esac

    done
}

# Directory Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias .....='cd ../../../..'
alias ......='cd ../../../../..'
function h { cd ${HOME}/$1; }                         # Use: h Projects/dotfiles
function d { cd /www/development/developer-name/$1; } # Use: d project-name
function s { cd /www/stage/$1; }                      # Use: s project-name
function p { cd /www/production/$1; }                 # Use: p project-name

# Make directory and move to it
function mkcdr() { mkdir -p $1 && cd $1; }

# Extract
function extract() {
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
```

Those are just a few examples of handy tools a developer can create when tailoring their bash environment.

#### IBM i Specific Highlights

Thanks [@PHPDave](https://twitter.com/php_dave) for the inspiration.

```bash
# IBM i DB2 alias.
# Use: db2 "select * from library.file"
function db2() {
    echo "Running $1";
    system -i "call QSYS/QZDFMDB2 parm('$1')";
}

# Recursively gives permission for QTMHHTTP to directory passed
# Use: http-permissions /www/production/project-name/htdocs
function http-permissions() {
    system -i "CHGAUT OBJ('${1}') USER(QTMHHTTP) DTAAUT(*RWX) OBJAUT(*ALL) SUBTREE(*ALL)";
}

# Call WRKACTJOB on IBM i
function wrkactjob() {
    system WRKACTJOB;
}

# Display the system CCSID value
function qccsid() {
    system "DSPSYSVAL SYSVAL(QCCSID)";
}

# Zend Server tool
function zend-server() {
    case $1 in
        start)
            system "STRTCPSVR SERVER(*HTTP) HTTPSVR(ZENDSVR)"
            ;;
        begin)
            system "STRTCPSVR SERVER(*HTTP) HTTPSVR(ZENDSVR)"
            ;;
        stop)
            system "ENDTCPSVR  SERVER(*HTTP) HTTPSVR(ZENDSVR)"
            ;;
        end)
            system "ENDTCPSVR  SERVER(*HTTP) HTTPSVR(ZENDSVR)"
            ;;
        jobs)
            ps -ef | grep -i zend
            ;;
        *)
            echo $"Usage: zend-server {start|begin|stop|end|jobs}"
    esac
}
```

Those are a few of my favorite ones. Explore other ways to make your own environment more efficient. It's what makes
BASH fun and worthwhile.

### Putting It All Together

> There are a few placeholders in this example configuration. It should be combed through and tailored before use.

```bash
# Example ~/.bashrc

# colors
RESET="\033[0m"
BOLD="\033[1m"
BLACK="\033[30m"
RED="\033[31m"
GREEN="\033[32m"
YELLOW="\033[33m"
BLUE="\033[34m"
MAGENTA="\033[35m"
CYAN="\033[36m"
WHITE="\033[37m"
BG_BLACK="\033[40m"
BG_RED="\033[41m"
BG_GREEN="\033[42m"
BG_YELLOW="\033[43m"
BG_BLUE="\033[44m"
BG_MAGENTA="\033[45m"
BG_CYAN="\033[46m"
BG_WHITE="\033[47m"

# Prettify the bash prompt. This will give us something like
# <folder> (<git-branch>) →
source "${HOME}/git-prompt.sh"
export PS1="${CYAN}\W${RESET} $(__git_ps1 "(${GREEN}%s${RESET}) ")${RED}→${RESET} "

# Reload bash configuration
alias reload="source ${HOME}/.bashrc"

# helpful ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# This is helpful for writing bash scripts.
# Use: if ask "Continue?" Y;
function ask() {
    # http://djm.me/ask
    while true; do

        if [ "${2:-}" = "Y" ]; then
            prompt="Y/n"
            default=Y
        elif [ "${2:-}" = "N" ]; then
            prompt="y/N"
            default=N
        else
            prompt="y/n"
            default=
        fi

        # Ask the question - use /dev/tty in case stdin is redirected from somewhere else
        echo -en "$1 $GREEN($prompt)$RESET "
        read REPLY </dev/tty

        # Default?
        if [ -z "$REPLY" ]; then
            REPLY=$default
        fi

        # Check if the reply is valid
        case "$REPLY" in
            Y*|y*) return 0 ;;
            N*|n*) return 1 ;;
        esac

    done
}

# Directory Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias .....='cd ../../../..'
alias ......='cd ../../../../..'
function h { cd ${HOME}/$1; }                               # Use: h Projects/dotfiles
function d { cd /www/development/developer-name/$1; } # Use: d project-name
function s { cd /www/stage/$1; }                      # Use: s project-name
function p { cd /www/production/$1; }                 # Use: p project-name

# Make directory and move to it
function mkcdr() { mkdir -p $1 && cd $1; }

# Extract
function extract() {
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

# IBM i DB2 alias.
# Use: db2 "select * from library.file"
function db2() {
    echo "Running $1";
    system -i "call QSYS/QZDFMDB2 parm('$1')";
}

# Recursively gives permission for QTMHHTTP to directory passed
# Use: http-permissions /www/production/project-name/htdocs
function http-permissions() {
    system -i "CHGAUT OBJ('${1}') USER(QTMHHTTP) DTAAUT(*RWX) OBJAUT(*ALL) SUBTREE(*ALL)";
}

# Call WRKACTJOB on IBM i
function wrkactjob() {
    system WRKACTJOB;
}

# Display the system CCSID value
function qccsid() {
    system "DSPSYSVAL SYSVAL(QCCSID)";
}

# Zend Server tool
function zend-server() {
    case $1 in
        start)
            system "STRTCPSVR SERVER(*HTTP) HTTPSVR(ZENDSVR)"
            ;;
        begin)
            system "STRTCPSVR SERVER(*HTTP) HTTPSVR(ZENDSVR)"
            ;;
        stop)
            system "ENDTCPSVR  SERVER(*HTTP) HTTPSVR(ZENDSVR)"
            ;;
        end)
            system "ENDTCPSVR  SERVER(*HTTP) HTTPSVR(ZENDSVR)"
            ;;
        jobs)
            ps -ef | grep -i zend
            ;;
        *)
            echo $"Usage: zend-server {start|begin|stop|end|jobs}"
    esac
}

# Recursively gives permission for QTMHHTTP to directory passed
# Use: http-permissions /www/production/project-name/htdocs
function http-permissions() {
    system -i "CHGAUT OBJ('${1}') USER(QTMHHTTP) DTAAUT(*RWX) OBJAUT(*ALL) SUBTREE(*ALL)"
}

```

### Taking it all apart

While it's acceptable to have everything in one large `.bashrc` located at the root of one's home directory,
it's a good idea to split up the configuration into smaller, more manageable files. Configurations can get large,
especially when complex functions get thrown into the mix.

My file structure ends up looking something like:

```
~/
    .bash_start
    .bash_aliases
    .bash_functions
    .bash_end
    .bashrc
    .bash-functions/   <== For more complex bash functions
        zend_server.sh
        ask.sh
```

To see how they connect, here are `.bashrc`, `.bash_start`, and `.bash_functions`:

```bash
# .bashrc

source "${HOME}/.bash_start"
source "${HOME}/.bash_aliases"
source "${HOME}/.bash_functions"
source "${HOME}/.bash_end"
```

```bash
# .bash_start

# colors
RESET="\033[0m"
BOLD="\033[1m"
BLACK="\033[30m"
RED="\033[31m"
GREEN="\033[32m"
YELLOW="\033[33m"
BLUE="\033[34m"
MAGENTA="\033[35m"
CYAN="\033[36m"
WHITE="\033[37m"
BG_BLACK="\033[40m"
BG_RED="\033[41m"
BG_GREEN="\033[42m"
BG_YELLOW="\033[43m"
BG_BLUE="\033[44m"
BG_MAGENTA="\033[45m"
BG_CYAN="\033[46m"
BG_WHITE="\033[47m"

# Prettify the bash prompt. This will give us something like
# <folder> (<git-branch>) →
source "${HOME}/git-prompt.sh"
export PS1="${CYAN}\W${RESET} $(__git_ps1 "(${GREEN}%s${RESET}) ")${RED}→${RESET} "
```

```bash
# .bash_functions

# Order matters. It is assumed that `ask` might be used by other functions.
source "${HOME}/.bash-functions/ask.sh"

# Display the system CCSID value
function qccsid() {
    system "DSPSYSVAL SYSVAL(QCCSID)";
}

source "${HOME}/.bash-functions/zend_server.sh"
```
