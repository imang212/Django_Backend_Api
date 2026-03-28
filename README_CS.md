## Django aplikace

Backend webová aplikace vytvořená pomocí django frameworku a sqlite databáze v rámci úkolu.

### Postup

Nejdřív si vytvořím django project například s názvem: todo_project, pak v todo_project složce startapp s názvem tasks.
```shell
django-admin startproject todo_project
cd todo_project
django-admin startapp tasks
```
Přidal jsem "tasks" a "rest_framework" do souboru settings.py
```python
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
Přešel jsem do vytvořené "tasks" složky Natavil model tabulky v databázi v models.py.
```python
from django.db import models
class Task(models.Model):
    title = models.CharField(max_length=100)
    description = models.TextField(max_length=500, blank=True, null=True)
    due_date = models.DateField(blank=True, null=True)
    photo = models.ImageField(upload_to="tasks_photos/", blank=True, null=True)

    def __str__(self):
        return self.title
```
Udělám migraci, aby se mi vytvořila daná tabulka.
```shell
python manage.py makemigrations tasks
python manage.py migrate
```

Udělal serializaci modelu pro nastavení vracení dat v serializers.py.
```python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = "__all__"
```
Přidal jsem GET metodu pro vrácení seznamu úkolů z databáze a další metody do views.py závislých na serializers.py a models.py.
```python
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
Přidal jsem do urls.py cesty k daným metodám v tasks složce a zároveň i v todo_project složce do urls.py cestu do app a k django adminovi.
```python
from django.urls import path
from .views import task_list

urlpatterns = [
    path("tasks/", task_list, name="task-list"),
]
```
V todo_project složce.:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("tasks.urls")),
]
```
Udělal jsem si docker-compose, Dockerfile danému k projektu a vyplnil jsem requirements.txt potřebnými knihovnami, kde mi funguje přes gunicorn a spustil jsem ho v dockeru. Běží to v python 3.11.

Inspirace ze stránky: https://testdriven.io/blog/dockerizing-django-with-postgres-gunicorn-and-nginx/

requirements.txt
```
gunicorn==23.0.0
django==5.1.6
djangorestframework==3.15.2
pillow==11.1.0
psycopg2-binary==2.9.10
dj-database-url==2.3.0
numpy==2.2.3
pillow==11.1.0
opencv-python==4.11.0.86
requests==2.32.3
```
Psaní v příkazové řádce v dané složce.
```shell
docker ps
docker-compose build
docker-compose up -d
docker exec -it django_app bash
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
exit
```

Tam jsem také musel umožnit migrace a vytvořil super uživatele.



### 1. úkol (GET)
Vrací na stránkách: "http://localhost:8000/tasks/" všechny úkoly z sqlite db. Výpis úkolů si je i možno zobrazit v html, přidal jsem tasks_list: "http://localhost:8000/tasks_list"

### 2. úkol (POST)
Když pošlu úkol a uložím ho do sqlite databáze.
Vyzkoušel jsem ho pomocí spuštění python scriptu "make_post.py" ve složce projektu.
Následně jsem uložil do databáze úkol i s jeho obrázkém vybraným z měho PC, kde se v django složce uložil obrázek: "http://localhost:8000/media/tasks_photos/1013511" a můžu si ho na webu normálně otevřít. Zároveň i úkol mohu vidět na "http://localhost:8000/tasks/", kde mi ho to vrátí pomocí GET metody.

### 3. úkol (GET podle id)
Vrátí mi na stránce úkol podle daného id např.:úkol s id 1"http://localhost:8000/tasks/1/" mi vrátí úkol s id 1. Pokud dané id neexistuje zase mi to vrátí chybu 404 stránka neexistuje.

### 4. úkol (PUT podle id)
Aktualizuji úkol pomocí PUT operace, kterou vyzkouším pomocí scriptu "test_update.py". Můžu například změnit jenom jednu informaci nebo i víc informací úkolu

### 5. úkol (DELETE podle id)
Smažu úkol pomocí DELETE operace podle zadaného id úkolu, ta je také dohromady v jedné funkci s GET a PUT operací. Vyzkouším jí pomocí spuštění skriptu "test_delete.py"

### 6. úkol (GET nearest deadline)
GET operace, která mi vrátí z databáze úkol s nejblišším deadline datumem od dnešního dne. Můžu si ji načíst pomocí "http://localhost:8000/tasks/nearest-deadline/".
### LeetCode úlohy

### 7. úkol (POST rotate array)
POST operace, kam zadám pole hodnot a počet rotací k. Vrátí mi(spočítá) rotované pole a vrátí v JSON.

### 8. úkol (POST kth largest)
POST operace, kam zadám pole hodnot a počet k rotací. Vrátí mi(spočítá) JSON s ktým největší prvkem pomocí heap algoritmu.

### 9. úkol (POST longest increasing path)
POST operace, kam zadám 2D pole (matici) celých čísel a spočítá mi délku nejdelší rostoucí cesty pomocí dfs algoritmu

Všechny tyto 3 leetcode úlohy testuji pomocí skriptu "tst_rotate_array.py"
