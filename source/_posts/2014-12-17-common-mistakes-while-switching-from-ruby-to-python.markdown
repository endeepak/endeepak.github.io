---
layout: post
title: "Common mistakes while switching from ruby to python"
date: 2014-12-17 13:50:24 +0530
comments: true
categories: ruby python tdd
---

This is more of a note to self sort of post to talk about silly mistakes which can take you down quietly when you switch from ruby to python.


## Problem 1 : The method which does nothing

```rb person.rb (Ruby)
class Person
	def initialize(name)
		@name = name
	end
	
	def say_hello
		puts "#{@name} says hello"
	end
end

bob = Person.new('Bob')
bob.say_hello # Bob says hello
```

```py person.py (Python)
class Person(object):
	def __init__(self, name):
		self.name = name
	
	def say_hello(self):
		print("%s says hello" % self.name)


bob = Person('Bob')
bob.say_hello # Danger! Danger! Method won't be called

# After breaking your head for a while. Correct it to
bob.say_hello() # Well remember the good old pranthesis to call function?
```

## Problem 2 : The method which returns wrong value

```rb calculator.rb (Ruby)
def add(x, y)
	x + y
end

add(2, 3) # returns 5
```

```py calculator.py (Python)
def add(x, y):
	x + y

add(2, 3) # Danger! Danger! Returns nil 

# After breaking your head for a while. Correct it to
def add(x, y):
	return x + y
```

## Solution

Test Driven Development
