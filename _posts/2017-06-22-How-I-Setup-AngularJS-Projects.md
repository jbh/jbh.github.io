---
layout: post
title: How I Setup AngularJS Projects
excerpt: |
    This article will cover my personal, opinionated way of setting up AngularJS projects. This way of doing it
    is admittedly simplistic.
image: /images/rpc-ibm-db2/stored-procedure-action.png
categories:
    - angularjs
---

* TOC
{:toc}

### Brief Explanation

My setup of AngularJS is simplistic. I rely on APIs for my backend, so services usually boil down to nothing but
methods that call REST API routes. I should probably leverage directives more, but API routes are easily exposed
by quickly creating services to access them. For this article, I would like to quickly define some terms in
context of this setup.

**[AngularJS](https://docs.angularjs.org/api){:target="_blank"}**
: When I say AngularJS, I mean Angular v1. When I say Angular, I am referring to Angular v2+. My Angular projects are
structured differently than my AngularJS projects.

**[State](https://ui-router.github.io/ng1/){:target="_blank"}**
: For my projects, states are usually synonymous with a route. When a state is changed, a controller is assigned and the
URI is usually modified via a
[pushState](https://developer.mozilla.org/en-US/docs/Web/API/History_API#The_pushState()_method){:target="_blank"}.
However, a state does not necessarily require a route change, just to be clear. This is all handled through
`angular-ui-router` in my AngularJS projects.

**[Service](https://docs.angularjs.org/guide/services){:target="_blank"}**
: For my projects, a service usually becomes nothing more than a few methods that access an API. For example, one might
have a service called `UserService` that only has CRUD methods like `find(id)`, `getAll()`, `create(user)`,
`delete(id)`. Then one could call API routes by simply calling `UserService.find(1)` in the Component.

**[Component](https://docs.angularjs.org/guide/component){:target="_blank"}**
: Components, directives, services, etc. are all controllers with different use cases and configurations. A component,
in my view, is a controller that lends itself to be used as a view/route/state/page controller. So, components can be
thought of as the controller for the views in this case. Bear in mind that a view is not the entire page, but a single
component on the page. A view could be a forum thread, for example. A post itself in the thread could be its own view
with its own component controller and state.

**[View](https://ui-router.github.io/guide/views){:target="_blank"}**
: When I refer to views, I'm speaking of `ui-view` and not `ng-view`. The difference being that `angular-ui-router` uses
`ui-view`.

### Tools used

**[Node.js](https://nodejs.org/en/){:target="_blank"}/[npm](https://www.npmjs.com/){:target="_blank"}**
: [Gulp](http://gulpjs.com/){:target="_blank"} is used in my AngularJS projects in order to minify all of the JavaScript
files. It's nice to have all the separate components, directives, services, and such, but I don't want to have that
many separate JavaScript files loaded, so I minify them into one. Sourcemapping allows for easy debugging, as the
console in the browser will still tell you the original filename and line when logging. I could apply the same idea to
the CSS, but my projects are relatively light on CSS, so I do not.

**[Bower](https://bower.io/){:target="_blank"}**
: I currently use Bower to maintain my frontend dependencies. I use `.bowerc` to force it to put dependency files under
`htdocs/assets/vendor`, because I put CSS and JavaScript under `assets`, and I like to be consistent with the word
vendor from composer.

> Even Bower themselves suggest using [yarn](https://yarnpkg.com/){:target="_blank"} or
[webpack](https://webpack.js.org/){:target="_blank"} for new projects.

**[Composer](https://getcomposer.org/){:target="_blank"}**
: I use composer to handle my PHP dependencies. I use PHP for a small wrapper that will apply an OAuth key securely to
the route being called. My wrapper is how I handle OAuth. There might well be a better way, but this is the easiest
way I found to get it working on the IBM i. **This could be completely ignored if your project doesn't need this sort
of security.** In fact, for the rest of this article, it will be mostly ignored. Just assume that most service methods
will be calling `apiWrapper.php` with a route, and `apiWrapper.php` will apply any necessary key and then call the API
route passed.


### Initial Setup & Folder Structure

I have a skeleton AngularJS application called Maya that I clone and start a project with. Maya has a structure similar
to:

```
maya/
  conf/                       <== Apache configuration files
    local.conf
    prod.conf
  htdocs/                     <== All files accessible to the web user
    app/                      <== Angular application files
      components/             <== Component-specific components, views, services, directives, etc.
        home/
          home.component.js
          home.html
      shared/                 <== Shared components, views, services, directives, etc.
        example/
          example.service.js
      app.module.js
      app.states.js
    assets/
      css/
      fonts/
      js/
        prototype-extensions/ <== Extensions to JavaScript prototypes
          array.js
          object.js
          string.js
      vendor/                 <== Where bower third-party files end up
    index.php
    apiWrapper.php            <== A small wrapper to apply OAuth keys if necessary
  logs/
  vendor/
  .bowerrc                    <== Bower configuration. This is where I set that vendor files go in htdocs/assets/vendor
  bower.json                  <== Bower project information and dependencies
  composer.json               <== Composer project information and dependencies
  gulpfile.js
  package.json                <== npm project information and dependencies
```

#### Tool configuration examples

**Bower**

.bowerrc

```json
{
  "directory": "htdocs/assets/vendor"
}
```

bower.json

```json
{
  "name": "project-name-maya",
  "authors": [
    "Author Name <author_email@example.com>"
  ],
  "description": "Project description",
  "keywords": [
    "project",
    "keywords"
  ],
  "license": "MIT",
  "homepage": "",
  "private": true,
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "htdocs/assets/vendor",
    "test",
    "tests"
  ],
  "dependencies": {
    "angular-material": "^1.1",
    "angular-ui-router": "^0.2.18",
    "angular-utils-ui-breadcrumbs": "^0.2.2",
    "angular-material-data-table": "^0.10.10",
    "angular-material-icons": "^0.7.1",
    "font-awesome": "^4.6.3",
    "angular-pdf": "^1.3.0",
    "angular-sanitize": "^1.5.8"
  }
}
```

> I do not suggest copying and pasting this bower.json. Instead, initiate a bower project and install the dependencies
normally through bower. This will ensure the proper versions are installed.

**npm**

The npm setup is simple. It only contains gulp dependencies.

package.json

```json
{
  "name": "project-name-maya",
  "version": "1.0.0",
  "description": "Project description",
  "devDependencies": {
    "gulp": "^3.9.1",
    "gulp-concat": "^2.6.0",
    "gulp-rename": "^1.2.2",
    "gulp-sourcemaps": "^1.6.0",
    "gulp-uglify": "^1.5.4",
    "gulp-watch": "^4.3.9",
    "pump": "^1.0.1"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "MIT"
}
```

> Again, I do not suggest copying and pasting this package.json. Instead, initiate a npm project and install the
dependencies normally through npm.

**Gulp**

gulpfile.js

```javascript
var gulp          = require("gulp"),
    sourcemaps    = require("gulp-sourcemaps"),
    uglify        = require("gulp-uglify"),
    rename        = require("gulp-rename"),
    concat        = require("gulp-concat"),
    watch         = require("gulp-watch"),
    pump          = require("pump");

gulp.task("compress:vendor", function (cb) {
    pump([
            gulp.src([
                __dirname + '/htdocs/assets/js/prototype-extensions/*.js',
                __dirname + '/htdocs/assets/vendor/angular/angular.min.js',
                __dirname + '/htdocs/assets/vendor/angular-animate/angular-animate.min.js',
                __dirname + '/htdocs/assets/vendor/angular-aria/angular-aria.min.js',
                __dirname + '/htdocs/assets/vendor/angular-messages/angular-messages.min.js',
                __dirname + '/htdocs/assets/vendor/angular-material/angular-material.min.js',
                __dirname + '/htdocs/assets/vendor/angular-material-data-table/dist/md-data-table.js',
                __dirname + '/htdocs/assets/vendor/angular-material-icons/angular-material-icons.min.js',
                __dirname + '/htdocs/assets/vendor/angular-ui-router/release/angular-ui-router.min.js',
                __dirname + '/htdocs/assets/vendor/angular-utils-ui-breadcrumbs/uiBreadcrumbs.js',
                __dirname + '/htdocs/assets/vendor/angular-sanitize/angular-sanitize.js',
            ]),
            sourcemaps.init(),
            concat('concat.js'),
            gulp.dest('htdocs/assets/js'),
            rename('vendor.min.js'),
            uglify({
                mangle: false
            }),
            sourcemaps.write(),
            gulp.dest("htdocs/assets/js")
        ],
        cb);
})

gulp.task("compress:client-app", function (cb) {
    pump([
        gulp.src([
            __dirname + '/htdocs/app/app.module.js',
            __dirname + '/htdocs/app/app.states.js',
            __dirname + '/htdocs/app/components/**/*.js',
            __dirname + '/htdocs/app/shared/**/*.js'
        ]),
        sourcemaps.init(),
        concat('concat.js'),
        gulp.dest('htdocs/assets/js'),
        rename('main.min.js'),
        uglify({
            mangle: false
        }),
        sourcemaps.write(),
        gulp.dest("htdocs/assets/js")
    ],
    cb);
});

gulp.task('watch', ['compress:client-app'], function () {
    watch(['htdocs/app/**/*.js', 'htdocs/assets/js/prototype-extensions/*.js'], function () {
        gulp.start('compress:client-app')
    });
});

gulp.task('compile', ['compress:client-app', 'compress:vendor']);

gulp.task("default", ["watch"]);
```

For Gulp, I keep vendor assets and app assets separate. This way I can compress them separately. In doing so, I save
a lot of time when running `gulp watch`, as only the few client app files are compressed at that time. Sure, I have two
JavaScript files, but I could combine those if I really wanted to.

> I'm sure there are a million different better ways to handle vendor files and minification. Right now, I manually add
any vendor files that I add to the gulpfile. I try to keep my frontend as lightweight as possible, so my dependencies
are usually few. This way works fine for me now.
