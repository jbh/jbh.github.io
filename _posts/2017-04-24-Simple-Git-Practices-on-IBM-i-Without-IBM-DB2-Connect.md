---
layout: post
title: Simple Git Practices on IBM i Without IBM DB2 Connect
excerpt: |
    How to take utilize Git on the IBM i with multiple development, stage, and production environments without
    the need for IBM DB2 Connect.
categories:
    - ibm i
    - git
    - db2
---

* TOC
{:toc}

### The Problem

Ideally, application development is separated into three environments: local (development), staging, and production.
Staging and production would reside on the IBM i, while local is specific to each developer and would reside on
the developer's local development machine. In a perfect world, the developer would have a mock of the database on
their local for development, but DB2 is proprietary to IBM i. Therefore, having a local copy of the database is
improbable at best, so how do we gain access to the data for development?

Currently, to my knowledge, there is no way to remotely access DB2 data unless one purchases
a license for [IBM DB2 Connect](https://www.ibm.com/bb-en/marketplace/db2-connect){:target="_blank"}. This makes
local, staging, and production environment development difficult.

One way to alleviate this frustration is to completely [decouple the backend](/Installing-and-Using-Apigility-on-IBM-i)
of applications from the frontend by developing a REST, CRUD, or other type of API in order to have web-callable
services. This at least allows one to develop the frontend of applications locally. However, then the question
arises: what about the API? How does one properly separate the environments for applications that must be developed
**on** the IBM i? What are good practices to keep each developer from stepping on others' toes?

This article will discuss how to setup a friendly development environment on the IBM i that allows for individual
development environments along with staging and production environments. Basic knowledge of git
and remote repositories is assumed to keep this light.

### File Structure and Apache Configuration

The file structure is important here. It will help us stay organized and give each developer their own environment.

> **Protip**: Keep apache configurations within each specific project. In this example they're under `project-root/conf`.
This allows for apache configurations for each environment to be version controlled. It also helps with organization
and will make enabling/disabling web applications a breeze. Simply add `IncludeOptional /www/.conf-enabled/*.conf` to
the end of the root Apache configuration file. This is usually found through either the Apache administration web
interface or through the IFS. The file is usually located at `/www/zendsvr6/conf/httpd.conf`. I leave the
`/www/zendsvr6` up as a placeholder website to fall back on. It usually just displays something like, "IT WORKS!!!"
just to prove apache is at least working.

```
/
  www/
    .conf-enabled/                   <=== Symlinked apache configuration files for each environment
      app-one-dev-developer-one.conf <=== Symlinked to /www/development/developer-one/app-one/config/developer-one.conf
      app-one-stage.conf
      app-one-production.conf
    development/
      developer-one/
        app-one/                     <=== Git repo on any development branch
          conf/                      <=== Project-specific, versioned apache configuration files that
            developer-one.conf
            developer-two.conf
            stage.conf
            production.conf
          public/                    <=== Public web files - some name it htdocs
        app-two/                     <=== Git repo on any development branch
        app-three/                   <=== Git repo on any development branch
      developer-two/
        app-one/                     <=== Git repo on any development branch
        app-four/                    <=== Git repo on any development branch
    stage/
      app-one/                       <=== Git repo on the stage branch
      app-two/                       <=== Git repo on the stage branch
      app-three/                     <=== Git repo on the stage branch
      app-four/                      <=== Git repo on the stage branch
    production/
      app-one/                       <=== Git repo on the master branch
      app-two/                       <=== Git repo on the master branch
    zendsvr6/
      conf/                          <=== Root apache configuration files
      htdocs/
        index.html                   <=== Simple landing page to show apache works. Nothing more.

```

{% highlight apache %}
# /www/development/developer-one/app-one/conf/developer-one.conf
<VirtualHost *:80>
    ServerName developer-one.app-one.ibmiserver.com
    DocumentRoot /www/development/developer-one/app-one/public

    SetEnv APPLICATION_ENV "development"

    <Directory "/www/development/developer-one/app-one/public">
        Options FollowSymLinks
        order allow,deny
        allow from all
        AllowOverride all
    </Directory>
</VirtualHost>
{% endhighlight %}

### Remote Syncing

While the developer could edit the remote files directly, I've found it better to sync the files remotely with local
files. This gives one a few different benefits. One of which is being able to quickly search the entire project since
it is searching local files.

Webstorm, PHPStorm, Netbeans, and a few other IDEs have this capability built in. If one is using Sublime or Atom,
I suggest finding a remote syncing plugin they like. I personally like
[Sublime SFTP](https://wbond.net/sublime_packages/sftp){:target="_blank"}.

Doing this is only useful for the development environment folders. Files under stage and production should never
be directly edited. They should only ever need to have `git pull` performed on them to sync files.

### No DB2 Connect

How does this help us remove the need for DB2 Connect? Well, now we are able to develop all our applications
on the IBM i while still retaining development environments as if we were developing locally. This gives us
complete access to the DB2 data without the need to access it remotely.

While all applications can be developed with remote syncing, I still suggest decoupling the backend. Not only is this best
practice, but it will allow the data to be accessed the same way by many different applications. This helps
with consistency, standards, and encapsulation. This will also allow the majority of applications to be developed
locally. Ideally there would be one application that need be edited remotely, and that would be the API application.
The rest would be frontend applications that consume the API services to deliver fast, user-friendly experiences.
