---
layout: post
title:  "AngularJS & Dirty Checking"
date:   2014-07-12 00:00:00
permalink: /angularjs-and-dirty-checking
---

AngularJS offers two way data binding in the form of dirty checking. To sum dirty checking up quickly, whenever a watcher value is updated, all the listener events are fired.

### $scope.$watch

To register a watch/listen event in AngularJS, you do so with `$scope.$watch`. As you are doing this on a `$scope`, you can only watch the values held within that scope.

{% highlight js %}
$scope.$watch( watcherEvent, listenerEvent );
{% endhighlight %}

### Digests

The `watcherEvent` and `listenerEvent` are used in digests. In a digest, the watcher value is evaluated, and if it doesn't match the old value, the listener events are called. Take the following code for example -

{% highlight js %}
$scope.name = 'Ryan';

$scope.$watch( function( ) {
	return $scope.name;
}, function( newValue, oldValue ) ) {
	console.log(newValue, oldValue);
} );

$scope.name = 'Bryan';
{% endhighlight %}

AngularJS runs the $apply function when a model value is expected to be updated. For inputs, this means on a `change` event, etc. This runs the digests. The digest will run the watcher function (the first argument in the $scope.$watch function) and check it against the last known value for it. For example, when the $watch function is evaluated -

{% highlight js %}
//lastKnownValue will equal Ryan
newValue = watcherFunction();

console.log( newValue, lastKnownValue ); // will equal Ryan Ryan
{% endhighlight %}

As `newValue` and `lastKnownValue` equal eachother, no listener needs to be notified. However, when we update `$scope.name` -

{% highlight js %}
//lastKnownValue will equal Ryan
newValue = watcherFunction();

console.log( newValue, lastKnownValue ); // will equal Bryan Ryan
{% endhighlight %}

`newValue` and `lastKnownValue` are different, meaning that the function will continue and run the listener functions. The digest will then class this value as being dirty. Dirty means they are not equal to each other and it's unknown if they do. Whilst the value is considered dirty, it will loop until it isn't, maxing 10 loops before throwing an error. Essentially, this will be ran -

{% highlight js %}
//lastKnownValue will equal Ryan
newValue = watcherFunction();

console.log( newValue, lastKnownValue ); // will equal Bryan Ryan

lastKnownValue = newValue;

//and repeat..

//lastKnownValue will equal Bryan
newValue = watcherFunction();

console.log( newValue, lastKnownValue ); // will equal Bryan Bryan
{% endhighlight %}

Simple. This will run the second argument in the `$watch` function (the listener event) where Angular will update the appropriate model value, as well as run your listener functions.

### $apply

`$apply` evaluates a given function in the scope context. This is ran by Angular itself, so you only need to worry about it if you're doing work outside of the Angular library.

`$apply` will evaluate the function first, then immediately after run the digests associated with that scope. The basis of the function is -

{% highlight js %}
try {
	evaluateFunction();
} finally {
	$scope.$digest();
}
{% endhighlight %}

When you use `ng-model` on an text input, the directive runs `$apply` with a given function. A simple version of this would be -

{% highlight js %}
// basic version of the code behind the ng-model directive
var elementToWatch = 'name'; // the element is watching an input called "name"

element.on('change', function(){
	$scope.$apply(function(){
		$scope[elementToWatch] = element.value;
	});
});

// in the controller/directive

$scope.$watch( function() { 
	return $scope.name;
}, function( newValue, oldValue ) {
	console.log('The input was updated!');
} );
{% endhighlight %}

In the example `ng-model` directive, when the input's value is updated, the $scope value is updated to match. The digests are then ran - comparing `$scope.name` to it's previously known value (possibly `undefined` if it hasn't been set), which in return fires the listener event, which logs the fact the input has been updated to the console.

This is a huge advantage over setting getters/settings, as it allows for us to call the watch function on unknown variables and have the listener functions ran as soon as the variable exists.

###The End

This is the basics behind how dirty checking works in AngularJS. If you want to read more into this, <a href="http://teropa.info/blog/2013/11/03/make-your-own-angular-part-1-scopes-and-digest.html" target="_blank">this is an amazing article with more detail into the other functions ran by AngularJS to do with $watch, $digest and $apply.</a>