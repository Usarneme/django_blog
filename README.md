# Django_Blog

A blog website built using Python+Django

---

## Build from scratch instructions

-> Set up a virtual environment to have consistent versions of your dependencies in your project
> ~ `python3 venv NAME`
Replacing NAME with your project’s name.
Confirm this works with 
> ~ `python —version`
And it should default to python3 since that is what you used to create the venv 

-> Confirm PIP is updated
> ~ `python -m pip install —upgrade pip`

-> Create requirements.txt inside your dir (sibling of venv/) (aka package.json)
> ~ `echo Django~=2.2.4 >> requirements.txt`

-> Install Django
> ~ `pip install -r requirements.txt`

-> Create Django files and folders boilerplate

```sh
touch manage.py
mkdir mysite
touch mysite/__init__.py
touch mysite/settings.py
touch mysite/urls.py
touch mysite/wsgi.py
```

manage.py script that helps with management of the site, start the server, etc.
settings.py contains website configuration
urls.py contains a list of patterns used by `urlresolver`

---

## Setup/Clone instructions

- `git clone https://github.com/Usarneme/django_blog`
-  `cd django_blog`


---

TODO
