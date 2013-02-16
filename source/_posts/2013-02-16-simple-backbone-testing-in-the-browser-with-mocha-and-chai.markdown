---
layout: post
title: "Simple Javascript testing in the browser with Mocha"
date: 2013-02-16 17:53
comments: true
categories: 
---
[Mocha](http://visionmedia.github.com/mocha/) is a javascript test runner for Node and the browser. There's plenty of documentation on how to set up Mocha with Node, I'm going to outline how easy it is to use Mocha in the browser too.
First we need to load the config file and then I'm using [RequireJS](http://requirejs.org/) to load the application `main.js`. I'm using the Backbone framework here, but this will work with any application.
``` html index.html
<html> 
<head>
    <title>Welcome!/title>
</head>
<body>
...
</script>
<script src="js/config.js"></script>
<script data-main="main" type="text/javascript" src="js/libs/require.min.js"></script>
</body>
</html>

```
By separating out the config, we can use the same config file for the main application and the testing, which helps keep out path aliases consistent. I'm going to use the [Chai](http://chaijs.com/) testing library, which allows TDD or BDD style tests to be written using Mocha as a test runner. Mocha doesn't export a module in the AMD style, but adding a simple shim will allow us to use it with RequireJS. We expect [mocha.js](https://github.com/visionmedia/mocha/blob/master/mocha.js) and [chai.js](http://chaijs.com/chai.js) to be in the `js/libs/` directory

``` javascript js/config.js
var require = {
    baseUrl : "/js",
    paths   : {
        underscore       : 'libs/lodash.min',
        jquery           : 'libs/jquery.min',
        backbone         : 'libs/backbone.min',
        templates        : '../templates',
        mustache         : "libs/mustache",
        mocha            : "libs/mocha",
        chai             : "libs/chai"
    },
    shim    : {
        'backbone'       : {
            deps    : ['underscore', 'jquery'],
            exports : 'Backbone'
        },
        'mocha'         : {
           exports : 'mocha'
        }
    }};

```
Our application `main.js` doesn't need to do anything special to use this config file, by using the variable `require` RequireJS will take care of this for us. So, for example, our basic Backbone application could look like this:

``` javascript js/main.js
require([
    'router',
    'views/Main'
], function (Router, MainView) {
    new MainView().render()
    Router.initialize();
});

```
Now we need to define our testing page, test.html. As before, we load in the config file and use RequireJS to load in `test.js`. We also have a div for mocha to fill with the test results and a stylesheet. This can be found [here](https://github.com/visionmedia/mocha/blob/master/mocha.css).
``` html test.html
<!DOCTYPE html>
<html>
<head>
    <title>Mocha Tests</title>
    <link rel="stylesheet" href="css/mocha.css"/>
</head>
<body>
<div id="mocha"></div>
<script src="js/config.js"></script>
<script data-main="test" type="text/javascript" src="js/libs/require.min.js"></script>
</body>
</html>
```
Our main testing application is very simple, we just load in Chai and Mocha. We set some aliases for Chai variables, allowing us to use functions like `should.exist(object)`, and then tell Mocha we want to define tests in BDD style. Then we can just use RequireJS to load in each test and run them all at once.
``` javascript js/test.js
require([
    'chai',
    'mocha'
], function (chai, mocha) {
    assert = chai.assert;
    should = chai.should();
    expect = chai.expect;

    mocha.setup('bdd')

    require(['tests/person', 'tests/car'], function(){
        mocha.run();
    });
});
```
Each test is a AMD module, we can load in any AMD modules from our application such as Models, Views or Controllers from Backbone. We then define tests in the Chai BDD style.
``` javascript tests/car.js
define([models/car],function(Car){

    describe('Car', function(){
        it('should have a speed of 0 on creation', function() {
            new Car().speed.should.equal(0);
        });

        it('should allow a make to be set on creation', function() {
            new Car('Ford').make.should.equal('Ford');
        });
    });

});
```
We can then simply navigate to our test.html page to view the results of each test.