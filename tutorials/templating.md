<div class="back"><a href="index.html">&laquo; Back to all tutorials</a></div>

#Using jQuery.tmpl
#Loading in templates
#Rendering templates
#Finding template elements

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