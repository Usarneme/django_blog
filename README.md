# Django_Blog

A blog website built using Python+Django

---

## Build from scratch instructions

- Set up a virtual environment to have consistent versions of your dependencies in your project

```sh
python3 -m venv NAME
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

- Update our blog/views.py to include our Post model, the result of our queryset on the db, which is then passed to the render function

```py
from django.shortcuts import render
from django.utils import timezone
from .models import Post

def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})
```

- Create a statics folder in the blog

```sh
mkdir blog/static
```

- Create a CSS file in the static folder

```sh
mkdir blog/static/css
touch blog/static/css/blog.css
```

- Create a base html file to hold layout for multiple pages at blog/templates/blog/base.html

```sh
touch blog/templates/blog/base.html
```

- Add content to the base.html file, including loading the static folder/files at the very top of the blog/templates/blog/post_list.html and the CSS styles via file reference between the opening and closing <head> tags:

```html
{% load static %}
<!DOCTYPE html>
<html>
  <head>
    <title>Blog Home</title>
    <link rel="stylesheet" href="{% static 'css/blog.css' %}" />
  </head>

  <body>
    <header class="page-header">
      <div class="container">
        <h1><a href="/">Django Blog</a></h1>
      </div>
    </header>
    <main>{% block content %} {% endblock %}</main>
  </body>
</html>
```

- Create the first template html file within blog/templates/blog

```sh
touch blog/templates/blog/post_list.html
```

- Add some markup to your blog/templates/blog/post_list.html file to display each of the posts individually inside of the layout container

```html
{% extends 'blog/base.html' %} {% block content %} {% for post in posts %}
<article>
  <time>published: {{ post.published_date }}</time>
  <h2><a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a></h2>
  <p>{{ post.text|linebreaksbr }}</p>
</article>
{% endfor %} {% endblock %}
```

NOTE: `post.text|linebreaksbr` pipes the post text into a Django filter method that replaces python newline characters with html <br> tags. See: https://www.djangotemplatetagsandfilters.com/filters/linebreaksbr/

NOTE: pk=post.pk is primary key, this sets the link address to be the primary key of the post so each link will be unique and tied to the post ID.

- Create a url by updating the urlpatterns matther in blog/urls.py for the post_detail view

```py
urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
]
```

NOTE: the integer in the url will be matched dynamically and passed to the view as a variable named pk

- Add a post_details view in blog/views.py

```py
def post_detail(request, pk):
  post = get_object_or_404(Post, pk=pk)
  return render(request, 'blog/post_detail.html', {'post': post})
```

NOTE: get_object_or_404 is a Django method that will return the 404/not found page if there is no post matching at primary key (or in general, no matcher for that route)

- Create the post_details html template in blog/templates/blog/post_detail.html

```html
{% extends 'blog/base.html' %} {% block content %}
<article class="post">
  <aside class="actions">
    <a class="btn btn-default" href="{% url 'post_edit' pk=post.pk %}">
      Edit Post
    </a>
  </aside>
  {% if post.published_date %}
  <time class="date"> {{ post.published_date }} </time>
  {% endif %}
  <h2>{{ post.title }}</h2>
  <p>{{ post.text|linebreaksbr }}</p>
</article>
{% endblock %}
```

- Create a forms.py file to add our own create and update blog post form

```sh
touch blog/forms.py
```

- Add ViewModel content to blog/forms.py

```py
from django import forms
from .models import Post


class PostForm(forms.ModelForm):

    class Meta:
        model = Post
        fields = ('title', 'text',)
```

NOTE: PostForm is the name of the new class, forms.ModelForm is Django connecting the view to the model for us, and the Meta class ties this viewmodel to our Post model and tells Django which input fields to expose (since author is logged-in user and creation_date is timezone.now())

- Create an icons folder and save an icon svg to it

```sh
mkdir blog/templates/blog/icons
echo "<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"16\" height=\"16\" fill=\"currentColor\" class=\"bi bi-file-earmark-plus\" viewBox=\"0 0 16 16\">
  <path d=\"M8 6.5a.5.5 0 0 1 .5.5v1.5H10a.5.5 0 0 1 0 1H8.5V11a.5.5 0 0 1-1 0V9.5H6a.5.5 0 0 1 0-1h1.5V7a.5.5 0 0 1 .5-.5z\"/>
  <path d=\"M14 4.5V14a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V2a2 2 0 0 1 2-2h5.5L14 4.5zm-3 0A1.5 1.5 0 0 1 9.5 3V1H4a1 1 0 0 0-1 1v12a1 1 0 0 0 1 1h8a1 1 0 0 0 1-1V4.5h-2z\"/>
</svg>" >> blog/templates/blog/icons/file-plus.svg
```

- Add the icon and a link to create a new blog post to blog/templates/blog/base.html

```html
<a href="{% url 'post_new' %}" class="top-menu">
  {% include './icons/file-plus.svg' %}
</a>
```

- Add the path to the post_new view in the urlpatterns array in blog/urls.py

```py
path('post/new/', views.post_new, name='post_new')
```

- Implement the post_new view in blog/views.py

````py
# near the top add the import
from .forms import PostForm
# other content unchanged...
# add the post_new function definition
def post_new(request):
  form = PostForm()
  return render(request, 'blog/post_edit.html', {'form': form})

- Create the post_edit.html template in blog/templates/blog

```sh
touch blog/templates/blog/post_edit.html
````

- Add html to the newly created blog/templates/blog/post_edit.html template

```html
{% extends 'blog/base.html' %} {% block content %}
<h2>New post</h2>
<form method="POST" class="post-form">
  {% csrf_token %} {{ form.as_p }}
  <button type="submit" class="save btn btn-default">Save</button>
</form>
{% endblock %}
```

NOTE: Cross Site Request Forgery (CRSF) tokens explained: https://stackoverflow.com/questions/5207160/what-is-a-csrf-token-what-is-its-importance-and-how-does-it-work Django will complain if we do not include this directive

- Update the post_new function in blog/views.py to handle submitting the post form

```py
# add the redirect import to the top
from django.shortcuts import redirect
# rest of the code is unchanged until we get to post_new
def post_new(request):
  if request.method == "POST":
    form = PostForm(request.POST)
    if form.is_valid():
        post = form.save(commit=False)
        post.author = request.user
        post.published_date = timezone.now()
        post.save()
        return redirect('post_detail', pk=post.pk)
  else:
    form = PostForm()
  return render(request, 'blog/post_edit.html', {'form': form})
```

- Add matcher to blog/urls.py for the Edit Post route

```py
path('post/<int:pk>/edit/', views.post_edit, name='post_edit'),
```

- Add route handler to blog/views.py to server the edit post template

```py
def post_edit(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == "POST":
        form = PostForm(request.POST, instance=post)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    return render(request, 'blog/post_edit.html', {'form': form})
```

NOTE: Instance passed to PostForm will populate the fields of the form from the values retrieved from the database with the get_object_or_404 query. See: https://docs.djangoproject.com/en/2.2/topics/forms/#instantiating-processing-and-rendering-forms

- Lock down post_new anchor link to only be visible for logged in users in blog/templates/blog/base.html

```html
{% if user.is_authenticated %}
<a href="{% url 'post_new' %}" class="top-menu">
  {% include './icons/file-plus.svg' %}
</a>
{% endif %}
```

- Also lock down the post_edit link in blog/templates/blog/post_detail.html

```html
{% if user.is_authenticated %} ... {% endif %}
```

- Add a link to the admin page so you can log in and out without typing in the url by editing blog/templates/blog/base.html

```html
<a href="/admin" class="top-menu">Admin</a>
```

---

## Setup/Clone instructions

- Clone the repository: `git clone https://github.com/Usarneme/django_blog`
- Enter the new directory: `cd django_blog`
- Ensure myvenv is setup correctly: `python3 -m venv myvenv`
- Install dependencies: `pip install -r requirements.txt`
- Run the server: `python manage.py runserver`
- Open your browser to localhost:8080 to view the project

---

### LICENSE

GPLv3

---

### Attribution/Credits

The majority of the code in this repository is from the Django Girls tutorial: [https://tutorial.djangogirls.org/en/](https://tutorial.djangogirls.org/en/). Which is licensed under Creative Commons Attribution-ShareAlike 4.0 International License. To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/4.0/

I added styles, the logout features, and will continue to add other features as I work to expand this project and learn more about Django.

---

### Thanks!
