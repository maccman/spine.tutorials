<div class="back"><a href="index.html">&laquo; Back to all tutorials</a></div>

#Layout

Aside from the simplest of applications, the vast majority have need of multiple views and layouts. Unfortunately, in our JavaScript web applications, we don't have the luxury of putting views on separate pages, since there's no page reloading. Instead, we need to place all the views inside the page, showing or hiding them as appropriate. 

[Spine](http://maccman.github.com/spine) has a couple of utilities to help you do this, *spine.manager.js*, *spine.tabs.js* and *spine.list.js*.

##Manager

`Spine.Manager` is essentially a state machine for controllers. Controllers are added to the manager, and the manager controls state for its associated controllers. A controller's state is either *active* or *deactive*. `Spine.Manager` ensures that only one controller is active at any one time; when another controller is 'activated', all other controllers will be deactivated. 

To use the `Manager` class, we first we need to include the *spine.manager.js* script, which you can find in [Spine's repository](https://github.com/maccman/spine/raw/master/lib/spine.manager.js). Let's demonstrate this by creating a couple of controllers, adding them to a `Manager` instance, and change their state. 

    var Users  = Spine.Controller.create();
    var Groups = Spine.Controller.create();
    
    var users  = Users.init();
    var groups = Groups.init();
    
    Spine.Manager.init(users, groups);
    
    users.active();
    assert( users.isActive() );
    assert( !groups.isActive() );
    
    groups.active();
    assert( groups.isActive() );
    assert( !users.isActive() );
    
So, as you can above, Spine gives your controllers an `isActive()` function which returns a boolean indicating whether the controller's state is *active*. In addition, we now have a `active()` function which we can call on controllers to activate them. 

The manager ensures that only one controller in its set is activated at any one time. When the `users` controller is activated (by calling `active()`), the groups controller will be deactivated, and vice versa. 

When a controller is activated, it's `activate()` function will be called. Likewise, when it's deactivated it's `deactivate()` function will be called. These are already implemented, but you can override them to add custom behavior.

By default, the controller's `activate()` function adds an *active* class onto the controller's element (`el`). The deactivate function removes this class.

    // ...
    activate: function(){
      this.el.addClass("active");
      return this;
    },

    deactivate: function(){
      this.el.removeClass("active");
      return this;
    }
    
Hopefully you're starting to see how we can apply this to showing and hiding views. Take [Holla](http://github.com/maccman/holla) for instance. It has a settings view, and a channel view - both associated with controllers. They need to be displayed independently, and changed using the sidebar menu. 

The simplest way of achieving this is by adding them both to a `Manager`, which will ensure that only one has an *active* class at any one time. Then, with CSS, we can hide views without the *active* class.

    #views > *:not(.active) {
      display: none !important;
    }
    
When the two controllers are activated by the sidebar menu, the *active* class switches and the relevant view is shown.

##Tabs

Spine provides a simple abstraction on top of it's `Spine.Manager` class for tab integration, `Spine.Tabs`. The class lets you associate a 'tab' with a controller, and when the tab is clicked the controller is activated. 

The first step is to include [spine.tabs.js](https://github.com/maccman/spine/raw/master/lib/spine.tabs.js). `Spine.Tabs` relies on either [jQuery](http://jquery.com) or Zepto, so you'll need to include one or the other in the page too, as well as [spine.manager.js](https://github.com/maccman/spine/raw/master/lib/spine.tabs.js).

    <script src="jquery.js" type="text/javascript" charset="utf-8"></script>
    <script src="spine.js" type="text/javascript" charset="utf-8"></script>
    <script src="spine.manager.js" type="text/javascript" charset="utf-8"></script>
    <script src="spine.tabs.js" type="text/javascript" charset="utf-8"></script>
    
Now all the required libraries are loaded, we can start creating our tabs and associating them with controllers. Spine has no restrictions on the tab markup, except that there's a `data-name` attribute present on the tab. Usually one would create a list of `<li>` elements, and float them like so:
  
    <style type="text/css" media="screen">
      .tabs {
        list-style: none;
        margin: 0; padding: 0;
        overflow: hidden;
      }

      .tabs li {
        float: left;
        padding: 5px 10px;
      }
    </style>
    
    <ul class="tabs">
      <li data-name="users">Users</li>
      <li data-name="groups">Groups</li>      
      <!-- ... -->
    </ul>
    
So we now have a horizontal list of tabs, let's bind them up with `Spine.Tabs` and make them interactive!
    
    var users  = Users.init({el: $("#layout1 .users")});
    var groups = Groups.init({el: $("#layout1 .groups")});
    
    Spine.Manager.init(users, groups);
    
    var tabs = Spine.Tabs.init({el: $("#layout1 .tabs")});
    tabs.connect("users", users);
    tabs.connect("groups", groups);
    
    // Select first tab
    tabs.render();
    
As you can see, we've added two controllers (`users`, `groups`) to a `Spine.Manager` instance, as explained in the first section of this tutorial. Then we're creating a `Spine.Tabs` instance, passing in the tabs `<ul>` element. Finally we're associating tabs with specific controllers using the `connect()` function. Connect takes two arguments: the name of the tab (i.e. the `data-name` attribute), and a controller. 
  
That's the full extent of the code needed to implement tabs. `Spine.Tabs` will make sure any click events on a tab activate the controller the tab is associated with, showing and hiding views as necessary. The class will also add a class of *active* onto the currently activated tab, so you can easily style it to show the user which tab is currently selected.

    .tabs li.active {
      border: 1px solid #000;
    }
    
The full code (minus styling) is shown below:

    <div id="layout1">
      <div class="tabs">
        <ul>
          <li data-name="users">Users</li>
          <li data-name="groups">Groups</li>
        </ul>
      </div>
  
      <div class="views">
        <div class="users">
          <h3>Users</h3>
          <!-- ... -->
        </div>
    
        <div class="groups">
          <h3>Groups</h3>
          <!-- ... -->
        </div>
    </div>

    <script type="text/javascript" charset="utf-8">
      jQuery(function($){
        var Users  = Spine.Controller.create();
        var Groups = Spine.Controller.create();
        
        var users  = Users.init({el: $("#layout1 .users")});
        var groups = Groups.init({el: $("#layout1 .groups")});
    
        var manager = Spine.Manager.init();
        manager.add(users, groups);
    
        var tabs = Spine.Tabs.init({el: $("#layout1 .tabs")});
        tabs.connect("users", users);
        tabs.connect("groups", groups);
        
        tabs.render();
      });
    </script>

##List

The `Spine.List` class is basically for a dynamic set of tabs populated by model data. For example, the [Spine.Contacts](https://github.com/maccman/spine.contacts) application uses `Spine.List` to render the list of contacts in the sidebar and changing the currently selected contact when clicked.

![Spine Contacts](https://lh5.googleusercontent.com/_IH1OempnqUc/TZpgYfnlUBI/AAAAAAAABKg/UYLhdmoc15o/s500/contacts.png)

As always, the first step is is to include the required libraries. We're going to need templating for rendering our list, and in this example we'll be using [jQuery.tmpl](https://github.com/jquery/jquery-tmpl). We also need to include [spine.tmpl.js](https://github.com/maccman/spine/raw/master/lib/spine.tmpl.js) which includes some templating utilities. Lastly we obviously need to include [spine.list.js](https://github.com/maccman/spine/raw/master/lib/spine.list.js),

    <script src="jquery.js" type="text/javascript" charset="utf-8"></script>
    <script src="jquery.tmpl.js" type="text/javascript" charset="utf-8"></script>
    <script src="spine.js" type="text/javascript" charset="utf-8"></script>
    <script src="spine.tmpl.js" type="text/javascript" charset="utf-8"></script>
    <script src="spine.list.js" type="text/javascript" charset="utf-8"></script>

As with the Spine.Contacts application, we're going to have a list of contact names in a sidebar. When users select a contact, its full information will be displayed in the main section. `Spine.List` will make that remarkably simple. It'll render the contacts list, control the currently selected contact, and trigger a *change* event when a new one is selected.

When instantiating a `Spine.List`, you need to pass in a `el` and `template` options, to indicate the element the list is associated with, and the template it should use for rendering records.

    var list = Spine.List.init({
      el: $("#sidebar ul"),
      template: function(items){
        return($("#items-template").tmpl(items));
      }
    });
    
The `template` option should be a function that takes records and returns DOM elements. In this example, we're calling `.tmpl()` on a inline script template. For more information on how this works, please see [jQuery.tmpl's docs](http://api.jquery.com/jquery.tmpl). 

We now need to render our list with records. Typically this is done whenever any records in the list are created, updated or destroyed. We can do this by passing a list of records to the list's `render()` function

    list.render(Contact.all());

Lastly we need to listen to *change* events, which will be invoked whenever users click on the list, changing the currently selected item.

    list.bind("change", function(item){
      // list was changed to item!
    });
    
That's all there is to `Spine.List`. Now let's show a full example to give you a bit of context. Firstly we're going to need two simple template, `#contacts-template` for the list and `#contact-template` for the main view. Then, as demonstrated above, we're going to render the list and listen to *change* events. When the currently selected item in the list changes, we'll re-render the main view with the newly selected record.

    <script type="text/x-template" charset="utf-8" id="contacts-template">
      <li>${name}</li>
    </script>
    
    <script type="text/x-template" charset="utf-8" id="contact-template">
      <p>${name}</p>
      <!-- ... -->
    </script>

    <div id="layout2">
      <div class="sidebar">
        <ul></ul>
      </div>
      <div class="main">
      </div>
    </div>

    <script type="text/javascript" charset="utf-8">
      (function(Spine, $){
        // Create model
        var Contact = Spine.Model.setup("Contact", ["name"]);
        
        // Create List, override default template
        var SideBar = Spine.List.create({
          template: function(items){
            return $("#contacts-template").tmpl(items)            
          }
        });
    
        // Create controller
        var Contacts = Spine.Controller.create({
          elements: {".sidebar": "listEl", ".main": "main"},
          proxied: ["render", "change"],
      
          init: function(){
            this.list = SideBar.init({el: this.listEl});
            this.list.bind("change", this.change);
            Contact.bind("refresh change", this.render);
          },

          render: function(){
            this.list.render(Contact.all());
            this.main.html($("#contact-template").tmpl(this.current));
          },
      
          change: function(item){
            this.current = item;
            this.render();
          }
        });
    
        Contacts.init({el: $("#layout2")});
        Contact.fetch();
      })(Spine, Spine.$);
    </script>
    
That may look like a lot of code, but in reality it's fairly straightforward. Whenever the currently selected item in the list changes, then `Contacts.fn.render()` function will be called, re-rendering the main view. When the application first loads, if no list items are selected then `Spine.List` will select the first item in its list. 

Like `Spine.Tabs`, `Spine.List` will add a class of *active* to the currently selected list item, which you can style with CSS to give the user a visual indication of what is currently selected. 