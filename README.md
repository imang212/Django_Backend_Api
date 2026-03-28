# Django Application
Backend web application created using the Django framework and SQLite
database as part of an assignment.

## Setup Process
First, create a Django project (e.g., named `todo_project`), then create
an app inside it called `tasks`.

``` shell
django-admin startproject todo_project
cd todo_project
django-admin startapp tasks
```

Add `"tasks"` and `"rest_framework"` to `settings.py`:

``` python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "rest_framework",
    "tasks",
]
```

Then define the database model in `models.py` inside the `tasks` folder:

``` python
from django.db import models

class Task(models.Model):
    title = models.CharField(max_length=100)
    description = models.TextField(max_length=500, blank=True, null=True)
    due_date = models.DateField(blank=True, null=True)
    photo = models.ImageField(upload_to="tasks_photos/", blank=True, null=True)

    def __str__(self):
        return self.title
```

Run migrations to create the database table:

``` shell
python manage.py makemigrations tasks
python manage.py migrate
```

Create a serializer in `serializers.py`:

``` python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = "__all__"
```

Add a GET method in `views.py`:

``` python
from rest_framework.response import Response
from rest_framework.decorators import api_view
from .models import Task
from .serializers import TaskSerializer

@api_view(["GET"])
def task_list(request):
    tasks = Task.objects.all()
    serializer = TaskSerializer(tasks, many=True)
    return Response(serializer.data)
```

Configure URLs in `tasks/urls.py`:

``` python
from django.urls import path
from .views import task_list

urlpatterns = [
    path("tasks/", task_list, name="task-list"),
]
```

And in `todo_project/urls.py`:

``` python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("tasks.urls")),
]
```

## Docker Setup
Created `docker-compose.yml`, `Dockerfile`, and filled
`requirements.txt` with necessary libraries.\
The app runs using Gunicorn inside Docker with Python 3.11.

Inspiration:\
https://testdriven.io/blog/dockerizing-django-with-postgres-gunicorn-and-nginx/

### requirements.txt
```
    gunicorn==23.0.0
    django==5.1.6
    djangorestframework==3.15.2
    pillow==11.1.0
    psycopg2-binary==2.9.10
    dj-database-url==2.3.0
    numpy==2.2.3
    opencv-python==4.11.0.86
    requests==2.32.3
```

### Docker Commands
``` shell
docker ps
docker-compose build
docker-compose up -d
docker exec -it django_app bash
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
exit
```

# API Tasks
## 1. Task (GET)
Returns all tasks from the SQLite database at:
`http://localhost:8000/tasks/`

HTML view available at: `http://localhost:8000/tasks_list`

## 2. Task (POST)
Saves a new task into the SQLite database.\
Tested using the script `make_post.py`.

The uploaded image is stored in:
`http://localhost:8000/media/tasks_photos/`

## 3. Task (GET by ID)
Returns a specific task by ID, e.g.: `http://localhost:8000/tasks/1/`

Returns 404 if the ID does not exist.

## 4. Task (PUT by ID)
Updates a task using PUT.\
Tested using `test_update.py`.

## 5. Task (DELETE by ID)
Deletes a task by ID.\
Tested using `test_delete.py`.

## 6. Task (GET nearest deadline)
Returns the task with the nearest upcoming deadline:
`http://localhost:8000/tasks/nearest-deadline/`

# LeetCode Tasks
## 7. Task (POST rotate array)
Accepts an array and rotation count `k`.\
Returns the rotated array in JSON format.

## 8. Task (POST kth largest)
Accepts an array and number `k`.\
Returns the k-th largest element using a heap algorithm.

## 9. Task (POST longest increasing path)
Accepts a 2D integer matrix.\
Returns the length of the longest increasing path using DFS algorithm.

All three LeetCode tasks are tested using the script:
`tst_rotate_array.py`
