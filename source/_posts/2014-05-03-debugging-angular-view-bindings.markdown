---
layout: post
title: "Debugging angular view bindings"
date: 2014-05-03 09:34:54 +0530
comments: true
categories: javascript angularjs debug
---

The way to debug problems with <code>\{\{\}\}</code> or <code>ng-bind</code> or any other directive is to use [angular.element](https://docs.angularjs.org/api/ng/function/angular.element) method. 

<!-- more -->

Example: If you want to debug missing data in <code>\<span>Name : \{\{person.fullName()\}\}\</span></code>

* Right click on the span and click Inspect element. Make sure span is selected in the elements tab
* Go to console tab and type 

```js
angular.element($0).scope().pesron.fullName()
```
Using <code>angular.element($0).scope()</code> you can inspect the data on scope. The angular.element can also get references to controller, injector using which you can inspect other objects.

```js
//Get controller
angular.element($0).controller()

//Getting injector and other objects
var injector = angular.element($0).injector()
injector.get('$rootScope')
injector.get('myService')
```

The other alternative is to use chrome extension [AngularJS Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk?hl=en). This will add AngularJS tab in developer Tools. It lists all scopes in the current page. But finding the correct scope for a particualr element quite difficult. The "Enable Inspector" button in batarang doesn't work that well. So far the [angular.element](https://docs.angularjs.org/api/ng/function/angular.element) is the best way to debug binding issues.


