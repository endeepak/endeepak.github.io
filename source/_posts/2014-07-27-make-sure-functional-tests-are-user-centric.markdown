---
layout: post
title: "Make sure functional tests are user centric"
date: 2014-07-27 16:43:49 +0530
comments: true
categories: javascript ajax SPA functional-tests
---

In modern day web applications, most of the data is fetched and saved using ajax calls and the mark up is generated on client side by compiling static HTML templates. The functional tests need to make sure assertions wait till completion of ajax call.

Couple of bad ways to achieve this is by using `sleep(xSeconds)` or `wait_for_ajax` like this

<!-- More -->

``` ruby
fiil_in 'name', :with => 'foo' # you might be using a page object pattern in real code
click_on 'Save' 
wait_for_ajax
expect(find('#number_of_people').text).to be(10)

# The implementation of wait_for_ajax which depends on jQuery.active
def wait_for_ajax
	wait_until { page.evaluate_script('jQuery.active') == 0 }
end
```

The `sleep(xSeconds)` makes your tests indeterministic and `wait_for_ajax` makes it dependent on javascript framework used in application. 

## Solution 1 : Choose a good framework

Frameworks like capybara have implicit wait mechanism which eliminates the need for `wait_for_ajax`

``` ruby
click_on 'Save' 
expect(find('#number_of_people').to have_content(10) # Implicit wait
```

But what happens when

* You are not using framework like capybara *or*
* The assertion is done on data saved in a database or file system

 In these cases, don't go back to `wait_for_ajax` or `sleep` solutions. Instead of depending on technical details of the app, you need to think in terms of

> How does the user know app is done loading or saving data?

This will give you a hint on any missing usability requirements in the app. 

## Solution 2 : Think like a user

When you think like user, you will realize there must be a visual clue in the app to indicate the progress. The functional tests should also depend on this indicator (like a spinner, overlay etc) to figure out when to assert on data.

``` ruby
# click
wait_for_completion
# assert

# Example implementation of wait_for_completion
def wait_for_completion
	wait_until { page.find('#loading-indicator').visible? == false }
end
```

When your tests user centric, they provide valuable feedback on the user experience.