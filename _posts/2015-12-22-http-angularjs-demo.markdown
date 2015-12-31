---
layout: post
title:  "HTTP factories in AngularJS"
date:   2015-12-22 14:35:29 -0400
categories: tutorials
---
####Source: [https://github.com/KevGary/http-angularjs-demo](https://github.com/KevGary/http-angularjs-demo)

##Overview

The following tutorial and provided source code shows how to consume an API, in this demo the Reddit API, using AngularJS through the use of a custom http factory. Factories are only instantiated once upon client load and it keeps our code clean and highly reusable.  Following the MVC pattern of app development, the controller responds to the user input by performing business logic on the model, and then renders or changes the view with the response from the model. 

##Getting Started
Follow along in the tutorial by creating an empty directory with two files named index.html and ng-app.js:
{% highlight bash %}
$ mkdir my-http-angularjs-demo
$ cd my-http-angularjs-demo
$ touch index.html ng-app.js
{% endhighlight %}

Or git clone final source code for reference and reuse:
{% highlight bash %}
$ git clone https://github.com/KevGary/http-angularjs-demo
{% endhighlight %}

##Tutorial
Let's load AngularJS and our ng-app.js file by adding all required script tags to index.html:
{% highlight html %}
<!DOCTYPE html>
<html>
<head>
  <title>HTTP AngularJS Demo</title>
</head>
<body>

  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.15/angular.min.js"></script>
  <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.5.0-beta.2/angular-route.min.js"></script>
  <script type="text/javascript" src="ng-app.js"></script>
</body>
</html>
{% endhighlight %}

Hook this up to an angular app. I've named it "httpDemo":
{% highlight html %}
<html ng-app="httpDemo">
{% endhighlight %}
And add a controller to the body of the html page:
{% highlight html %}
<body ng-controller="GlobalController">
{% endhighlight %}

Now we need to make the "httpDemo" angular app.  Initialize it in ng-app.js:
{% highlight javascript %}
var app = angular.module('httpDemo', ['ngRoute']);
{% endhighlight %}
And add the controller to ng-app.js as well:
{% highlight javascript %}
app.controller('GlobalController', function ($scope) {

});
{% endhighlight %}
 
In order for us to make http requests in AngularJS, we will make use of two angular services, [$http](https://docs.angularjs.org/api/ng/service/$http) and [$q](https://docs.angularjs.org/api/ng/service/$q).  $http allows us to make AJAX calls, and $q allows us to perform promise based operations.  In the next couple examples, we will be making an AJAX GET request to retrieve the data sent down by reddit when a user would search for 'ferrari' on [https://www.reddit.com/](https://www.reddit.com/).

__THE WRONG WAY__ would be to inject $http into the controller and perform the AJAX calls inside the controller: 
{% highlight javascript %}
app.controller('GlobalController', function ($scope, $http, $q) {
  $scope.get = function () {
    $http.get('http://www.reddit.com/search.json?q=ferrari')
      .then(function (response) {
        console.log(response);
        $scope.result = response;
      })
  }
});
{% endhighlight %}
##### __Note__: .then(...) is from the $q service and is not required to use $http
__THE RIGHT WAY__. We create a factory and move the dependency injection, $http, and business logic code to said factory, then inject our factory into the controller and call our factory method.
{% highlight javascript %}
app.controller('GlobalController', function ($scope, $q, httpFactory) {
  $scope.get = function() {
    httpFactory.redditGet()
      .then(function (response) {
        console.log(response);
        $scope.result = response;
      })
  }
});      

app.factory('httpFactory', function ($http) {
  return {
    redditGet: redditGet
  }
  function redditGet () {
    return $http.get('http://www.reddit.com/search.json?q=ferrari')
  }
});
{% endhighlight %}
Awesome! We now have a http factory with a GET method to the reddit API we can use in any controller, as many times as we want.  This is a far superior approach than writing out the same AJAX calls everytime we want to use one.

Now, lets add a text input to the view above our search button and search reddit for whatever the user wants:
{% highlight html %}
<input type="text" ng-model="searchValue">
<button ng-click="get(searchValue)">search</button>
{% endhighlight %}
And update our ng-app.js file:
{% highlight javascript %}
app.controller('GlobalController', function ($scope, $q, httpFactory) {
  $scope.get = function(searchValue) {
    httpFactory.redditGet(searchValue)
      .then(function (response) {
        console.log(response);
        $scope.result = response;
      })
  }
});      

app.factory('httpFactory', function ($http) {
  return {
    redditGet: redditGet
  }
  function redditGet (searchValue) {
    return $http.get('http://www.reddit.com/search.json?q=' + String(searchValue))
  }
});
{% endhighlight %}
Now we are making an AJAX request to the reddit api with our user's text input.

##Conclusion
In this demo, we built out a simple AngularJS front end that consumes the Reddit API through using a http factory.  To detail our MVC design- the 'ng-click=get(search)' (index.html:11) is the user input from the view index.html), the controller 'GlobalController' receives the input from the view and calls the httpFactory where our data model, json data from reddit, is retrieved.  Our controller then applies the gathered data from the model to the view with $scope.result (ng-app.js:7), displayed using angular syntax, {% raw %}{{result}}{% endraw %} (index.html:12).

The final production code, also found at [https://github.com/KevGary/http-angularjs-demo](https://github.com/KevGary/http-angularjs-demo):

###index.html:
{% highlight html %}
<!DOCTYPE html>
<html ng-app="httpDemo">
<head>
  <title>HTTP AngularJS Demo</title>
  <link rel="stylesheet" type="text/css" href="main.css">
  <link rel="stylesheet" href="https://ajax.googleapis.com/ajax/libs/angular_material/0.11.2/angular-material.min.css">
</head>
<body ng-controller="GlobalController">
  <input type="text" ng-model="searchValue">
  <button ng-click="get(searchValue)">search</button>
  <div class="result">{% raw %}{{result}}{% endraw %}</div>

  <script type="text/javascript" src="https://code.jquery.com/jquery-2.1.4.min.js"></script>
  <script type="text/javascript" src="https://code.jquery.com/ui/1.11.4/jquery-ui.min.js"></script>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.15/angular.min.js"></script>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.15/angular-aria.min.js"></script>
  <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.5.0-beta.2/angular-route.min.js"></script>
  <script type="text/javascript" src="ng-app.js"></script>
</body>
</html>
{% endhighlight %}

###ng-app.js:
{% highlight javascript %}
var app = angular.module('httpDemo', ['ngRoute']);

app.controller('GlobalController', function ($scope, $q, httpFactory) {
  $scope.get = function(searchValue) {
    httpFactory.redditGet(searchValue)
      .then(function (response) {
        $scope.result = response;
      })
  }
});      

app.factory('httpFactory', function ($http) {
  return {
    redditGet: redditGet
  }
  function redditGet (searchValue) {
    return $http.get('http://www.reddit.com/search.json?q=' + String(searchValue))
  }
});
{% endhighlight %}