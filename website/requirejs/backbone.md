

### Backbone.js+Require.js+Bootstrap构建web平台

https://www.cnblogs.com/xu-wojustme/p/5685197.html



```html
<!DOCTYPE html>
<html>
<head>
    <title>the5fire.com-backbone.js-Hello World</title>
</head>
<body>
    <button id="check">新手报到</button>
    <ul id="world-list">
    </ul>
    <a href="http://www.the5fire.com">更多教程</a>
    <script src="http://the5fireblog.b0.upaiyun.com/staticfile/jquery-1.10.2.js"></script>
    <script src="http://the5fireblog.b0.upaiyun.com/staticfile/underscore.js"></script>
    <script src="http://the5fireblog.b0.upaiyun.com/staticfile/backbone.js"></script>
    <script>
    (function ($) {
        World = Backbone.Model.extend({
            //创建一个World的对象，拥有name属性
            name: null
        });

        Worlds = Backbone.Collection.extend({
            //World对象的集合
            initialize: function (models, options) {
                    this.bind("add", options.view.addOneWorld);
            }
        });

        AppView = Backbone.View.extend({
            el: $("body"),
            initialize: function () {
                //构造函数，实例化一个World集合类
                //并且以字典方式传入AppView的对象
                this.worlds = new Worlds(null, { view : this })
            },
            events: {
                //事件绑定，绑定Dom中id为check的元素
                "click #check":  "checkIn",
            },
            checkIn: function () {
                var world_name = prompt("请问，您是哪星人?");
                if(world_name == "") world_name = '未知';
                var world = new World({ name: world_name });
                this.worlds.add(world);
            },
            addOneWorld: function(model) {
                $("#world-list").append("<li>这里是来自 <b>" + model.get('name') + "</b> 星球的问候：hello world！</li>");
            }
        });
        //实例化AppView
        var appview = new AppView;
    })(jQuery);
    </script>
</body>
</html>
```

