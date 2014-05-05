---
layout: post
title: "Introducing f.js"
date: 2014-05-01 18:59:39 +0530
comments: true
categories: javascript f.js
---

Javascript code is quite verbose when compared to other langauges. One of the features I missed coming from the ruby world is lambdas and the <code>&:</code> sugar.

```ruby
books.map { |book| => book.title }
# or even better
books.map(&:title)
```
compare this with javascript code

```js
books.map(function(book) { return book.title })
```
<!-- more -->

Good news is ES6 might come with proposed [arrow functions](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/arrow_functions), but it might take a while before all the browsers implement this. This is where [f.js](https://github.com/endeepak/f.js) can be handy. This library will help to write redable code by writing less. You can write the above code as

```js
books.map(f.y('book => book.title'))
// or
books.map(f.x('title'))
```

[f.js](https://github.com/endeepak/f.js) supports methods, negation and includes utilities like noop and identity. Examples

```js
movies.filter(f.x('!watched'))
people.map(f.x('fullName()'))
shows.filter(f.x('!isGood()'))

//Lambda with multiple parameters
//sort movies in descending order on rating
movies.sort(f.y('(m1, m2) => m2.rating - m1.rating'));

//Noop and identity examples
books.forEach(callback || f()) //f() returns noop function
movies.filter(filterFunction || f(true)) //f(arg) returns identity function
```

The f.y(lambda) method uses same syntax as arrow functions. Once ES6 arrow functions feature is available on all browsers, write a regex to remove usage of f.y and say bye to f.js

Code & Documentation: https://github.com/endeepak/f.js
