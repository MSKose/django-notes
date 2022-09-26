# django-starter


### Installation

```python
# 1. step (creating an environment)
python -m venv env

# 2. step (activating)
source env/bin/activate  (env/Scripts/activate for Windows)

# extra step if you want to deactivate your environment
deactivate

# to activate back:
source env/bin/activate

# this command should return nothing. If it returns something as of now it could mean you are not running the new environment
pip freeze

# 3. step (make sure you are activated)
pip install django

# 4. step (creating a project)
django-admin startproject <yourprojectname> .

# a one-liner with a project called main:
python -m venv env && source env/bin/activate && pip install django && django-admin startproject main .

# 5. creating an app
python manage.py startapp <yourappname>

# running the server
python manage.py runserver
```

- Setting up a .env file for your secret key. See [here](https://pypi.org/project/python-decouple/#example-how-do-i-use-it-with-django) more details

```python
# first, install this into your virtual environment
pip install python-decouple

# open a .env file and copy paste your SECRET_KEY and its value here
# remove the quotation marks and all the white spaces available from your secret key and add the following import into <project>/settings.py:
from decouple import config

# finally, add this:
SECRET_KEY = config("SECRET_KEY")
```

- Open a .gitignore file and paste [this](https://www.toptal.com/developers/gitignore/api/django) (or [this](https://www.toptal.com/developers/gitignore/api/django,macos) for MacOs)into it
- Add your app name as a string to your INSTALLED_APPS list in projectname/settings.py
- After setting up your dependencies run these command:

```python
# to see the installed dependencies:
pip freeze

# to save the dependencies (run this command everytime you install a new dependency to make it up-to-date with your dependencies):
pip freeze > requirements.txt

# to run on the current depencies.txt after, say, cloning a project from github
pip install -r requirements.txt
```