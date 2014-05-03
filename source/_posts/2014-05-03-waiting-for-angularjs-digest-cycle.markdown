---
layout: post
title: "Waiting for AngularJS digest cycle"
date: 2014-05-03 17:43:34 +0530
comments: true
categories: angularjs javascript
---

The million dollar question most of the AngularJS users(devs) end up asking is

* How to wait for angular digest cycle to be completed?  _or in simple terms_ 
* How to wait for browser to complete rendering the view with angular bindings? 

AngularJS does not raise any event to notify this. The suggested simple solution is to use $timeout to queue your work to be run after current digest cycle (also waits for DOM renedering to be completed by the browser).

```js
$timeout(function(){
	//the code which needs to run after dom rendering
})
```

The above solution works only for views which don't have ng-include or directives with template url. In this case you have to wait for all the templates to be loaded(async) and then run your code. This can be achived by waiting for [$http.pendingRequests](https://docs.angularjs.org/api/ng/service/$http#pendingRequests) to be zero. The enhanced solution is
 
```js
var waitForRenderAndDoSomething = function() {
	if($http.pendingRequests.length > 0) {
	    $timeout(waitForRenderAndDoSomething); // Wait for all templates to be loaded
	} else {
		//the code which needs to run after dom rendering
	}
}
$timeout(waitForRenderAndDoSomething); // Waits for first digest cycle
```

Hopefully AngularJS will come up with a easier solution in future releases.

####Note

The [$http.pendingRequests](https://docs.angularjs.org/api/ng/service/$http#pendingRequests) supposed to be used for debugging purpose only. If angular team decides to remove this, you can implement the same using http interceptors as suggested in this [link](http://stackoverflow.com/a/20062899/69362). 


