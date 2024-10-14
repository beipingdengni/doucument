

```html
<!DOCTYPE html>
<html>
<head>
    <title>获取元素的id</title>
    <script src="
</head>
<body>
    <button id="myButton">点击我</button>
    
    <script>
        $(document).ready(function(){
            $("button").click(function(){
                var id = $(this).attr("id");
                alert("按钮的id为：" + id);
            });
        });
    </script>
</body>
</html>
```

