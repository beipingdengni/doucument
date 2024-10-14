 

#### 模板引擎

模板是一个`web`开发必备的模块。因为我们在渲染一个网页的时候，并不是只渲染一个纯文本[字符串](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E4%B8%B2&spm=1001.2101.3001.7020)，而是需要渲染一个有富文本标签的页面。这时候我们就需要使用模板了。

在`Flask`中，配套的模板是`Jinja2`，`Jinja2`的作者也是`Flask`的作者。这个模板非常的强大，并且执行效率高。以下对`Jinja2`做一个简单介绍！

```
from flask import Flask
from flask import Manager

#创建实例
app = Flask(__name__)


#创建对象
manager = Manager(app)

#视图函数
@app.route('/')
def index():
    return  '<h1>ig牛x</h1>'
    
## 启动实例
if __name__ == '__main__':
    app.run(debug=True,threaded=True,port=5050)
```

模版预热笔记：
-------

1.  在渲染模版的时候，默认会从项目根目录下的`templates`目录下查找模版。
    
2.  如果不想把模版文件放在`templates`目录下，那么可以在`Flask`初始化的时候指定`template_folder`来指定模版的路径。
    

####Jinja2使用

1.  准备工作，目录结构
    
    ```
    project/
    	manage.py			# 项目启动控制文件
    	templates/			# 所有的模板文件
    ```
    
    ```
        <meta charset="UTF-8">
        <title>flask模板引擎</title>
    </head>
    <body>
        <h1>test</h1>
    {#   四个{}表示变量#}
        {{ username }} 
        
       <ul>
           {% for foo in request %}
                <li></li>
           {% endfor %}
           
       </ul>
        
        <tr>
           {% for foo in request %}  控制语句 
                <td></td>
           {% endfor %}
           
        </tr>
        
        {% for foo in request %}
            <tr>
                <td></td>
            </tr>
        {% endfor %}
        
        {% if request %}
        
        {% endif %}   这是表达式  
    
    {#这是注释 #}  
    </body>
    </html>
    ```
    
2.  渲染模板文件
    
    在templates下创建一个模板文件(hello.html)，内容如下：
    
    ```html
    <h1>Hello Flask !</h1>
    ```
    
    模板渲染
    
    ```python
    # 自动加载模板文件
    app.config['TEMPLATES_AUTO_RELOAD'] = True
    
    @app.route('/')
    def index():
        # 渲染模板文件
        # return render_template('hello.html')
        # 渲染模板字符串
        return render_template_string('<h1>渲染字符串</h1>')
    
    from flask import Flask,render_template
    
    app = Flask(__name__,template_folder='C:/templates')
    
    #DTL:Django Tmplate Languate
    @app.route('/')
    def hello_world():
        return render_template('index.html')
    
    @app.route('/list/')
    def my_list():
        return render_template('posts/list.html')
    
    if __name__ == '__main__':
        app.run(debug=True)
    ```
    
3.  使用变量
    
    在templates下创建一个模板文件var.html，内容如下：
    
    ```html
    {# 这里是注释，渲染的变量放在两个大括号中 #}
    <h1>Hello {{ name }}</h1>
    <h1>Hello {{ g.name }}</h1>
    ```
    
    模板渲染
    
    ```python
    @app.route('/var/')
    def var():
        # g对象，在模板中使用不需要分配
        g.name = 'g的name属性'
        return render_template('var.html', name='xiaoming')
      	# 渲染模板字符串
        return render_template_string('<h1>Hello {{ name }}</h1>', 
                                      name='xxx')
    ```
    
4.  使用函数 过滤器
    
    在模板文件中可以使用特定的函数，对指定的变量处理后再显示，用法如下：
    
    ```html
    <h1>Hello {{ name | upper }}</h1>
    ```
    
    > 使用’|'将变量与函数分开，左边是变量名，右边是函数名
    
    常用函数
    
    | 函数 | 功能 |
    | --- | --- |
    | upper | 全部大写 |
    | lower | 全部小写 |
    | title | 每个单词首字母大写 |
    | capitalize | 首字母大写 |
    | trim | 去掉两边的空白 |
    | striptags | 过滤HTML标签 |
    | safe | 渲染时不转义，默认转义 |
    
    > 不要滥用safe，否则可能不安全，只能用在自己信任的变量上。
    
5.  流程控制
    
    分支结构(if-else)
    
    ```html
    {% if name %}
        <h1>Hello {{ name }} !</h1>
    {% else %}
        <h1>Hello world !</h1>
    {% endif %}
    ```
    
    循环结构(for-in)
    
    ```html
    <ol>
        {% for i in range(1, 6) %}
        <li>{{ i }}</li>
        {% endfor %}
    </ol>
    ```
    
6.  文件包含
    
    include1.html内容
    
    ```html
    <h1>include1.html中的内容</h1>
    
    {# 包含另一个文件，意思就是将被包含的文件内容粘贴到此处 #}
    {% include 'include2.html' %}
    ```
    
    include2.html内容
    
    ```html
    <div>include2.html中的内容</div>
    ```
    
7.  宏的使用
    
    宏的定义
    
    ```html
    {# 定义宏 #}
    {% macro show_name(name) %}
        <h1>Hello {{ name }}</h1>
    {% endmacro %}
    ```
    
    宏的调用
    
    ```html
    {# 调用宏 #}
    {{ show_name(name) }}
    ```
    
    宏的导入
    
    ```html
    {# 导入宏 #}
    {% from 'macro1.html' import show_name %}
    
    {# 调用宏 #}
    {{ show_name(name) }}
    ```
    
    说明：
    
    采用类似于python中的函数的形式进行定义和调用，通常我们可以把特定功能的内容显示定义成一个宏，哪里使用哪里调用即可，避免了相同效果的重复书写。
    
8.  模板继承
    
    基模板parents.html，内容如下：
    
    ```html
    <html>
    <head>
        <title>{% block title %}默认标题{% endblock %}</title>
    </head>
    <body>
        {% block body %}默认内容{% endblock %}
    </body>
    </html>
    ```
    
    子模板children.html，内容如下：
    
    ```html
    {# 继承自指定模板文件 #}
    {% extends 'parents.html' %}
    
    {# 可以对基础模板中的block进行删除，重写 #}
    {% block title %}欢迎登录{% endblock %}
    
    {# 重写基础模板中的block #}
    {% block body %}
        {# 保留基础模板中的内容 #}
        {{ super() }}
        <div>欢迎您的到来...</div>
    {% endblock %}
    ```
    
    > 当重写了一个block，原来的显示内容就没了，八成的原因是没有调用super.
    

#### flask\-bootsrap

1.  安装：
    
    ```shell
    pip install flask-bootstrap
    ```
    
2.  使用：
    
    ```python
    # 导入类库
    from flask_bootstrap import Bootstrap
    # 创建对象
    bootstrap = Bootstrap(app)
    ```
    
3.  测试模板文件boot.html，内容如下：
    
    ```html
    {# 继承自bootstrap的基础模板 #}
    {% extends 'bootstrap/base.html' %}
    
    {% block title %}用户注册{% endblock %}
    
    {% block content %}
        <div class="container">欢迎注册</div>
    {% endblock %}
    ```
    
4.  bootstrap继承模板base.html中定义的block
    
    | block | 说明 |
    | --- | --- |
    | doc | 整个HTML文档 |
    | html | html标签 |
    | head | head标签 |
    | title | title标签 |
    | styles | 引入css |
    | metas | 一组meta标签 |
    | body | body标签 |
    | navbar | 用户自定义导航条 |
    | content | 用户自定义内容 |
    | scripts | 用户定义的JS |
    
    > 使用bootstrap时，发现重写block后原来的显示效果消失，可能是忘了调用super
    

#### 定义项目基础模板

1.  说明：
    
    一个项目中，很多页面都很相似，只有细微的差别，如果每个都定制势必会有大量的重复代码的书写。为了简化这种重复工作，我们通常为项目定制一个基础模板(base)，让它继承自bootstrap的基础模板，其它页面继承该基础模板，只需稍微定制即可。
    
2.  步骤
    
    ```
    1.从bootcss.com复制一个顺眼的导航条
    2.将container-fluid改为container
    3.显示反色导航条：navbar-inverse
    4.将圆角改为直角：style="border-radius: 0px;"
    5.根据需要定制显示内容
    6.修改折叠目标的定位：data-target=".navbar-collapse"
    ```
    
3.  基础模板文件base.html，内容如下：
    
    ```html
    {% extends 'bootstrap/base.html' %}
    
    {# 定制标题 #}
    {% block title %}默认标题{% endblock %}
    
    {# 定制导航条 #}
    {% block navbar %}
        <nav class="navbar navbar-inverse" style="border-radius: 0px;">
            <div class="bs">
                <!-- Brand and toggle get grouped for better mobile display -->
                <div class="navbar-header">
                    <button type="button" class="navbar-toggle collapsed" data-toggle="collapse"
                            data-target=".navbar-collapse" aria-expanded="false">
                        <span class="sr-only">Toggle navigation</span>
                        <span class="icon-bar"></span>
                        <span class="icon-bar"></span>
                        <span class="icon-bar"></span>
                    </button>
                    <a class="navbar-brand" href="#">首页</a>
                </div>
    
                <!-- Collect the nav links, forms, and other content for toggling -->
                <div class="collapse navbar-collapse">
                    <ul class="nav navbar-nav">
                        <li><a href="#">板块一</a></li>
                        <li><a href="#">板块二</a></li>
                    </ul>
    
                    <ul class="nav navbar-nav navbar-right">
                        <li><a href="#">注册</a></li>
                        <li><a href="#">登录</a></li>
                    </ul>
                </div><!-- /.navbar-collapse -->
            </div><!-- /.container -->
        </nav>
    {% endblock %}
    
    {# 定制内容 #}
    {% block content %}
        <div class="container">
            {# 这里可以为弹出内容显示预留位置 #}
            {% block page_content %}默认内容{% endblock %}
        </div>
    {% endblock %}
    ```
    

#### 错误页面定制

1.  添加视图函数：
    
    ```python
    # 错误定制
    @app.errorhandler(404)
    def page_not_found(e):
        return render_template('404.html')
    ```
    
2.  定制404.html，内容如下：
    
    ```html
    {% extends 'base.html' %}
    
    {% block title %}出错了{% endblock %}
    
    {% block page_content %}<h1>臣妾实在找不到啊@_@</h1>{% endblock %}
    ```
    
3.  练习：再定义其它的错误显示页面，如：500等
    

#### 回顾url\_for函数

1.  根据视图函数名反向构造路由地址，路由需要的直接构造，多出来的参数以GET传递
    
    ```python
    url_for('var', name='xiaoming', pwd='123456')
    ```
    
    > /var/?name=xiaoming&pwd=123456，不完整，没有主机和端口
    
2.  若需要构造完整的外部链接，需要添加\_extenal=True的参数
    
    ```python
    url_for('var', name='xiaoming', pwd='123456', _external=True)
    ```
    
    > http://127.0.0.1:5000/var/?name=xiaoming&pwd=123456
    
3.  通常网站中的点击链接都是使用url\_for构造的，如：
    
    ```html
    <li><a href="{{ url_for('register') }}">注册</a></li>
    ```
    

#### 加载静态资源

1.  flask框架中静态资源的默认目录为static，项目目录结构如下：
    
    ```
    project/
    	manage.py			# 启动控制文件
    	static/				# 静态资源
    	templates/			# 模板文件
    ```
    
2.  加载收藏夹的图标
    
    ```html
    {# 加载收藏夹的图标 #}
    {% block head %}
        {{ super() }}
        <link rel="icon" type="image/x-icon" href="{{ url_for('static', 
                                             filename='favicon.ico') }}" />
    {% endblock %}
    ```
    
3.  加载图片资源
    
    ```html
    <img src="{{ url_for('static', filename='cluo.jpg') }}">
    ```
    
4.  加载CSS
    
    ```
    {# 加载CSS #}
    {% block styles %}
        {{ super() }}
        <link rel="stylesheet" type="text/css" href="{{ url_for('static', 
        	filename='common.css') }}" />
    {% endblock %}
    ```
    
5.  加载JS
    
    ```html
    {# 加载JS #}
    {% block scripts %}
        {{ super() }}
        <script type="text/javascript" src="{{ url_for('static', filename='common.js') }}"></script>
    {% endblock %}
    ```

本文转自 <https://blog.csdn.net/yinjun3215/article/details/108208214>，如有侵权，请联系删除。