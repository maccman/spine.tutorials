<div class="back"><a href="index.html">&laquo; Back to all tutorials</a></div>

#Patterns

This section is reproduced from the [docs](http://maccman.github.com/spine) for convenience, and shows a number of common controller patterns for binding data and rendering views. 

##The Render Pattern

The *render pattern* is a really useful way of binding models and views together. When the controller is instantiated, it adds an event listener to the relevant model, invoking a callback when the model is refreshed or changed. The callback will update `el`, usually by replacing its contents with a rendered template. 

    var Contacts = Spine.Controller.create({
      init: function(){
        Contact.bind("refresh change", this.proxy(this.render));
      },

      template: function(items){ 
        return($("#contactsTemplate").tmpl(items));
      },

      render: function(){
        this.el.html(this.template(Contact.all()));
      }
    });
    
This is a simple but blunt method for data binding, updating every element whenever a single record is changed. This is fine for uncomplicated and small lists, but you may find you need more control over individual elements, such as adding event handlers to items. This is where the *element pattern* comes in.

##The Element pattern

The element pattern essentially gives you the same functionality as the render pattern, but a lot more control. It consists of two controllers, one that controls a collection of items, and the other deals with each individual item. Let's dive right into the code to give you a good indication of how it works.

    var ContactItem = Spine.Controller.create({
      // Delegate the click event to a local handler
      events: {
        "click": "click"
      },
      
      // Ensure functions have the correct context
      proxied: ["render", "remove"],
      
      // Bind events to the record
      init: function(){
        this.item.bind("update", this.render);
        this.item.bind("destroy", this.remove);
      },

      // Render an element
      render: function(item){
        if (item) this.item = item;

        this.el.html(this.template(this.item));
        return this;
      },

      // Use a template, in this case via jQuery.tmpl.js
      template: function(items){ 
        return($("#contactsTemplate").tmpl(items));
      },

      // Called after an element is destroyed
      remove: function(){
        this.el.remove();
      },
      
      // We have fine control over events, and 
      // easy access to the record too
      click: function(){ /* ... */ }
    });

    var Contacts = Spine.Controller.create({
      proxied: ["addAll", "addOne"],

      init: function(){
        Contact.bind("refresh", this.addAll);
        Contact.bind("create",  this.addOne);
      },

      addOne: function(item){
        var contact = ContactItem.init({item: item});
        this.el.append(contact.render().el);
      },

      addAll: function(){
        Contact.each(this.addOne);
      }
    });
    
In the example above, `Contacts` has responsibility for adding records when they're initially created, and `ContactItem` has responsibility for the record's update and destroy events, re-rendering the record when necessary. Albeit more complicated, this gives us some advantages over the previous render pattern. 

For one thing, it's more performant; the list doesn't need to be re-drawn whenever a single element changes. Furthermore, we now have a lot more control over individual items. We can place event handlers, as demonstrated with the `click` callback, and manage rendering on an item by item basis.