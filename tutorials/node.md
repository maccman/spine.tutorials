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