---
layout: post
title: LoopBack on IBM i
excerpt: |
    A guide for installing LoopBack on IBM i and utilizing
    loopback-connector-db2ibmi to access DB2 data.
image: /images/loopback/loopback-model.jpg
categories:
    - loopback
    - ibm i
    - db2
---

* TOC
{:toc}

> `loopback-connector-db2ibmi` is still pre-pre-alpha. Use with caution.

### Installing LoopBack

This uses
[loopback-cli](https://loopback.io/doc/en/lb3/Installation.html#install-loopback-cli-tool)
in order to create a LoopBack Project. That is the `lb` command seen below. A
list of `loopback-cli` commands can be found
[here](https://loopback.io/doc/en/lb3/Command-line-tools.html).

![LoopBack Install](/images/loopback/loopback-install.gif)

The commands above in order are simply:

```bash
$ lb # Follow configuration instructions. Above I choose default for each.
$ npm install loopback-connector-db2ibmi --save
```

This will give us everything we need to begin making an API. To start with,
let's define our data source.

### Defining a Data Source

At the moment, a data source will need to be created for each library. So, in
this case, think of a data source as a library, or schema, in the database.
Let's first create the data source via `lb datasource`, and then we'll edit
the contents of `datasource.js` once it the source has been generated.

![New LoopBack Datasource](/images/loopback/new-datasource.gif)

All that needs to happen for now is to add a schema to the data source:

![Modify LoopBack Datasource](/images/loopback/modify-datasource.gif)

Now that we have a data source, we can create models to our heart's content,
and they'll have CRUD + Querying out of the box.

### Defining Models

Below is me defining a model via `lb model` and filling out the configuration to
match `ECOMMERCE_USERS`, which I made in my
[Installing and Using Apigility on IBM i](/2017-04-21-Installing-and-Using-Apigility-on-IBM-i)
article. Tables, columns, etc. are all case sensitive. This is why you see me
yelling all my properties and such.

![New LoopBack Model](/images/loopback/new-model.gif)

Don't forget to define which column is the identity column. This is defined
within `project-dir/common/models/ecommerce_users.json`.

![Modify LoopBack Model](/images/loopback/modify-model.gif)

That's it! Now we have a nifty API out of the box to manipulate our data we just
defined with a model. Just start the server with `node .`, and go to the address
in your browser to explore your API. Make sure to define the correct port in
the `project-dir/server/config.json`. For example, the port I had to use for my
Litmis Space was `63433`.

Below is me exploring the API at `http://spaces.litmis.com:63433/explorer`. I
didn't do anything other than what I did above for these requests to work.

![Modify LoopBack Model](/images/loopback/explore-api.gif)
