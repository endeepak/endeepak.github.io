---
layout: post
title: "Python's elegance in dynamic methods compared to ruby"
date: 2014-12-27 16:48:17 +0530
comments: true
categories: ruby python metaprogramming
---

I have been a ruby fanboy for a long time because of its expressiveness and elegance in defining dsls. One of the scary thing in ruby is, when you implement `method_missing` you need to make sure to implement `respond_to_missing?`, otherwise [bad things will happen to you](http://robots.thoughtbot.com/always-define-respond-to-missing-when-overriding). The below ruby example shows minimal parts recomonded for providing dynamic methods

<!-- More -->

```rb
class Foo
	def method_missing(method_name, *args, &block)
		if method_name.to_s.start_with?('bar_')
			#do_some_thing...
	end

	# You should define this for your good
	# Read http://robots.thoughtbot.com/always-define-respond-to-missing-when-overriding for more details
	def respond_to_missing?(method_name)
		method_name.to_s.start_with?('bar_') || super
	end
end

# So that following will work consistently
foo = Foo.new
foo.bar_qux
foo.bar_qux() 
foo.respond_to?(:bar_qux)
foo.method(:bar_qux)
```

The same can be achived in python using `__getattr__` with lesser code.

```py
class Foo(object):
	def __getattr__(self, attr):
		if attr.startswith('bar_'):
			#return method or an attribute..

# Following will work consistently
foo = Foo()
foo.bar_qux
foo.bar_qux() 
hasattr(foo, 'bar_qux')
```

Eventhough it isn't a lot of code in ruby, people can forget to implement both methods or implement them differently by mistake leading to tricky bugs and higher maintainance cost. 

### Can ruby have a similar implementation?

May be not. It is because of the fact that ruby functions are not first class objects which can be retuned in a single method_missing hook. Also ruby's syntax of calling a method without paranthesis(i.e. `foo.bar_qux` is same as `foo.bar_qux()`) makes it hard to treat functions as callable objects.



