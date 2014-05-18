---
layout: post
title: "Javascript functions in JSON configuration"
date: 2014-05-18 15:20:32 +0530
comments: true
categories: javascript JSON
---

There is no native support for defining functions in JSON. Commonly used approach is to define function as string and use <code>eval()</code> or <code>new Function()</code> to contruct the function. The basic difference between these two are

* <code>eval()</code> works within the current execution scope. It can access or modify local variables.
* <code>new Function()</code> runs in a separate scope. It cannot access or modify local variables.

These samples show how the json would differ in these two cases

<!-- More -->

```js Using eval()
// JSON config
{
	section: "Additional Details",
	showIf: "function(context) { return context.person.age > 60; }"
}

//This can parsed in javascript as
var showIf = eval('(' + config.showIf + ')')
var shouldShowThisSection = showIf({person: personData}));
```

```js Using new Function()
// JSON config
{
	section: "Additional Details",
	showIf: "return context.person.age > 60"
}

//This can parsed in javascript as
var showIf = new Function('context', config.showIf)
var shouldShowThisSection = showIf({person: personData}));
```

You can use either based on the use case in your application. In [bahmni](http://bahmni.org), we went with <code>new Function()</code> for couple of reasons

* We did not want config code to modify variables in application execution scope by mistake.
* We wanted to control the function signature such as number of parameters and its name to keep it simple and less error prone.

If you prefer using <code>eval</code> syntax, try [vkiryukhin/jsonfn](https://github.com/vkiryukhin/jsonfn).

#### Multi line functions

The above examples work fine for sinle line expressions. If you need multiple line functions, you need to tweak it a bit. Firstly, JSON does not support multiline string. The work around is to define an array of strings as shown below.

```js Multi line functions parsed using new Function()
// JSON config
{
	section: "Additional Details",
	showIf: ["if(context.person.gender === 'M' && context.person.age > 60)",
				"return true;", 
			 "else if(context.person.gender === 'F' && context.person.age > 55)",
				"return true;", 
			"else",
				"return false;"] 
}

//This can parsed in javascript as
var showIf = new Function('context', config.showIf.join("\n"))
var shouldShowThisSection = showIf({person: personData}));
```

#### Performance

There is not much difference in performance when you define a function using <code>function() {}</code> expression <code>eval('function() {}')</code> or <code>new Function()</code>. Have a look at this [benchmark](http://jsperf.com/function-vs-new-function-vs-eval-function) using jsperf.