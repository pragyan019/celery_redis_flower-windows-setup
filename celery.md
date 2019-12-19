### Setting up Celery,Redis and Flower in windows

Redis don't have a native support for windows. 

**System Requirements**
* Anaconda 
* Docker

**Package Requirements**

* conda install -c conda-forge celery
* pip install redis
* pip install flower
  
**Steps**
* In project's __init__.py add
```python
from __future__ import absolute_import, unicode_literals

# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from .celery import app as celery_app

__all__ = ('celery_app',)

```
* Add a file at django project level called celery.py and paste the following code
```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', '####.settings')#replace #### with your project name
os.environ.setdefault('FORKED_BY_MULTIPROCESSING', '1')

app = Celery('####')#replace #### with your project name

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
* In django's settings.py add
```python
CELERY_BROKER_URL = 'redis://127.0.0.1:6379/'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
```
* In your app,add a file tasks.py and add following snnipet to it
```python
from celery import shared_task
from time import sleep

@shared_task
def your_task():
    sleep(10)
    #body
    return None
```
* In views.py of same app
```python
from .tasks import your_task
........
............
#call your task
your_task.delay()
```
* pull redis docker image
  ```python
  docker pull redis
  ```
* map the docker default port 6379 for redis to local host port 6379
  ```python
  docker run -d -p 6379:6379 redis
  ```
* check for running container
```python
docker ps
```
* Open 3 separate command prompts
* Run your django server
```python
python manage.py runserver
```
* Run your Celery server
```python
celery -A project_name worker -l info 
```
* Run your Flower server
```python
flower -A project_name --port=5555 
``` 
* Open http://localhost:5555/ for flower admin panel