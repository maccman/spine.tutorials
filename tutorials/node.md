<div class="back"><a href="index.html">&laquo; Back to all tutorials</a></div>

#Using Spine and NodeJS

[NodeJS](http://nodejs.org/) is essentially a way of using Google's [V8 JavaScript](http://code.google.com/p/v8/) engine for running JavaScript on the server side, taking advantage of its blazing speed and evented IO. Node provides a similar environment for scripts as in Google Chrome, with the exception of a DOM. Spine has special Node support, so everything that doesn't touch the DOM will work as usual, such as `Spine.Events`, `Spine.Class` and `Spine.Model`. The only component to Spine that won't work with NodeJS is `Spine.Controller`; as it relies on the DOM so even loading controllers will cause errors. 

So why would you want to use Spine with Node? Well for a start Spine's classes and models are just as useful server side as they are client side - and you can use them to structure and modularize your code. More importantly though, it paves the way for code re-use between server and client. You could potentially use the same models, classes and libraries on both the client and server side. Finally, you can run automated unit-tests in Node, testing your client-side code. 

You can install Spine using [npm](http://npmjs.org):

    npm install spine

Spine's API in Node is exactly the same, the one difference being you first need to fetch a reference to the library using `require()`.

    var Spine = require("spine");
    
    var MyClass = Spine.Class.create({
      sameApi: "asUsual"
    });
    
#Exports

By default NodeJS encapsulates code preventing global scope pollution. If you want your client-side scripts to work with Node you'll have to explicitly expose any variables you want to be global. This is done by using the `exports` object, setting global variables as properties on it. To allow your scripts to run in either a server or client side environments, you can check to see if it exists, creating it if otherwise. 

    (function(){
      // Set exports to global context unless defined
      if (typeof exports == "undefined")
        var exports = this;
      
      // Expose MyClass
      exports.MyClass = Spine.Class.create();
    })();

#Testing with NodeJS & Jasmine

The [jasmine-node](https://github.com/mhevery/jasmine-node) project is a great way of running [Jasmine](http://github.com/pivotal/jasmine) tests with Node. First install the package using npm:

    npm install jasmine-node
    
Then you have a command line executable, `jasmine-node`, that can be used to run tests.

    jasmine-node specs
    
You'll need to pass as directory to `jasmine-node` containing all your specs. Additionally, your specs need to be named \*.spec.js to be picked up properly. Any files named \*.helper.js will be loaded in before the specs are executed. 

For example, we can create a spec called *class.spec.js* to test Spine's classes.  

    var Spine = require("spine");

    describe("Class", function(){
      var User;

      beforeEach(function(){
        User = Spine.Class.create();
      });

      it("is sane", function(){
        expect(Spine).toBeTruthy();
      });

      it("can create subclasses", function(){
        User.extend({classProperty: true});

        var Friend = User.create();
        expect(Friend).toBeTruthy();
        expect(Friend.classProperty).toBeTruthy();
      });
    });
    
And we can now run this spec using jasmine-node:

    $ jasmine-node spec/
      Started
      ..........

      Finished in 0.003 seconds
      1 test, 17 assertions, 0 failures
    
To find out more information about Jasmine's spec syntax, see [its documentation](http://github.com/pivotal/jasmine). 

#Generating applications with Spine.App

Using [Spine.App](http://github.com/maccman/spine.app) is great way of structuring your application, managing dependencies and automatically compiling [CoffeeScript](http://jashkenas.github.com/coffee-script) and [LessCSS](http://lesscss.org) files. Spine.App will also start up a server and host your application during development. 

##Installation

First step is to install the [npm](http://npmjs.org/) package. If you don't already have [npm](http://npmjs.org/) or [NodeJS](http://nodejs.org/) you'll need to install them first.

    $ npm install spine

Then we can generate the initial application structure like this:

    $ spine app my_app
    
Now we've produced a directory structure looking like:

    my_app
    my_app/app
    my_app/app/controllers
    my_app/app/models
    my_app/lib
    my_app/lib/jquery.js
    my_app/lib/spine.js
    my_app/public
    my_app/public/index.html
    my_app/public/css
    my_app/public/css/application.css
    my_app/public/images
    my_app/index.js

We can start the application by running `index.js`.
    
    $ cd my_app
    $ node index.js
    
This will boot up an [Express](http://expressjs.com) server on port [9294](http://localhost:9294). 

##Generating

You can now start generating Spine controllers and models:
    
    $ spine controller users
        app/controllers/users.coffee
    
    $ spine model user
      app/models/user.coffee
    
Any application specific code should go in the `app` folder. Otherwise, generic code, should go in the `lib` folder. 

__Any [CoffeeScript](http://jashkenas.github.com/coffee-script) or [LessCSS](http://lesscss.org) files inside the application will be automatically compiled when requested__, you don't need to worry about compiling them manually. 

##CommonJS modules

[Stitch](https://github.com/sstephenson/stitch) bundles up all your JavaScript files, enclosing them in a CommonJS wrapper. This means that scripts in the `app` folder need to be CommonJS compliant (basically exactly like normal Node scripts). In other words, to use a module you'll need to `require()` it, and you'll need to explicitly export any global variables. 

    Guide = require("controllers/guide")
    
    module.exports = Spine.Controller.create
      elements:
        "#item": "item"
      
      init ->
        this.guide = Guide.init(el: this.item)
        
Inside your HTML files, you need only require *application.js* and every module will be wrapped up and ready to be loaded. As you can see, the generated *index.html* kicks things off by instantiating *app/app.coffee* when the page loads.

    var exports = this;
    jQuery(function(){
      var App = require("app");
      exports.App = App.init({el: $("#body")});      
    });