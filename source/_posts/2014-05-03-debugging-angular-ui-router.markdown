---
layout: post
title: "Debugging angular ui-router"
date: 2014-05-03 08:21:40 +0530
comments: true
categories: javascript angularjs angular-ui ui-router debug
---

The [ui-router](https://github.com/angular-ui/ui-router) from [angular-ui](https://github.com/angular-ui) gives power of nested states and multiple named views which is essential for angular apps with non-trivial layout and workflow.

The main problem with angular ui-router is getting the route configurations right in the first time. You start seeing blank pages for misconfigured views. The sad part is these errors are not logged to console. It gets quite frustrating in tracking down the cause.

This where I found this [stackoverflow link](http://stackoverflow.com/a/20786262/69362
) which helps in solving this issue. If you find yourself in trouble with ui-router, just open the developer console on the browser and paste the beloew code.

<script src="https://gist.github.com/endeepak/387102e7bc94f2ef505e.js"></script>

#### Some of the common mistakes made while moving fron angular's ng-route to ui-router are

+ Dependency on ui-router is not added

```js
//Make sure 'ui.router' module is added as dependency
var myApp = angular.module('myApp', ['ui.router']);
```

+ The parent state doesn't have template with ui-view

```js
$stateProvider
.state('user', { //parent state
    url: '/user/:userId',
    // Important that parent has a template with ui-view which the child states can replace
    template: '<ui-view/>' 
})
.state('user.projects', { //child state
    url: 'projects',
    templateUrl: 'projects.html',
})
```