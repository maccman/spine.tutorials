<div class="back"><a href="index.html">&laquo; Back to all tutorials</a></div>

#Using Spine and CoffeeScript

[CoffeeScript](http://jashkenas.github.com/coffee-script) is a "little language that compiles into JavaScript". Essentially, its aim is to improve JavaScript syntax, meaning less typing for you, and often less errors. 

CoffeeScript files compile down to pure JavaScript, and there's no interpretation at run time. This means you can use any existing JavaScript library seamlessly, and vice-versa. In other words, you can write you Spine applications in CoffeeScript, giving you a much sweeter syntax and saving time.

You can use Spine & CoffeeScript as-is, without any API changes. Use the same function names and properties, just use CoffeeScript's syntax for defining function and object literals. 

I'm not going to elaborate much on CoffeeScript's syntax here, see the [official documentation](http://jashkenas.github.com/coffee-script) for that. What I am going to do though, is show you an example of a few Spine models and controllers, namely convert [Spine.Todos](http://github.com/maccman/spine.todos) into CoffeeScript.

Let's take the `Tasks` controller:

    window.Tasks = Spine.Controller.create({
      tag: "li",

      proxied: ["render", "remove"],

      events: {
        "change   input[type=checkbox]": "toggle",
        "blur     input[type=text]":     "close"
      },

      init: function(){
        this.item.bind("update",  this.render);
      }
      // ...
      
First we're going to remove all `var` keywords, CoffeeScript will deal with scope for us. Then remove all semi colons, CoffeeScript inserts those later automatically. The next step is to use CoffeeScript's object literal syntax; as you can see braces are optional. Finally, we're going to convert calls to `function()` to CoffeeScript's function syntax, namely `->`. Here's the finished result:
      
    Tasks = Spine.Controller.create
      tag: "li"

      proxied: ["render", "remove"]

      events:
       "change   input[type=checkbox]": "toggle",
       "blur     input[type=text]":     "close"

      init: ->
        this.item.bind("update",  this.render)

You can see the resultant syntax is much clearer, with less room for errors. You can also see the full [before](https://github.com/maccman/spine.todos/blob/master/app/application.js) and [after](https://github.com/maccman/spine.todos/blob/coffee/app/application.coffee) scripts for a comparison, and the CoffeeScript version of application.js is included below for convenience. 


    Task = Spine.Model.setup("Task", ["name", "done"])

    Task.extend(Spine.Model.Local)

    Task.extend
      active: ->
        this.select (item) -> !item.done

      done: ->
        this.select (item) -> !!item.done

      destroyDone: ->
        rec.destroy() for rec in this.done()

    Tasks = Spine.Controller.create
      tag: "li"

      proxied: ["render", "remove"]

      events:
       "change   input[type=checkbox]": "toggle",
       "click    .destroy":             "destroy",
       "dblclick .view":                "edit",
       "keypress input[type=text]":     "blurOnEnter",
       "blur     input[type=text]":     "close"

      elements:
        "input[type=text]": "input",
        ".item": "wrapper"

      init: ->
        this.item.bind("update",  this.render)
        this.item.bind("destroy", this.remove)

      render: ->
        elements = $("#taskTemplate").tmpl(this.item)
        this.el.html(elements)
        this.refreshElements()
        this

      toggle: ->
        this.item.done = !this.item.done
        this.item.save()

      destroy: ->
        this.item.destroy()

      edit: ->
        this.wrapper.addClass("editing")
        this.input.focus()

      blurOnEnter: (e) ->
        if e.keyCode == 13 then e.target.blur()

      close: ->
        this.wrapper.removeClass("editing")
        this.item.updateAttributes({name: this.input.val()})

      remove: ->
        this.el.remove()

    TaskApp = Spine.Controller.create    
      proxied: ["addOne", "addAll", "renderCount"]

      events:
        "submit form":   "create",
        "click  .clear": "clear"

      elements:
        ".items":     "items",
        ".countVal":  "count",
        ".clear":     "clear",
        "form input": "input"

      init: ->
        Task.bind("create",  this.addOne)
        Task.bind("refresh", this.addAll)
        Task.bind("refresh change", this.renderCount)
        Task.fetch()

      addOne: (task) ->
        view = Tasks.init({item: task})
        this.items.append(view.render().el)

      addAll: ->
        Task.each(this.addOne)

      create: (e) ->
        e.preventDefault()
        Task.create({name: this.input.val()})
        this.input.val("")

      clear: ->
        Task.destroyDone()

      renderCount: ->
        active = Task.active().length
        this.count.text(active)

        inactive = Task.done().length
        if inactive 
          this.clear.show()
        else
          this.clear.hide()

    jQuery ->
      TaskApp.init(el: $("#tasks"))