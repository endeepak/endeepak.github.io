---
layout: post
title: "Spinners in single page apps"
date: 2014-05-11 16:55:23 +0530
comments: true
categories: javascript SPA overlay spinner
---

One of the good practices in single page apps, is to show spinners (overlay + loading icon) while retrieving data or saving asynchronously. Often people tend to add a generic interceptor to show spinner for all ajax calls. In any mid size app, you will realize that the assumption **"Show an overlay for all ajax call"** is incorrect. Few examples

* An [autocomplete](http://jqueryui.com/autocomplete/) input box fetching reote data
* An infininte scroll fetching more data as the use scrolls down the list.

<!-- More -->

To solve this, one might add an option in interceptor to not show spinner for certain calls. This leads to complicated code due to initial wrong assumption. A better soultion is to have simple reusable code to show/hide spinner and use it explicitly for calls which need spinner.

If you are using a library which returns a promise for ajax call(or object like xhr returned by jQuery.ajax), the API and implementation would look like this.


```js Api should be simple as
spinner.forPromise($http.get('/items')) //AngularJS

spinner.forPromise($.ajax('/items')) //jQuery
```

```js Simple Spinner Implementation in AngularJS
myModule.factory('spinner', function () {
	var forPromise = function(promise) {
	    $('#overlay').show();
	    promise['finally'](function(){ //use promise.always(hide) in jQuery
	    	$('#overlay').hide();
	    }); 
	};

	return { forPromise: forPromise }
 )
```

If you have a multiple components of the page using same spinner, we need to enhance the the code to make sure spinner is hidden only after both components have completed async calls. This can be implemented by keeping spinner count as shown below.


```js Muliple Call Supported Spinner Implementation in AngularJS
myModule.factory('spinner', function () {
	var showCount = 0;

	var show = function () {
		showCount++;
	    $('#overlay').show();
	}

	var hide = function () {
		showCount--;
	    if(showCount === 0) {
		    $('#overlay').hide();
	    }
	}

	var forPromise = function(promise) {
	    show();
	    promise['finally'](hide); //use promise.always(hide) in jQuery
	};

	return { forPromise: forPromise };
 )
```

In [Bahmni](http://www.bahmni.org), we use a spinner with animation which can be found [here](https://github.com/Bhamni/openmrs-module-bahmniapps/blob/master/ui/app/common/ui-helper/spinner.js).

