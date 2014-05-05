---
layout: post
title: "Printing html with image and css"
date: 2014-05-03 16:54:00 +0530
comments: true
categories: javascript print
---

There are plenty of examples for printing html of an element. Most of them look like this

```js
//The simple soultion but has few problems listed below.
function printHtml(html) 
{
    var mywindow = window.open();
    mywindow.document.write('<html><head><title>My App</title>');
    mywindow.document.write('</head><body>');
    mywindow.document.write(html);
    mywindow.document.write('</body></html>');
    mywindow.print();
    mywindow.close();
    return true;
}
```
The issues with above solution:

<!-- more -->

1. The new window pop up would be blocked by browsers. Users need to enable pop up.
2. If you have external css files, you will notice the styling not applied some times.
3. If you have images(like logo), the print will be missing these images intermittently.

The first issue can be addressed by using an iframe instead of new window. The 2nd and 3rd issues are addressed by making sure print happens after page has loaded css files and images. The working solution we use in [Bahmni](http://www.bahmni.org) for printing patient registration cards looks like this

```js
var printHtml = function (html) {
    var hiddenFrame = $('<iframe style="display: none"></iframe>').appendTo('body')[0];
    hiddenFrame.contentWindow.printAndRemove = function() {
        hiddenFrame.contentWindow.print();
        $(hiddenFrame).remove();
    };
    var htmlDocument = "<!doctype html>"+
                "<html>"+
                    '<body onload="printAndRemove();">' + // Print only after document is loaded
                        html +
                    '</body>'+
                "</html>";
    var doc = hiddenFrame.contentWindow.document.open("text/html", "replace");
    doc.write(htmlDocument);
    doc.close();
};
```

For printing contents of an element you can use this

```js
//element can be found by document.querySelectorAll(slector)[0]
//or using jQuery(selector)[0]
var printElement = function (element) {
	printHtml(element.innerHTML)
};
```