---
layout: post
title: "Keep your models independent of angular"
date: 2014-05-18 16:38:01 +0530
comments: true
categories: angularjs javascript refactor
---

> If you are using MVC framework you need to make sure controllers are very thin and domain logic lies in small, framework independent, composable models - Wise People

In AngularJS, you need to make sure lot of data is not defined directly on $scope and domain logic is not dependent on angular's digest cycle. If follow this mantra, unit testing the models would be a lot simpler which in turn is a indicates that your code in is good shape.

Alright lets get to some code. Let's consider a simple example where we have a form to capture person's information such as firstName, lastName, age or dateOfBirth. The age or dateOfBirth should be auto populated based on its counter part.

```js Bad code
//In controller
$scope.firstName = ""
$scope.lastName = ""
$scope.fullName = function() { 
	return $scope.firstName + ' ' + $scope.lastName; 
}
$scope.age = ""
$scope.dateOfBirth = ""
$scope.$watch('age', setDateOfBirthBasedOnAge);
$scope.$watch('dateOfBirth', setAgeBasedOnDateOfBirth);
```

If you want to test the logic to compute fullName or age<->dateOfBirth logic, you will have to use angular-mock and inject $scope in your tests. This leads to lot of unnecessary boilerplate code. Lets look at how to refactor this code.

#### Refactor 1 : Introduce a model

```js
//Model
var Person = function() {
	this.firstName = ""
	this.lastName = ""
	this.fullName = function() { 
		return this.firstName + ' ' + this.lastName; 
	}
	this.age = ""
	this.dateOfBirth = ""
}

//In controller
$scope.person = new Person();
$scope.$watch('person.age', setDateOfBirthBasedOnAge);
$scope.$watch('person.dateOfBirth', setAgeBasedOnDateOfBirth);
```
Now you can simply instante a person object and test the fullName method.

#### Refactor 2 : Remove dependency on $scope.$watch

In this step we will use [Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty), ES5 API which works on [most of the browsers](http://kangax.github.io/compat-table/es5/#Object.defineProperty)

```js
//Model
var Person = function() {
	..		//lastName, firstName, fullName code remains same as above
	var _age, _dateOfBirth; // Private fields
	
	Object.defineProperty(this, 'age', {
		get: function() { return _age; }
		set: fubction(value) {
			_age = value;
			setDateOfBirthBasedOnAge();
		}
	});

	Object.defineProperty(this, 'dateOfBirth', {
		get: function() { return _dateOfBirth; }
		set: fubction(value) {
			_dateOfBirth = value;
			setAgeBasedOnDateOfBirth();
		}
	});
}

//In controller
$scope.person = new Person();
```
After this step, your domain logic can be tested without having to use angular-mock or injectors etc.

### Conclusion

One of the boasted feature in AngualrJS is using POJOs for data binding, compared to using special observables or models  in knockout, ember etc. If this is one of the reason you are using AngularJS, it is very important to make sure your domain logic doesn't leak into controllers.