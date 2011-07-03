<div class="back"><a href="index.html">&laquo; Back to all tutorials</a></div>

#Templating

Templates are a core part of rendering views in JavaScript applications. In a nutshell, templates contain variables and simple flow logic, which you can parse with a templating library, interpolating data with template variables and generating raw HTML. 

To that extent, templating in JavaScript is very similar to templating in any other language, such as Ruby's ERB or Python's string interoperation. You just need to choose a templating library and stick with it, as the templating syntax differs slightly between them.

One good choice is [Mustache](http://mustache.github.com), a JavaScript templating library that also has lots of implementations in other languages. 

However, the one I personally use and am going to demonstrate below is [jQuery.tmpl](http://api.jquery.com/jquery.tmpl), a templating plugin for jQuery. The general concepts should be the same whichever library you choose; you'll just have to look at the documentation for the specific syntax.

##Rendering templates

jQuery.tmpl has one main function, `jQuery.tmpl()`, which you can pass a template and some data. It renders the template and returns raw HTML which you can then append to the document. If the data is an array, the template is rendered once for every data item in the array, otherwise a single template is rendered.

    var object = {
      url: "http://example.com",
      getName: function(){ return "James"; }
    };

    var template = '<li><a href="${url}">${getName()}</a></li>';
    
    var element = jQuery.tmpl(template, object);
    // Produces: <li><a href="http://example.com">James</a></li>

    $("body").append(element);

So, you can see we're interpolating variables using the `${}` syntax. Whatever is inside the brackets get executed in the context of the object passed to `jQuery.tmpl()`, regardless if it's a property or function. 

##Flow control

However, templates are much more powerful then mere interpolation. Most templating libraries have advanced features like conditional flow and iteration. You can control flow by using `if` and `else` statements, the same as with pure JavaScript. The difference here is that we need to wrap the keyword with double brackets, so the templating engine can pick them up.

    {{if url}}
      ${url}
    {{/if}}

The `if` block will be executed if the specified attribute value doesn't evaluate to `false`, `0`, `null` or `undefined`. As you can see, the block is closed with a `{{/if}}`, don't forget to include that! 
A common pattern is to display a message when an array, say of chat messages, is empty. 

    {{if messages.length}}
      <!-- Display messages... -->
    {{else}}
      <p>Sorry, there are no messages</p>
    {{/if}}

Iteration is something no templating library can afford to be without. With JS templating libraries, you can iterate over any JavaScript type, be that an Object or Array, using the `{{each}}` keyword. If you pass an Object to `{{each}}`, it'll iterate a block over the object's properties. Likewise passing an array, results in the block iterating over every index in the array.

When inside the block, you can access the value currently being iterated over using the `$value` variable. Displaying the value is the same as the interpolation example above, using `${$value}`. So, if we take an object like this:

    var object = {
      foo: "bar",
      messages: ["Hi there", "Foo bar"]
    };
    
Then use the following template to iterate through the `messages` array, displaying each message. Additionally, the current iteration's index is also exposed using the `$index` variable.

    <ul>
      {{each messages}}
        <li>${$index + 1}: <em>${$value}</em></li>
      {{/each}}
    </ul>

That pretty much covers the jQuery.tmpl templating API - you can see it's very straightforward. As I mentioned earlier, most of the alternative templating libraries have a similar API, although many offer more advanced features, such as lambdas, partials and comments.

##Loading in templates

So far we've been using templates defined inline in JavaScript, which is far from ideal. We want to keep our views and controller separate to conform to MVC, besides that's just plain ugly. However fortunately there's an alternative. We can define the templates as custom `<script>` elements in the page, like so:

    <script type="text/template" charset="utf-8" id="contact-template">
      <li>${name}</li>
    </script>
    
The browser will just interpret the template as text, and won't try to render it. However, the script elements HTML will still be available. We can now render templates by fetching the template source straight from the `#contact-template` script.

    var elements = $.tmpl($("#contact-template").html(), Contact.all());
    
That syntax is a bit convoluted, and jQuery.tmpl provides a shortcut, like so:
    
    var elements = $("#contact-template").tmpl(Contact.all());
    $("#view").html(elements);
    
So far so good, we've now solved the problem of having templates inline in our JavaScript files. However, we still have to put them all inline in the page (`<script src='foo'>` doesn't work). One way round this is by loading templates in with Ajax, and appending them to the page. There are a lot of loader libraries, like [Require.js](http://requirejs.org), which will do this for you. However, let's create a custom jQuery plugin to show you the basics:
    
    (function($){
      
      var srcToName = function(name){
        return name
        .replace(/\.\w+?$/, "")
        .replace(/(\/|_)/, "-");
      };
    
      var appendTemplate = function(src, html){
        var template = $("<script />");
        template.attr({
          type: "text/x-template",
          id:   srcToName(src) + "-template"
        });
        template.html(html);
        $("head").append(template);
      };
    
      $.loadTemplate = function(src, name){
        $.ajax({url: src})
          .success(function(result){
            appendTemplate(name || src, result);
          });
        return this;
      };
    
    })(jQuery);
    
    
When `$.loadTemplate()` is called, the template is fetched from the server with Ajax and downloaded. Finally, when the request has finished, the template is appended to the page in much the same fashion as our manual implementation earlier. So, on startup, our application can remotely load all the templates it needs, which can then be used to render views.

    $.loadTemplate("/templates/contacts.tmpl", "contacts");

##Finding template records

The last part of this tutorial deals with finding the original piece of data a rendered template is associated with. In other words, given a rendered contact element, we want to find the original `Contact` instance that was used to render the template. This is very useful in *click* events, for example, where you want to find out which record a user clicked on. 

To achieve this, jQuery.tmpl provides the `tmplItem()` function, which returns an object full of template information. You can call this on a rendered, template, for example:

    var elements = $("#contact-template").tmpl(Contact.all());
    
    var templateData = elements.first().tmplItem();
    
We won't be needing most of the information returned by `tmplItem()`, but the `data` property is the one piece of information we do need. `data` will be set to the original object used by the template to render the element.

    var elements = $("#contact-template").tmpl(Contact.first());
    var contact  = elements.tmplItem().data;
    
I often shortcut this process with some extensions to jQuery.tmpl, shown below.
    
    (function($){
      $.fn.item = function(){
        var item = $(this).tmplItem().data;
        return($.isFunction(item.reload) ? item.reload() : null);
      };

      $.fn.forItem = function(item){
        return this.filter(function(){
          var compare = $(this).tmplItem().data;
          if (item.eql && item.eql(compare) || item === compare)
            return true;
        });
      };
    })(jQuery);
    
`$.fn.item` can be used for finding the `Spine.Model` instance used when generating the template, and `$.fn.forItem` is useful for filtering elements to match a certain record.

    var elements = $("#contact-template").tmpl(Contact.first());
    var contact  = elements.item();
