---
title: Celery分布式任务队列入门(三)-任务
date: 2017-12-10 23:57:26
updated: 2017-12-11 16:21:46
tags:
    - Celery
categories: 
    - Celery分布式任务队列入门
description: Celery加载配置文件，设置异步任务和定时任务的方法等
---

在上一篇 [Celery分布式任务队列入门(二)-环境配置](/2017/12/10/celery-distributed-task-queue-introduction-second/) 中介绍了一种简单任务的创建方法。

对于任务，在Celery中主要分为 `异步任务` 和 `定时任务`，下面详细的来说说。


#### 配置

Celery中的配置可以直接在应用上设置，也可以使用一个独立的配置模块。

##### 直接配置

例如你可以通过修改 `CELERY_TASK_SERIALIZER` 选项来配置序列化任务载荷的默认的序列化方式：

```
app.conf.CELERY_TASK_SERIALIZER = 'json'
```

一次性设置多个选项，你可以使用 `update()` 方法：

```
app.conf.update(
    CELERY_TASK_SERIALIZER='json',
    CELERY_ACCEPT_CONTENT=['json'],  # Ignore other content
    CELERY_RESULT_SERIALIZER='json',
    CELERY_TIMEZONE='Europe/Oslo',
    CELERY_ENABLE_UTC=True,
)
```

示例：

```
from celery import Celery


CELERY_CONFIG = {
    'CELERY_TIMEZONE': 'Asia/Shanghai',
    'CELERY_ENABLE_UTC': True,
    # content
    'CELERY_TASK_SERIALIZER': 'json',
    'CELERY_RESULT_SERIALIZER': 'json',
    'CELERY_ACCEPT_CONTENT': ['json'],
    'CELERYD_MAX_TASKS_PER_CHILD': 1
}
SETTINGS = {
    'user': 'www-data',
    'password': 'www-data',
    'host': '127.0.0.1',
    'port': '5672',
    'vhost': 't_celery'
}

app = Celery(
    'test_celery',
    broker='amqp://{user}:{password}@{host}:{port}/{vhost}'.format(
        **SETTINGS)
)
app.conf.update(**CELERY_CONFIG)

```

****

##### 独立配置模块

对于大型项目，采用独立配置模块更为有效。

可以调用 `config_from_object()` 来让 `Celery` 实例加载配置模块：

```
app.config_from_object('celeryconfig')
```

配置模块通常称为 `celeryconfig` ，你也可以使用任意的模块名。名为 `celeryconfig.py` 的模块必须可以从当前目录或 `Python` 路径加载。

`celeryconfig.py` 格式一般为：

```
BROKER_URL = 'amqp://'
CELERY_RESULT_BACKEND = 'amqp://'

CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_ACCEPT_CONTENT=['json']
CELERY_TIMEZONE = 'Europe/Oslo'
CELERY_ENABLE_UTC = True
```

检查配置文件的语法错误，可以：

```
$ python -m celeryconfig
```

更多配置参数参考：[Configuration and defaults](http://docs.jinkan.org/docs/celery/configuration.html#configuration)


有关配置的使用可以看我另一篇文章中的详细介绍。

***

#### 异步任务

在初级篇中我们创建的简单任务 `tasks.py` 就是一个异步任务。`tasks.py` ：

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from celery import Celery

app = Celery('tasks',
             broker='amqp://myuser:hello@localhost:5672/hellohost',
             backend='amqp://myuser:hello@localhost:5672/hellohost'
             )


@app.task
def add(x, y):
    return x + y

```

* 创建了一个 `Celery` 实例 `app` ，指定名称为 `tasks`
* 指定消息中间件 `broker` 使用 `RabbitMQ` ，指定结果存储 `backend` 使用 `RabbitMQ`
* 创建了一个 `Celery` 任务 `add`，当函数被 `@app.task` 装饰后，就成为可被 `Celery` 调度的任务

##### 运行 worker

在 tasks.py 文件所在目录执行：

```
$ celery -A tasks worker --loglevel=info
```

这个命令会开启一个在前台运行的 worker。

参数说明：

* `worker` : 运行 `worker` 模块
* `-A: –app=APP`  指定使用的 `Celery` 实例所在的文件模块
* `-l: -–loglevel=INFO` 指定日志级别，默认为 `WARNING`，可选：`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`, `FATAL`


如果是创建任务模块，可以使用模块名称来启动：

```
$ celery -A proj worker -l info
```

或者使用完整命令：

```
$ celery worker --app=proj -l info
```

查看完整的帮助信息：

```
$ celery worker --help
```

示例：

```
root@b792ae940e3e:/app# celery worker --help
usage: celery worker [options] 

Start worker instance.

Examples:

        $ celery worker --app=proj -l info
        $ celery worker -A proj -l info -Q hipri,lopri

        $ celery worker -A proj --concurrency=4
        $ celery worker -A proj --concurrency=1000 -P eventlet
        $ celery worker --autoscale=10,0
```

* [Workers Guide](http://docs.celeryproject.org/en/latest/userguide/workers.html#starting-the-worker)
* [celery.bin.worker](http://docs.celeryproject.org/en/latest/reference/celery.bin.worker.html#celery-bin-worker)


**扩展**

对于参数 `-A: –app=APP` 表示指定使用的 `Celery` 实例。即指py文件的文件名（不包括扩展名.py）或项目的模块名


比如项目结构如下：

```
testcelery/
    |- src/
        |- __init__.py
        |- app.py
        |- task.py
```

那么启动worker任务 `task.py` 的命令即为：

```
$ celery worker -A src.task -l info
```

##### 调用 Task

###### delay调用

在异步调用方式中，可以通过 `delay` 或者 `apply_async` 来实现。

```
from tasks import add

add.delay(3, 4)
```

示例，创建文件 `client.py` :

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from tasks import add

if __name__ == '__main__':
    add.delay(1, 5)
```

执行 `$ python3 client.py` 就能调用执行了。


###### apply_async

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from tasks import add

if __name__ == '__main__':
    # add.delay(1, 5)
    add.apply_async(args=(5, 6))

```

`delay` 和 `apply_async` 这两种调用方式等价，`delay` 是 `apply_async` 的简写。用于一个任务消息（task message）。之前的示例中我们发现 `add` 任务并没有返回 “计算结果”，而是返回了一个对象 `AsyncResult`，它的作用是被用来检查任务状态，等待任务执行完毕或获取任务结果，如果任务失败，它会返回异常信息或者调用栈。


`apply_async` 参数

apply_async 相比 delay的优点就是，apply_async支持更多的参数。

```
apply_async(args=(), kwargs={}, route_name=None, **options)
```

apply_async 常用的参数如下：

* `countdown` ：任务延迟执行的秒数，默认立即执行；
```
task1.apply_async(args=(2, 3), countdown=5)    # 5 秒后执行任务
```
* `eta` ：任务被执行的绝对时间，参数类型是 `datetime`
```
from datetime import datetime, timedelta

# 当前 UTC 时间再加 10 秒后执行任务
task1.multiply.apply_async(args=[3, 7], eta=datetime.utcnow() + timedelta(seconds=10))
```
* `expires` : 任务过期时间，参数类型可以是 `int`，也可以是 `datetime`
```
task1.multiply.apply_async(args=[3, 7], expires=10)    # 10 秒后过期
```

更多的参数列表可以在 [官方文档](http://docs.celeryproject.org/en/latest/reference/celery.app.task.html#celery.app.task.Task.apply_async) 中查看。

###### send_task调用

除了使用 `delay` 的方式，还已通过 `send_task()` 的方式来调用。同时 `send_task()` 也支持设置更多的参数。

示例，`client.py` :

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from tasks import app

if __name__ == '__main__':
    # add.delay(1, 5)
    # add.apply_async(args=(5, 6))
    app.send_task('tasks.add', args=(12, 23),)

```

注意，这里引入的是 `app` 实例。`args` 参数是一个元组类型。相应的执行结果为：

```
[2017-12-10 07:11:38,057: INFO/MainProcess] Received task: tasks.add[83fd530f-d800-43c7-bcfe-920a176812e2]  
[2017-12-10 07:11:43,217: INFO/ForkPoolWorker-1] Task tasks.add[83fd530f-d800-43c7-bcfe-920a176812e2] succeeded in 5.1578904589996455s: 35
```

##### AsyncResult方法

上一篇文章中我们提到过返回对象 `AsyncResult` 的 `ready()` 方法，继续来看一下其他的方法：

`ready` 为 `True` 表示已经返回结果了

```
>>> result.ready()
True
```

`status` 表示任务执行状态，失败还是成功

```
>>> result.status
'SUCCESS'
```

`result` 和 `get()` 表示返回的结果

```
>>> result.result
7

>>> result.get() 
7
```

`id` 用来查看任务的id属性：

```
>>> result.id
'c178619e-3af3-41ed-8d2c-6371de80a601'
```


#### 定时任务

`Celery Beat` 进程通过读取配置文件的内容，周期性地将定时任务发往任务队列。

在Celery的定时任务中，重要的两个方法是 定时器 和 执行器：

* 定时器，也叫作 beater，也就是帮助我们计算什么时候执行什么操作
* 执行器，也叫作 worker，真正执行任务的地方，我们的任务都是通过这个运行的

创建Celery定时任务有多中方法。

##### 通过配置文件方式

可以在配置文件中通过 `CELERYBEAT_SCHEDULE` 设置定时。

###### 简单定时任务

创建一个Celery的任务 `tasks.py`：

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from celery import Celery

app = Celery('tasks',
             broker='amqp://myuser:hello@localhost:5672/hellohost',
             backend='amqp://myuser:hello@localhost:5672/hellohost'
             )


app.conf.update(
    # 配置定时任务
    CELERYBEAT_SCHEDULE={
        'my_task': {
            'task': 'tasks.add',
            'schedule': 60,
            'args': (22, 34),
        }
    }
)


@app.task
def add(x, y):
    return x + y

```

然后，启动这个 `worker` 进程：

```
# celery -A tasks worker -l info
```

接着，启动 `Celery Beat` 进程，定时将任务发送到 `Broker` ，在另一个命令行窗口下执行：

```
# celery beat -A tasks -l info
```

可以看到提示信息：

```
root@b792ae940e3e:/app# celery beat -A tasks -l info
celery beat v4.1.0 (latentcall) is starting.
__    -    ... __   -        _
LocalTime -> 2017-12-10 08:48:29
Configuration ->
    . broker -> amqp://myuser:**@localhost:5672/hellohost
    . loader -> celery.loaders.app.AppLoader
    . scheduler -> celery.beat.PersistentScheduler
    . db -> celerybeat-schedule
    . logfile -> [stderr]@%INFO
    . maxinterval -> 5.00 minutes (300s)
[2017-12-10 08:48:29,141: INFO/MainProcess] beat: Starting...
```

一段时间后，可以看到执行结果：

`beat` 执行结果：

```
[2017-12-10 08:49:29,277: INFO/MainProcess] Scheduler: Sending due task my_task (tasks.add)
[2017-12-10 08:50:29,289: INFO/MainProcess] Scheduler: Sending due task my_task (tasks.add)
[2017-12-10 08:51:29,322: INFO/MainProcess] Scheduler: Sending due task my_task (tasks.add)
[2017-12-10 08:52:29,334: INFO/MainProcess] Scheduler: Sending due task my_task (tasks.add)
[2017-12-10 08:53:29,375: INFO/MainProcess] Scheduler: Sending due task my_task (tasks.add)
[2017-12-10 08:54:29,411: INFO/MainProcess] Scheduler: Sending due task my_task (tasks.add)
[2017-12-10 08:55:29,426: INFO/MainProcess] Scheduler: Sending due task my_task (tasks.add)
```

`workder` 执行结果：

```
[2017-12-10 08:41:37,839: INFO/MainProcess] celery@b792ae940e3e ready.
[2017-12-10 08:49:29,290: INFO/MainProcess] Received task: tasks.add[1667ee3b-58c8-4a1a-be9f-0c20c3086bde]  
[2017-12-10 08:49:29,321: INFO/ForkPoolWorker-1] Task tasks.add[1667ee3b-58c8-4a1a-be9f-0c20c3086bde] succeeded in 0.029423292000501533s: 56
[2017-12-10 08:50:29,295: INFO/MainProcess] Received task: tasks.add[db2d9ead-c22f-4efe-8d1c-6995f0ce9148]  
[2017-12-10 08:50:29,345: INFO/ForkPoolWorker-1] Task tasks.add[db2d9ead-c22f-4efe-8d1c-6995f0ce9148] succeeded in 0.046812374001092394s: 56
[2017-12-10 08:51:29,324: INFO/MainProcess] Received task: tasks.add[7b23af5f-a467-4263-9cd0-87486f2df25d]  
[2017-12-10 08:51:29,338: INFO/ForkPoolWorker-1] Task tasks.add[7b23af5f-a467-4263-9cd0-87486f2df25d] succeeded in 0.011024585000996012s: 56
[2017-12-10 08:52:29,336: INFO/MainProcess] Received task: tasks.add[2f3489e4-625e-460a-bc57-3fdf13456d97]  
[2017-12-10 08:52:29,349: INFO/ForkPoolWorker-1] Task tasks.add[2f3489e4-625e-460a-bc57-3fdf13456d97] succeeded in 0.01185457899919129s: 56
[2017-12-10 08:53:29,381: INFO/MainProcess] Received task: tasks.add[ec7377fb-1859-4980-861b-2592422aad8c]  
[2017-12-10 08:53:29,408: INFO/ForkPoolWorker-1] Task tasks.add[ec7377fb-1859-4980-861b-2592422aad8c] succeeded in 0.021194035000007716s: 56
```

***

上面定时任务的配置信息表示：

```
    # 配置定时任务
    CELERYBEAT_SCHEDULE={
        'my_task': {
            'task': 'tasks.add',
            'schedule': 60,
            'args': (22, 34),
        }
    }
```

其中：

* `my_task` 表示当前任务的名称，可以自定义指定
* `task` 表示 `tasks.py` 模块下的 `add` 方法
* `schedule` 表示 任务执行的间隔，如果使用 `int` 类型，则单位是秒；还可以使用 `timedelta` 类型
* `args` 表示任务函数参数，注意参数类型为元组

***

如果不通过 `update` 来修改，还可以通过设置 `beat_schedule` 配置项来设置。

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from celery import Celery

app = Celery('tasks',
             broker='amqp://myuser:hello@localhost:5672/hellohost',
             backend='amqp://myuser:hello@localhost:5672/hellohost'
             )


app.conf.beat_schedule = {
    'my_task': {
        'task': 'tasks.add',
        'schedule': 60,
        'args': (1, 2),
    }
}
app.conf.timezone = 'UTC'

@app.task
def add(x, y):
    return x + y

```

###### 多项定时任务

我们还可以在配置文件中同时定义多个定时任务，只需要在 `CELERYBEAT_SCHEDULE` 项中添加即可:

```
    # 配置定时任务
    CELERYBEAT_SCHEDULE={
        'my_task': {
            'task': 'tasks.add',
            'schedule': 60,
            'args': (22, 34),
        },
        'your_task':{
          'task':'tasks,add',
          'schedule':30,
          'args':(1,4),
        }
    }
```

相应的执行结果：

`beat` 执行结果：

```
root@b792ae940e3e:/app# celery beat -A tasks -l info

[2017-12-10 09:34:50,113: INFO/MainProcess] beat: Starting...
[2017-12-10 09:35:20,151: INFO/MainProcess] Scheduler: Sending due task your_task (tasks.add)
[2017-12-10 09:35:50,145: INFO/MainProcess] Scheduler: Sending due task your_task (tasks.add)
[2017-12-10 09:35:50,149: INFO/MainProcess] Scheduler: Sending due task my_task (tasks.add)
[2017-12-10 09:36:20,166: INFO/MainProcess] Scheduler: Sending due task your_task (tasks.add)
[2017-12-10 09:36:50,187: INFO/MainProcess] Scheduler: Sending due task your_task (tasks.add)
[2017-12-10 09:36:50,188: INFO/MainProcess] Scheduler: Sending due task my_task (tasks.add)
[2017-12-10 09:37:20,208: INFO/MainProcess] Scheduler: Sending due task your_task (tasks.add)
```

`worker` 执行结果：

```
root@b792ae940e3e:/app# celery worker -A tasks -l info

[2017-12-10 09:34:45,833: INFO/MainProcess] celery@b792ae940e3e ready.
[2017-12-10 09:35:20,165: INFO/MainProcess] Received task: tasks.add[2b236ccf-c4af-430c-8d4e-a5cbaf331123]  
[2017-12-10 09:35:20,213: INFO/ForkPoolWorker-1] Task tasks.add[2b236ccf-c4af-430c-8d4e-a5cbaf331123] succeeded in 0.04618659699917771s: 5
[2017-12-10 09:35:50,151: INFO/MainProcess] Received task: tasks.add[01f843bc-8506-4d61-97ce-5f989c576509]  
[2017-12-10 09:35:50,159: INFO/MainProcess] Received task: tasks.add[f3230694-ce5a-4bc3-9c92-eea71b9c4949]  
[2017-12-10 09:35:50,187: INFO/ForkPoolWorker-1] Task tasks.add[01f843bc-8506-4d61-97ce-5f989c576509] succeeded in 0.03170933700130263s: 5
[2017-12-10 09:35:50,214: INFO/ForkPoolWorker-2] Task tasks.add[f3230694-ce5a-4bc3-9c92-eea71b9c4949] succeeded in 0.04673135899975023s: 56
[2017-12-10 09:36:20,168: INFO/MainProcess] Received task: tasks.add[410e86bf-2e06-4a7d-965d-92d2e6a72c12]  
[2017-12-10 09:36:20,186: INFO/ForkPoolWorker-1] Task tasks.add[410e86bf-2e06-4a7d-965d-92d2e6a72c12] succeeded in 0.01682951399925514s: 5
[2017-12-10 09:36:50,191: INFO/MainProcess] Received task: tasks.add[d2b32ff2-8e8e-4903-ad8f-7d12da132739]  
[2017-12-10 09:36:50,193: INFO/MainProcess] Received task: tasks.add[dc7fc344-374e-438b-8a0f-85f9e9f08aac]  
[2017-12-10 09:36:50,207: INFO/ForkPoolWorker-1] Task tasks.add[d2b32ff2-8e8e-4903-ad8f-7d12da132739] succeeded in 0.013693080998564255s: 5
[2017-12-10 09:36:50,211: INFO/ForkPoolWorker-2] Task tasks.add[dc7fc344-374e-438b-8a0f-85f9e9f08aac] succeeded in 0.017022554000504897s: 56
[2017-12-10 09:37:20,213: INFO/MainProcess] Received task: tasks.add[49113d2d-a36d-44b0-92dc-9ad0382c26a2]  
[2017-12-10 09:37:20,246: INFO/ForkPoolWorker-1] Task tasks.add[49113d2d-a36d-44b0-92dc-9ad0382c26a2] succeeded in 0.027591496000241023s: 5
```


在上面，我们用两个命令启动了 `Worker` 进程和 `Beat` 进程，我们也可以将它们放在一个命令中：

```
$ celery -B -A tasks worker --loglevel=info
```

相应的执行结果为：

```
root@b792ae940e3e:/app# celery -B -A tasks worker -l info

[2017-12-10 10:07:36,153: INFO/MainProcess] celery@b792ae940e3e ready.
[2017-12-10 10:07:36,158: INFO/Beat] Scheduler: Sending due task your_task (tasks.add)
[2017-12-10 10:07:36,169: INFO/MainProcess] Received task: tasks.add[d92a952e-e3f6-4224-9e8b-dda771740fe2]  
[2017-12-10 10:07:36,221: INFO/ForkPoolWorker-2] Task tasks.add[d92a952e-e3f6-4224-9e8b-dda771740fe2] succeeded in 0.049944616001084796s: 5
[2017-12-10 10:08:06,148: INFO/Beat] Scheduler: Sending due task your_task (tasks.add)
[2017-12-10 10:08:06,151: INFO/MainProcess] Received task: tasks.add[678c33ae-9d2d-40c1-b5ce-25ffbdf763d8]  
[2017-12-10 10:08:06,175: INFO/ForkPoolWorker-2] Task tasks.add[678c33ae-9d2d-40c1-b5ce-25ffbdf763d8] succeeded in 0.02311466900027881s: 5
[2017-12-10 10:08:36,148: INFO/Beat] Scheduler: Sending due task your_task (tasks.add)
[2017-12-10 10:08:36,149: INFO/Beat] Scheduler: Sending due task my_task (tasks.add)
[2017-12-10 10:08:36,150: INFO/MainProcess] Received task: tasks.add[b05841b8-a382-4a1a-a583-3695e6d369d2]  
[2017-12-10 10:08:36,152: INFO/MainProcess] Received task: tasks.add[baebd820-4a22-496c-8c39-6b9f52d2f22b]  
[2017-12-10 10:08:36,195: INFO/ForkPoolWorker-2] Task tasks.add[b05841b8-a382-4a1a-a583-3695e6d369d2] succeeded in 0.04302837300019746s: 5
[2017-12-10 10:08:36,198: INFO/ForkPoolWorker-3] Task tasks.add[baebd820-4a22-496c-8c39-6b9f52d2f22b] succeeded in 0.0449830910001765s: 56
```

##### 时区

Celery定时任务默认使用UTC时区。我们可以在配置文件中来设置。最终的 `tasks.py` 文件：

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from celery import Celery

app = Celery('tasks',
             broker='amqp://myuser:hello@localhost:5672/hellohost',
             backend='amqp://myuser:hello@localhost:5672/hellohost'
             )


app.conf.update(
    # 配置所在时区
    CELERY_TIMEZONE='Asia/Shanghai',
    CELERY_ENABLE_UTC=True,
    # 官网推荐消息序列化方式为json
    CELERY_ACCEPT_CONTENT=['json'],
    CELERY_TASK_SERIALIZER='json',
    CELERY_RESULT_SERIALIZER='json',
    # 配置定时任务
    CELERYBEAT_SCHEDULE={
        'my_task': {
            'task': 'tasks.add',
            'schedule': 60,
            'args': (22, 34),
        },
        'your_task': {
            'task': 'tasks.add',
            'schedule': 30,
            'args': (1, 4),
        }
    }
)


@app.task
def add(x, y):
    return x + y

```

##### 通过on_after_configure定义

使用 `on_after_configure` 处理程序来装饰定时任务。如 `tasks.py` 文件：

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from celery import Celery

app = Celery('tasks',
             broker='amqp://myuser:hello@localhost:5672/hellohost',
             backend='amqp://myuser:hello@localhost:5672/hellohost'
             )


@app.on_after_configure.connect
def setup_periodic_tasks(sender, **kwargs):
    # Calls test('hello') every 10 seconds.
    sender.add_periodic_task(10.0, add.s('hello'), name='add every 10')


@app.task
def add(arg):
    print(arg)
    return arg

```

执行示例：

```
root@b792ae940e3e:/app# celery -B -A tasks worker -l info

[2017-12-10 10:53:30,420: INFO/MainProcess] celery@b792ae940e3e ready.
[2017-12-10 10:53:30,596: INFO/Beat] beat: Starting...
[2017-12-10 10:53:30,620: INFO/Beat] Scheduler: Sending due task add every 10 (tasks.add)
[2017-12-10 10:53:30,628: INFO/MainProcess] Received task: tasks.add[315383b8-5bd7-48ab-8dd2-c5d39e71f058]  
[2017-12-10 10:53:30,630: WARNING/ForkPoolWorker-3] hello
[2017-12-10 10:53:30,667: INFO/ForkPoolWorker-3] Task tasks.add[315383b8-5bd7-48ab-8dd2-c5d39e71f058] succeeded in 0.03778552899893839s: 'hello'
[2017-12-10 10:53:40,602: INFO/Beat] Scheduler: Sending due task add every 10 (tasks.add)
[2017-12-10 10:53:40,607: INFO/MainProcess] Received task: tasks.add[5f8ef21e-fbfd-4c82-8dbd-6fa2aeaf4fa4]  
[2017-12-10 10:53:40,610: WARNING/ForkPoolWorker-3] hello
[2017-12-10 10:53:40,631: INFO/ForkPoolWorker-3] Task tasks.add[5f8ef21e-fbfd-4c82-8dbd-6fa2aeaf4fa4] succeeded in 0.022007824998581782s: 'hello'
```

***

同时，`add_periodic_task()` 方法也能设置其他参数：

```
    # Calls test('world') every 30 seconds
    sender.add_periodic_task(30.0, add.s('world'), expires=10)
```

或者也能通过 `crontab()` 来应用 `cron` 表达式，实现多种时间的设定。

```
    # Executes every Monday morning at 7:30 a.m.
    sender.add_periodic_task(
        crontab(hour=7, minute=30, day_of_week=1),
        add.s('Happy Mondays!'),
    )
```

#### 相关参考

* [Periodic Tasks](http://docs.celeryproject.org/en/master/userguide/periodic-tasks.html)
* [异步任务神器 Celery ](http://funhacks.net/2016/12/13/celery/)
