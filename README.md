[![Build Status](https://travis-ci.org/mkay581/router-js.svg?branch=master)](https://travis-ci.org/mkay581/router-js)

# RouterJS

A simple framework for single-page, in-browser apps that allows you to load, show, and
hide "pages" dynamically when urls are requested without having to refresh the page.
Also allows you to map specific modules to pages all through one simple configuration file.

As seen on [fallout4.com](http://www.fallout4.com).

## Benefits

* Loads scripts, templates, data, and css files
* Supports Browserify and RequireJS builds
* Caches requests for faster performance
* Supports handlebar templates (.hbs) allowing them to be loaded on the client-side


## Prerequisites

Before you begin using, you must setup your server to have all of urls point to 
your index.html page that will house your code.
 

## Setup

### 1. Create a container element for your pages

First, create your index.html or similar (if you don't already have one) with at least 
one html element that your pages will be shown in.

```html
<html>
    <body>
        <div class="page-container"></div>
    </body>
</html>
```

### 2. Style your divs

When a page url (route) is requested, css classes are applied and removed. So you'll need to setup a few lines of css 
to show and hide based on the css classes that Router applies. 

```css
.page {
    display: none;
}

.page-active {
    display: block;
}
```

Of course, you can use fancier CSS transitions if you'd like.

### 3. Configure your modules, pages and routes

Create your modules configuration:

```javascript
var modules = {
    'header': {
        script: 'path/to/header.js',
        template: 'path/to/header.html',
        data: 'url/to/my/header/data',
        global: true
    },
    'custom-module': {
        script: 'custom/module/path.js',
        template: 'custom/module/template.html'
    }
};
```

Then map your urls to your pages in another configuration object.

```javascript
var pages = {
    '^home(/)?$': {
        template: '/path/to/homepage.html',
        modules: [
            'header',
            'custom-module'
        ],
        script: 'home-page.js',
        data: 'url/to/home-page/data'
    }
};

```

### 4. Start Routing

You must start the Router and pass it your routes object to begin listening in on url requests.

```javascript
var Router = require('router-js');
Router.start({
    pagesConfig: pages,
    modulesConfig: modules,
    pagesContainer: document.body.getElementsByClassName('page-container')[0]
});
```

Then, when a user requests the `/home` url,  the templates, script, modules and data 
under your `home` pages config entry will load instantly.


## Usage

### Initial Page Load

When loading the initial page from your browser, the Router could possibly load before the DOM has been loaded
(depending on when you decide to call `Router.start()`). If so, you'll need to listen for the DOM to be loaded,
and then trigger the current url as illustrated below.
The code below should be ran right after your `Router.start()` call.

```javascript
window.addEventListener('DOMContentLoaded', function () {
    Router.triggerRoute(window.location.pathname);
});
```

### Triggering URLs

If you need to trigger new urls in your javascript programmatically, you can do things like this:

```javascript
Router.triggerRoute('home').then(function () {
   // home page element has been injected into DOM and active class has been applied
});
```

### Regex URL Captured Groups

The router accepts regex entries and [captured groups] in your pages configuration. So for instance, if you wanted all
urls going to [/profile/[SOME_ID]](/profile/[SOME_ID]) to request data from http://someurl.com/[SOME_ID] and
inject it into your template. You could set up your page configuration like so:

```javascript
var pages = {
    '^profile/([0-9])': {
        template: '/path/to/profile.hbs',
        data: 'http://someurl.com/$1'
    }
};

```

Then any id used when accessing /profile/[SOME_ID], will request the appropriate http://someurl.com/[SOME_ID] uri.

### Important Notes

* Any javascript files that you include in your routes configuration must be "require"-able using either 
Browserify, RequireJS or any other script loader that exposes a global "require" variable.
* Once a CSS file is loaded, it is loaded infinitely, so it's important to namespace your styles and be specific 
 if you do not want your styles to overlap and apply between pages.
* When a page is loaded, it is cached and will remain in the DOM until you physically remove the element in your custom destroy logic.
