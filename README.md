# Django_Blog

A blog website built using Python+Django

---

## Build from scratch instructions

- Set up a virtual environment to have consistent versions of your dependencies in your project

```sh
python3 venv NAME
```

Replacing NAME with your project’s name.

Confirm this works with:

```sh
python —version
```

And it should default to python3 since that is what you used to create the venv

- Confirm PIP is updated

```sh
python -m pip install —upgrade pip
```

- Create requirements.txt inside your dir (sibling of venv/) (aka package.json)

```sh
echo Django~=2.2.4 >> requirements.txt
```

- Install Django

```sh
pip install -r requirements.txt
```

- Create Django files and folders boilerplate

```sh
django-admin startproject mysite .
```

NOTE: manage.py script that helps with management of the site, start the server, etc.; settings.py contains website configuration; urls.py contains a list of patterns used by `urlresolver`

- Update mysite/settings.py to include your time zone and default language

Visit https://en.wikipedia.org/wiki/List_of_tz_database_time_zones to find the time zone format, eg:

```
TIME_ZONE = 'America/Los_Angeles'
LANGUAGE_CODE = 'en-us'
```

- Update mysite/settings.py to add a path for serving static files (at the bottom of the file)

```
STATIC_URL = '/static'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```

- Update mysite/settings.py to ensure localhost (and later your online host domain) is allowed

```
ALLOWED_HOSTS = ['127.0.0.1', 'localhost']
```

- Set up a database

Here I'll use sqlite3. Confirm this is in mysite/settings.py:

```
DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.sqlite3',
    'NAME': os.path.join(BASE_DIR, 'db.sqlite3')
  }
}
```

- Create the database using Python migration command

```sh
python manage.py migrate
```

Which should confirm 'Ok' after each of the migration commands ran in the terminal. You can now start the web server with:

```sh
python manage.py runserver 0.0.0.0:8080
```

Open your browser of choice and navigate to http://localhost:8080 to view your started Django web app project!

- Create the blog boilerplate using the startapp command (see: https://docs.djangoproject.com/en/3.2/ref/django-admin/#startapp)

```sh
python manage.py startapp blog
```

- Configure Django to use the blog application created with the last command by editing mysite/settings.py

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # NEW CONTENT BELOW THIS LINE
    'blog.apps.BlogConfig',
]
```

- Define first model in blog/models.py

```py
from django.conf import settings
from django.db import models
from django.utils import timezone


class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    published_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title
```

- Generate a database migration to add our newly-created Post model

```sh
python manage.py makemigrations blog
```

- Update the database using the newly-created migration

```sh
python manage.py migrate blog
```

- Rewrite blog/admin.py so we can administrate (CRUD) our blog model as administrators

```py
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

- Create a superuser account to access admin capabilities. Follow the prompts to create the user after running this command in your terminal:

```sh
python manage.py createsuperuser
```

NOTE: You can now log in by running the server and visiting localhost:8000/admin. The admin view also displays your Blog function and Posts class so you can create new posts. Create a few blog posts to test the functionality.

- Update mysite/urls.py to import defined route handlers from your blog site

```py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
]
```

- Create the imported blog urls file at blog/urls.py, add path import and blog views reference, and our first view the Posts list

```py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.post_list, name='post_list'),
]
```

- Define the referenced view in blog/views.py

```py
def post_list(request):
    return render(request, 'blog/post_list.html', {})
```

- Create a templates folder within blog to hold all of our html templates

```sh
mkdir blog/templates
```

- Create the blog directory within templates to hold our views templates for the blog route

```sh
mkdir blog/templates/blog
```

- Create the first template html file within blog/templates/blog

```sh
touch blog/templates/blog/post_list.html
```

- Add some placeholder markup to your blog/templates/blog/post_list.html file

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Blog Home</title>
  </head>

  <body>
    <header>
      <h1><a href="/">Blog Home</a></h1>
    </header>
  </body>
</html>
```

---

## Setup/Clone instructions

- Clone the repository: `git clone https://github.com/Usarneme/django_blog`
- Enter the new directory: `cd django_blog`
- Ensure myvenv is setup correctly: `python3 venv myvenv`
- Install dependencies: `pip install -r requirements.txt`
- Run the server: `python manage.py runserver`
- Open your browser to localhost:8080 to view the project

---

TODO
