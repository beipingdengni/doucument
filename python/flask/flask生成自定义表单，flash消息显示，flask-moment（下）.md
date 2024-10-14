 

1.构造下图所示的表结构：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad86af0b5359959aa6c690d44d339a9e.png#pic_center)  
2.index.html里面的代码如下：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="" method="post">
    <table>
        <tbody>
{#            <tr>#}
{#                <td>用户名:</td>#}
{#                <td><input type="text" name="username"></td>#}
{#            </tr>#}
            <tr>
                <td>{{ form.username.label }}</td>
                <td>{{ form.username(class='haha') }}</td>
            </tr>
            <tr>
                <td>{{ form.password.label }}</td>
                <td>{{ form.password}}</td>
            </tr>
            <tr>
                <td></td>
                <td>{{ form.submit }}</td>
            </tr>
        </tbody>
    </table>

</form>
</body>
</html>
```

3.login.html里面的代码如下：

```
{#<!DOCTYPE html>#}
{#<html lang="en">#}
{#<head>#}
{#    <meta charset="UTF-8">#}
{#    <title>登录页面</title>#}
{#</head>#}
{#<body>#}
{#action不写 默认提交到本页面#}
{#<form action="" method="post">#}
{#<form action="{{ url_for('check') }}" method="post">#}
{#    <label for="username">用户名:</label>#}
{#    <input type="text" name="username666" placeholder="请输入用户名"/>#}
{#    <label for="username">密码:</label>#}
{#    <input type="password" name="password666" placeholder="请输入密码"/>#}
{#    <input type="submit" value="立即登录" />#}
{#</form>#}
{##}
{##}
{#</body>#}
{#</html>#}

{% extends 'bootstrap/base.html' %}


{#导入渲染工具  bootstrap提供的宏 #}
{% import 'bootstrap/wtf.html' as wtf %}


{% block content %}
    <div class="container">


        {{ wtf.quick_form(form) }}
    </div>
    {% for message in get_flashed_messages() %}
          <div class="alert alert-warning alert-dismissible" role="alert">
     <button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>
        <strong>Warning!</strong>{{ message }}
    </div>
    {% endfor %}

{% endblock %}
```

4.moment.html里面的代码如下：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>


{#  使用 moment.js需要导入 moment.js文件  它还依赖于另外一个 jQuery
先把jQuery导入

#}

<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
{{ moment.include_moment() }}
{{ moment.locale('zh-CN') }}
<h1>简单时间格式化</h1>
<div>{{ moment(pub_time).format('LLLL') }}</div>
<div>{{ moment(pub_time).format('LLL') }}</div>
<div>{{ moment(pub_time).format('LL') }}</div>
<div>{{ moment(pub_time).format('L') }}</div>


<div>{{ moment(pub_time).format('YYYY-MM-DD') }}</div>
<div>{{ moment(pub_time).fromNow() }}</div>


<h2>时间差值</h2>



</body>
</html>
```

5.register.html里面的代码如下：

```
{% extends 'bootstrap/base.html' %}


{#导入渲染工具  bootstrap提供的宏 #}
{% import 'bootstrap/wtf.html' as wtf %}


{% block content %}
    <div class="container">


        {{ wtf.quick_form(form) }}
    </div>
{% endblock %}
```

6.app.py里面的代码如下：

```
from flask import Flask,render_template,request,flash,redirect,url_for
from flask_wtf import FlaskForm
from flask_script import Manager
from flask_bootstrap import Bootstrap
from wtforms import StringField,SubmitField #导入表单字段
from wtforms.validators import DataRequired,Length
from flask_moment import Moment
app = Flask(__name__)
app.config['SECRET_KEY'] = '12SDADFS'  #表单提交防止csrf 跨站请求伪造
#需要设置 密钥
app.config['BOOTSTRAP_SERVE_LOCAL'] = True #使用本地的bootstrap静态文件
bootstrap = Bootstrap(app)
manager = Manager(app)
moment  = Moment(app)
# app.config['TEMPLATES_AUTO_RELOAD'] = True

#定义一个表单类
class LoginForm(FlaskForm):
    username = StringField('用户名',validators=[DataRequired(message='请输入用户名'),
    Length(min=6,max=30,message='最少6位最大30位')])#<input type='text'>
    password = StringField('密码',validators=[DataRequired(message='请输入密码'),
    Length(min=6,max=30,message='密码必须6到30位')])
    submit = SubmitField('提交')#<input type='submit'>



@app.route('/')
def hello_world():
    form = LoginForm()
    return render_template('index.html',form=form)

@app.route('/register')
def register():
    form = LoginForm()
    return render_template('register.html',form=form)


@app.route('/login',methods=['GET','POST'])
def login():
    form = LoginForm()
    if request.method == 'GET':
        return render_template('login.html',form=form)
    else:
        if form.validate():
            username = form.username.data
            password = form.password.data
            print(username,password)
            return 'success'
        else:
            flash(message=form.errors)
            # flash全局作用域中用来存放提示消息
            #在页面上 我们直接可以   {% for message in get_flashed_messages() %}
            # {{message}}
            return redirect(url_for('login'))
    # if form.validate():
    #     if form.username.data != 'kangbazi' or form.password.data != '123456':
    #         flash('用户名或者密码错误')
    #         return redirect(url_for('login'))
    # return render_template('login.html',form=form)



#登录注册都是这个思路 首先判断用户请求 get 请求返回登录或者注册页面
#post 请求 接收表单的数据
# @app.route('/login',methods=['GET','POST'])
# def login():
#     # print(request.method)
#     # print(request.args['username']) #默认只允许get请求过来
#     if request.method == 'GET':
#         return render_template('login.html')
#     else:
#         print(request.form['username666'])
#         print(request.form['password666'])
#         return 'success'

@app.route('/check',methods=['POST'])
def check():
    return "hello：%s" % request.form['username666']


from datetime import datetime,timedelta
@app.route('/moment',methods=['GET'])
def moment():
    # pub_time = datetime.utcnow()
    # pub_time = datetime(year=2020,month=8,day=19,hour=13,minute=30,second=0)
    pub_time = datetime.utcnow() + timedelta(seconds=-3600)
    return render_template('moment.html',pub_time=pub_time)
if __name__ == '__main__':
    manager.run()

```

标红的可能是没有下载相应的包，请记得提前下载好。  
回到app.py文件，在下面输入：  
python app.py runserver -d -r -p 5050  
点击回车，出现网址，点击进去进行一系列验证操作了，这里我就不再多说，朋友们可以自由发挥。
