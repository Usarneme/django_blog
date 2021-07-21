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

> TIME_ZONE = 'America/Los_Angeles'
> LANGUAGE_CODE = 'en-us'

- Update mysite/settings.py to add a path for serving static files (at the bottom of the file)

> STATIC_URL = '/static'
> STATIC_ROOT = os.path.join(BASE_DIR, 'static')

- Update mysite/settings.py to ensure localhost (and later your online host domain) is allowed

> ALLOWED_HOSTS = ['127.0.0.1', 'localhost']

- Set up a database

Here I'll use sqlite3. Confirm this is in mysite/settings.py:

> DATABASES = {
> 'default': {
> 'ENGINE': 'django.db.backends.sqlite3',
> 'NAME': os.path.join(BASE_DIR, 'db.sqlite3')
> }
> }

- Create the database using Python migration command

```sh
python manage.py migrate
```

Which should confirm 'Ok' after each of the migration commands ran in the terminal. You can now start the web server with:

```sh
python manage.py runserver 0.0.0.0:8080
```

Open your browser of choice and navigate to http://localhost:8080 to view your started Django web app project!

---

## Setup/Clone instructions

- `git clone https://github.com/Usarneme/django_blog`
- `cd django_blog`

---

TODO
