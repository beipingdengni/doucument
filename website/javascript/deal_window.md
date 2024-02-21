## 触发浏览器事件

### onload 页面加载



### beforeunload

####  刷新浏览器、关闭浏览器、关闭浏览器tab页触发事件

> 在 JavaScript 中检测浏览器或标签页关闭事件
>
> 参考博客: https://www.jiyik.com/tm/xwzj/web_3024.html 
>
> 不同浏览器的处理：https://blog.csdn.net/liming1016/article/details/120930257

```javascript
// beforeunload 事件用于检测浏览器或标签页是否正在关闭或网页是否正在重新加载。
// beforeunload 事件提醒用户。addEventListener() 函数在事件发生时监听事件。
// 在卸载窗口及其资源之前，会触发 beforeunload 事件
window.addEventListener('beforeunload', function (e) {
 	e.preventDefault();
 	e.returnValue = '';
});
```

#### onunload

> 使用window对象的onunload属性来指定一个函数，该函数将在窗口或框架被卸载时执行

```javascript
window.onunload = function() {  // 在这里编写需要执行的清理操作
  //如下：
  // 关闭数据库连接
  db.close();
  // 保存表单数据
  localStorage.setItem('form_data', JSON.stringify($('#myForm').serializeArray()));
};
```



>click，document write，document open，document close，window close ，window navigate ，window NavigateAndFind,location replace,location reload,form submit