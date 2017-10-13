---
layout: post
title:  "Executing time-consuming tasks asynchronously with Django and Celery"
date:   2017-08-26
author:     "Kyle Ding"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags: 
    - Python
    - Django
    - Celery
---

## What’s the matter of having time-consuming tasks on the server side?

Every time the client makes a request, the server has to read the request, parse the received data, retrieve or create something into the database, process what the user will receive, renders a template and send a response to the client. That’s usually what happens in a Django app.



Depending on what you are executing on the server, the response can take too long and it leads to problems such as poor user experience or even a time-out error. It’s a big issue. [Loading time is a major contributing factor to page abandonment](https://blog.kissmetrics.com/loading-time/?wide=1). So, slow pages lose money.

There’s a lot of functions that can take a long time to run, for instance, a large data report requested by a web client, emailing a long list or even editing a video after it’s uploaded on your website.

## Real Case:

That’s a problem that I’ve face once when I was creating a report. The report took around 20 minutes to be sent and the client got a time-out error and obviously, nobody wants to wait 20 minutes for getting something. So, to handle this I’d have to let the task run in the background. (On Linux, you can do this putting a & at the end of a command and the OS will execute the command in the background)

[![django-view-os-system](https://i1.wp.com/fernandofreitasalves.com/wp-content/uploads/2015/07/tabelao2.jpg?resize=700%2C296)](https://i1.wp.com/fernandofreitasalves.com/wp-content/uploads/2015/07/tabelao2.jpg)

It looks like the worst code ever.

## Celery – the solution for those problems!

[Celery ](http://www.celeryproject.org/)is a distributed system to process lots of messages. You can use it to run a task queue (through messages). You can schedule tasks on your own project, without using crontab and it has an easy integration with the major Python frameworks.

## How does celery work?

Celery Architecture Overview. (from this [SlideShare](https://www.slideshare.net/idangazit/an-introduction-to-celery/11-Celery_Architecture_AMQP_celery_task))

[![estrutura-celery](https://i1.wp.com/fernandofreitasalves.com/wp-content/uploads/2015/07/estrutura-celery1.jpg?resize=700%2C521)](https://i1.wp.com/fernandofreitasalves.com/wp-content/uploads/2015/07/estrutura-celery1.jpg)

- The User (or Client or Producer) is your Django Application.
- The AMPQ Broker is a Message Broker. A program responsible for the message queue, it receives messages from the Client and delivers it to the workers when requested. For Celery the AMPQ Broker is generally [RabbitMQ](http://www.rabbitmq.com/) or [Redis](http://redis.io/)
- The workers (or consumers) that will run your tasks asynchronously.
- The Result Store, a persistent layer where workers store the result of tasks.


The client produces messages, deliver them to the Message Broker and the workers read this messages from the broker, execute them and can store the results on a Memcached, RDBMS, MongoDB, whatever the client can access later to read the result.

## Installing and configuring RabbitMQ

There is a lot of examples on How to Use Celery with Redis. I’m doing this with RabbitMQ.

1. Install RabbitMQ

   ```shell
   sudo apt-get install rabbitmq-server
   ```

2. Create a User, a virtual host and grant permissions for this user on the virtual host:
   ```shell
   sudo rabbitmqctl add_user myuser mypassword
   sudo rabbitmqctl add_vhost myvhost
   sudo rabbitmqctl set_permissions -p myvhost myuser ".*" ".*" ".*"
   ```
> Without RabbitMQ, you can use redis taken broker.

## Installing and configuring Celery

```shell
pip install celery
```
In your settings.py:
```python
#Celery Config
BROKER_URL = 'amqp://guest:guest@localhost:5672//'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_RESULT_BACKEND = 'db+sqlite:///results.sqlite'
CELERY_TASK_SERIALIZER = 'json'
```

In your project’s directory (the same folder of settings.py), creates a **celery.py** file as following.

```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'shop.settings')

app = Celery('shop')

# Using a string here means the worker doesn't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```
## Creating tasks for your app

In your app’s directory, put a **tasks.py**:

```python
from celery import shared_task

@shared_task
def auto_meal_leave(meal):
    meal = Meal.objects.filter(id=meal)
    if meal.exists():
        meal = meal.first()
        order = Order.objects.filter(meal=meal)
        for o in order:
            if o.status > 0:
                return True
        meal.status = 1
        meal.save()
        return True
    else:
        return True
```

Now you just have to import this function anywhere you want and call the delay method, that was added by the shared_task decorator.

```python
from tasks import generate_report
@login_required()
def my_view(request):
    ...
    generate_report.delay(ini_date, final_date, email)
    or
    generate_report.apply_async((meal.id,),eta=datetime.utcnow() + timedelta(minutes=5))
    return "You will receive an email when the report is done"
```
> The [`delay()`](http://docs.celeryproject.org/en/latest/reference/celery.app.task.html#celery.app.task.Task.delay) method is convenient as it looks like calling a regular function
>
> Using [`apply_async()`](http://docs.celeryproject.org/en/latest/reference/celery.app.task.html#celery.app.task.Task.apply_async) instead you have to write all the args.

So delay is clearly convenient, but if you want to set additional execution options you have to use `apply_async`.

The ETA (estimated time of arrival) lets you set a specific date and time that is the earliest time at which your task will be executed. countdown is a shortcut to set ETA by seconds into the future.



## Running celery workers

Now you have to run the celery workers so they can execute the tasks getting the messages from the RabbitMQ Broker. By default, celery starts one worker per available CPU. But you can change it using the concurrency parameter (-c)

```shell
celery --app=nome_projeto worker --loglevel=INFO
```

And your screen will look like this:

[![celery-rodando](https://i1.wp.com/fernandofreitasalves.com/wp-content/uploads/2015/07/celery-rodando.jpg?resize=700%2C271)](https://i1.wp.com/fernandofreitasalves.com/wp-content/uploads/2015/07/celery-rodando.jpg)

In another terminal you can open the shell and call your task to see it working:

 

And you will see something like this on Celery:

[![aparece no celery](https://i1.wp.com/fernandofreitasalves.com/wp-content/uploads/2015/07/aparece-no-celery.jpg?resize=700%2C42)](https://i1.wp.com/fernandofreitasalves.com/wp-content/uploads/2015/07/aparece-no-celery.jpg)

## Deploying Celery

To use celery in production you’ll need a process control system like [Supervisor](http://supervisord.org/)

To install supervisor:

Write a run celery script:

```shell
#!/bin/bash

NAME="celery"                                  # Name of the application
CELERYDIR=/shop/shop/shop             # Celery shell file directory
USER=root                                        # the user to run as
GROUP=root                                     # the group to run as


echo "Starting $NAME as `root`"

# Activate the virtual environment
cd $CELERYDIR
source ../bin/activate


# Start your Celery worker`
# Programs meant to be run under supervisor should not daemonize themselves (do not use --daemon)
echo "Starting celery worker..."
exec ../bin/celery -A shop worker -l info
```

Now you have to create a configuration file for celery in */etc/supervisor/conf.d/*


```shell
[program:celeryd]
command=/shop/shop/shop/celery_start
stdout_logfile=/var/log/celeryd.log
stderr_logfile=/var/log/celeryd_err.log
autostart=true
autorestart=true
startsecs=10
stopwaitsecs=600
redirect_stderr = true
```



Now inform Supervisor that there is a new process.