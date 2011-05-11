#Manager
#Tabs

    <div id="layout1">
      <div class="tabs">
        <ul>
          <li data-name="users">Users</li>
          <li data-name="groups">Groups</li>
        </ul>
      </div>
  
      <div class="views">
  
        <div class="users">
          <div class="sidebar">
            <ul>
              <li>Option 1</li>
              <li>Option 2</li>
            </ul>
          </div>
  
          <div class="main">
            Main
          </div>
        </div>
    
        <div class="groups">
          <h3>Groups</h3>
        </div>
    </div>

    <script type="text/javascript" charset="utf-8">
      (function(){
        var Users  = Spine.Controller.create();
        var Groups = Spine.Controller.create();
        
        var users  = Users.init();
        var groups = Groups.init();
    
        Spine.Manager.init(users, groups);
    
        var tabs = Spine.Tabs.init();
        tabs.connect("users", users);
        tabs.connect("groups", groups);
      })();
    </script>

#List



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
        
        var SideBar = Spine.List.create({
          template: function(items){
            return $("#contacts-template").tmpl(items)            
          }
        });
    
        // Create controller
        var Contacts = Spine.Controller.create({
          elements: {".list": "listEl"},
          proxied: ["change"],
      
          init: function(){
            this.list = SideBar.init({el: this.listEl});

            this.list.bind("change", this.change);
            Contact.bind("refresh change", this.render);
          },

          render: function(){
            this.list.render(Contact.all());
          },
      
          change: function(item){
            this.current = item;
            this.el.html($("#contact-template").tmpl(item));
          }
        });
    
        Contacts.init({el: $("#layout2")});
      })(Spine, Spine.$);
    </script>