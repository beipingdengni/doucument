# Celery

> pip install -U Celery
>
> pip install redis 
>
> ​	pip install -U "celery[redis]"
>
> pip install eventlet
>
> ```
> pip install "celery[librabbitmq]"
> pip install "celery[librabbitmq,redis,auth,msgpack]"
> ```

tasks.py

```python
from celery import Celery
#第一个参数"my_task"是celery实例应用名
app=Celery("my_task",broker="redis://localhost:6379/0",backend="redis://localhost:6379/1")
# app = Celery('tasks', backend='redis://localhost', broker='pyamqp://')
# use Redis as the result backend, but still use RabbitMQ as the message broker (a popular combination)

@app.task
def send_mail():
    print("发送邮件中****************************")
    return "邮件发送成功"
```

app.py

```python
from tasks import send_mail

if __name__ == '__main__':
    result=send_mail.delay()
    print(result)
```

执行命令`celery worker -A tasks -l info -P eventlet` 

或 `celery -A tasks worker --loglevel=INFO`

> -A 表示当前的任务的模块名，这里就是task.py的文件名；
> -l  表示celery的日志等级如info、debug
> -P 表示Pool implementation，线程池实现类？

