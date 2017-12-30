---
layout: post
title: IBM i Open Source Tools
excerpt: A quick explanation about IBM i Open Source Tools, 5733-OPS, and how quickly it is evolving.
categories:
    - ibm i
---

* TOC
{:toc}

> IBM i Open Source Tools were originally installed via a complex process of installing packages from
[Perzl](http://www.perzl.org/aix/){:target="_blank"}. I have described how to do this in past posts such as
[BASH on IBM i](/BASH-on-IBM-i){:target="_blank"}. While this was helpful in the past, it wasn't officially supported
by IBM. [5733-OPS](http://p.jbh.io/d){:target="_blank"} is a new product from IBM that supports Open Source Tools
and can be installed on the IBM i.


### 5733-OPS

5733-OPS brings many open source tools to the IBM i and helps create a development environment that is familiar
to a unix or linux developer. Tools such as BASH, rsync, curl, git, python, and much more are included. See the
[Open Source Technologies](http://p.jbh.io/d){:target="_blank"} details page for more information on what is included
with each option.

#### How to Install 5733-OPS

[Kevin Adler](http://p.jbh.io/g){:target="_blank"} has written a thorough and detailed
[article](http://p.jbh.io/e){:target="_blank"} that explains exactly how to obtain 5733-OPS.

### Tools

Below are quick references to the pros for some of the tools and how they development.

#### BASH

BASH is an enhanced version of BSH. It gives the developer some extra features. Anyone that has had to use the default
shell on the IBM i has probably ran into the issue of the arrow keys and backspace virtually being unusable. Make a
mistake in BSH and you might as well `ctrl+c` to start writing the command all over again. At least it gives you an
ascii heart to make you feel a little better about the mistake.

BASH allows for arrow keys, which means up arrow for past commands. The developer will also be able to use tab
completion. It even enables shortcuts like `esc + .` to cycle through past parameters.

I have written an article that describes a [pretty BASH configuration](/Pretty-BASH-Configuration-with-IBM-i-Helpers).
There's a section
[specifically for the IBM i](/Pretty-BASH-Configuration-with-IBM-i-Helpers/#ibm-i-specific-highlights).

#### Git

Having a simple way of bringing Git to the IBM i is one of the most advantageous features of 5733-OPS. Git allows for
simple and widely accepted version control practices. Atlassian has a great
[tutorial](https://www.atlassian.com/git/tutorials/comparing-workflows){:target="_blank"} that compares the different
workflows of Git. GitHub also has a nice [tutorial](https://try.github.io){:target="_blank"} as an introduction to Git.
Please keep in mind that Git and GitHub are not the same thing. GitHub is a cloud solution for hosting Git repositories.

#### Node

Bringing Node to the IBM i is helpful in many ways, and it's not just for using Node as a web development language.
Having access to Node packages such as [Grunt](https://www.npmjs.com/package/grunt){:target="_blank"},
[Gulp](https://www.npmjs.com/package/gulp){:target="_blank"}, and some others can be used as basic command line tools to
automate tasks. Before `which` was available on the IBM i, I could use the Node package
[which](https://www.npmjs.com/package/which){:target="_blank"} to find commands.

There are other tools that are included, but these are the ones I use the most. I'd love to hear what everyone else's
favorite parts of 5733-OPS are. I know it has made web development a much smoother process.
