---
layout: post
title: "Printing external html templates using AngularJS"
date: 2014-05-03 20:50:48 +0530
comments: true
categories: javascript angularjs print template
---
## Use case

In [Bahmni](http://bahmni.org) EMR we needed to support customizable html templates for printing patient regisatrtion card and other printable documents. We needed a print API which looks like

```js
//Contract
printer.print(templateUrl, data)

//This would called on clicking the print button
printer.print('/config/registrationCardTempate.html', {patient: {name: 'Ram Kumar', dateOfBirth: '1978-08-23', gender: 'M'}})
```
Sample html template

```html
<link rel="stylesheet" href="/config/registrationCard.css"/>
<img src="/config/logo.jpg"><img>
<h1>The hospital name</h1>
<table>
	<tr><td>Name</td><td ng-bind="patient.name"></td></tr>
	<tr><td>Age</td><td ng-bind="patient.dateOfBirth | age"></td></tr>
	<tr><td>Gender</td><td ng-bind="patient.gender"></td></tr>
</table>
```

## Implementation

As the app is built using angularjs, we decided to use angular as the templating engine for rendering these templates as well. This also helped to reuse filters and other templating features of angular. The implementation consists of following steps

1. Fetch the html template
2. Compile the html template with given data using angular's $complile
3. Wait for angular to complete rendering the template (Explained [here](waiting-for-angularjs-digest-cycle))
4. Print the html(Explained [here](printing-html-with-image-and-css))

The code for print function looks like this

```js
var print = function (templateUrl, data) {
    $http.get(templateUrl).success(function(template){
        var printScope = angular.extend($rootScope.$new(), data);
        var element = $compile($('<div>' + template + '</div>'))(printScope);
        var waitForRenderAndPrint = function() {
            if(printScope.$$phase || $http.pendingRequests.length) {
                $timeout(waitForRenderAndPrint);
            } else {
                printHtml(element.html());
                printScope.$destroy(); // To avoid memory leaks from scope create by $rootScope.$new()
            }
        }
        waitForRenderAndPrint();
    });
};
```
The complete solution is available on [github](https://github.com/Bhamni/openmrs-module-bahmniapps/blob/master/ui/app/common/ui-helper/printer.js).