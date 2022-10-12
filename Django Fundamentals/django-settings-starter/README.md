# settings.py configuration

<aside>
💡 NOTE: this Django repo is opinionated in that it comes with "drf_yasg" (a 3rd party swagger package) and "django-debug-toolbar" already installed and configured. Also, sqlite for dev and postgresql for prod settings was chosen. Therefore, after cloning this repository, keep those in mind and change the parts to your linking.

</aside>

```python
# Clone this repository
$ git clone https://github.com/MSKose/django-notes

# copy django-settings-starter to your local machine

# Install dependencies
    $ python -m venv env
    > env/Scripts/activate (for win OS)
    $ source env/bin/activate (for macOs/linux OS)
    $ pip install -r requirements.txt

# Add the following to your .env file
    SECRET_KEY=<yourSecretKeyHere>
    DEBUG=True 
    ENV_NAME=dev 
    DEBUG=True 
    SQL_DATABASE=<yourDatabaseProjectName>
    SQL_USER=<yourDatabaseUsername> 
    SQL_PASSWORD=<yourDatabasePassword>
    SQL_HOST=localhost 
    SQL_PORT=5432

# Run the app
    $ python manage.py runserver
```

## Separate development and production settings

- When we start to deploy our Django application to the server or develop a Django application with the team, settings will be a serious problem
- There is no built-in universal way to configure Django settings without hardcoding them. But books, open-source and work projects provide a lot of recommendations and approaches on how to do it best. Let’s take a brief look at the most popular ones to examine their weaknesses and strengths.

### First Solution: Keeping local settings in "settings_local.py”

- The basic idea of this method is to extend all environment-specific settings in the settings_local.py file, which is ignored by VCS.
    - Pros: Secrets not in VCS.
    - Cons: settings_local.py is not in VCS, so you can lose some of your Django environment settings. The Django settings file is a Python code, so settings_local.py can have some non-obvious logic. You need to have settings_local.example (in VCS) to share the default configurations for developers.

### Second Solution: Separate settings file for each environment

- This is an extension of the previous approach. It allows you to keep all configurations in VCS and to share default settings between developers.
- In this case, you make a settings package with the following file structure in your project directory:

```python
.── settings (folder)
│
├── __init__.py
├── base.py
├── dev.py
└── prod.py
```

- Copy all the stuff inside settings.py to base.py. And delete settings.py
- Our base.py would look like this;

```python
# settings/base.py
from pathlib import Path
from decouple import config

# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = Path(__file__).resolve().parent.parent

# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/4.1/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = config("SECRET_KEY")

ALLOWED_HOSTS = ['*']

# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # Third party apps:
    'rest_framework',
		...
		# 'debug_toolbar', # I'll be adding the third party apps I will only be using on development to dev.py, like debug_toolbar. See dev.py below

    # myapps
    'my_first_app',
    'my_second_app',
]

MIDDLEWARE = [
    # 'debug_toolbar.middleware.DebugToolbarMiddleware', # I'll be adding the third party apps I will only be using on development to dev.py, like debug_toolbar. See dev.py below
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'main.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'main.wsgi.application'

# Password validation
# https://docs.djangoproject.com/en/4.1/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

# Internationalization
# https://docs.djangoproject.com/en/4.1/topics/i18n/

LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'UTC'

USE_I18N = True

USE_TZ = True

# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/4.1/howto/static-files/

STATIC_URL = 'static/'

# Default primary key field type
# https://docs.djangoproject.com/en/4.1/ref/settings/#default-auto-field

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

- Hence, dev.py will look like this;

```python
# settings/dev.py
from .base import *

THIRD_PARTY_APPS = ["debug_toolbar"]
DEBUG = config("DEBUG") # we'll be determining whether DEBUG is True or False in our .env file
INSTALLED_APPS += THIRD_PARTY_APPS

THIRD_PARTY_MIDDLEWARE = ["debug_toolbar.middleware.DebugToolbarMiddleware"]
MIDDLEWARE += THIRD_PARTY_MIDDLEWARE # added the debug_toolbar to development-only settings

# I'll be using sqlite for production
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

INTERNAL_IPS = [ 
    "127.0.0.1", 
]
```

- prod.py will look like this;

```python
# settings/prod.py
from .base import *

# I'll be using postgresql for production
DATABASES = { 
    "default": { 
        "ENGINE": "django.db.backends.postgresql_psycopg2", 
        "NAME": config("SQL_DATABASE"), 
        "USER": config("SQL_USER"), 
        "PASSWORD": config("SQL_PASSWORD"), 
        "HOST": config("SQL_HOST"), 
        "PORT": config("SQL_PORT"), 
        "ATOMIC_REQUESTS": True, 
    } 
}
```

- The __init__.py file;

```python
# settings/__init__.py
from .base import *

env_name = config("ENV_NAME")

if env_name == "prod":
    from .prod import *
    
elif env_name == "dev":
    from .dev import *
```

- And finally we'll modify .env file with an environment name, postgres and debug variables;

```python

SECRET_KEY=<yourSecretKeyHere>

DEBUG=True # switch to True when in production
ENV_NAME=dev # switch to prod when in production
SQL_DATABASE=<yourDatabaseName> 
SQL_USER=postgres 
SQL_PASSWORD=postgres 
SQL_HOST=localhost 
SQL_PORT=5432
```

- Don't forget to run migrate command after all these settings

## Logging

- Python programmers will often use `print()` in their code as a quick and convenient debugging tool. Using the logging framework is only a little more effort than that, but it’s much more elegant and flexible. As well as being useful for debugging, logging can also provide you with more - and better structured - information about the state and health of your application.
- Django uses and extends Python’s built-in logging module to perform system logging. This module is discussed in detail in Python’s own documentation; following section provides a quick overview.
- A Python logging configuration consists of four parts:
    - Loggers
    - Handlers
    - Filters
    - Formatters
- Python defines the following log levels:
    - `DEBUG`: Low level system information for debugging purposes
    - `INFO`: General system information
    - `WARNING`: Information describing a minor problem that has occurred.
    - `ERROR`: Information describing a major problem that has occurred.
    - `CRITICAL`: Information describing a critical problem that has occurred
- An example of a logging setting might look like this;

```python
LOGGING = {
    "version": 1,
    # is set to True then all loggers from the default configuration will be disabled.
    "disable_existing_loggers": True,
    # Formatters describe the exact format of that text of a log record. 
    "formatters": {
        "standard": {
            "format": "[%(levelname)s] %(asctime)s %(name)s: %(message)s"
        },
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
        'simple': {
            'format': '{levelname} {message}',
            'style': '{',
        },
    },
    # The handler is the engine that determines what happens to each message in a logger.
    # It describes a particular logging behavior, such as writing a message to the screen, 
    # to a file, or to a network socket.
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "formatter": "standard",
            "level": "ERROR",
            "stream": "ext://sys.stdout",
            },
        'file': {
            'class': 'logging.FileHandler',
            "formatter": "verbose",
            'filename': './debug.log',
            'level': 'INFO', # change this to WARNING and you'll capture warnings, for instance
        },
    },
    # A logger is the entry point into the logging system.
    "loggers": {
        "django": {
            "handlers": ["console", 'file'],
            # log level describes the severity of the messages that the logger will handle. 
            "level": config("DJANGO_LOG_LEVEL", "INFO"),
            'propagate': True,
            # If False, this means that log messages written to django.request 
            # will not be handled by the django logger.
        },
    },
} 
```

- If you have separated your dev and prod settings into two files, then it makes sense to copy the above code into your dev settings
- After having set a LOGGING to your project, it will automatically generate a debug.log file and that file is where you'll see your log saves.

## Django Settings: Best practices

- Keep settings in environment variables.
- Write default values for production configuration (excluding secret keys and tokens).
- Don’t hardcode sensitive settings, and don’t put them in VCS.
- Split settings into groups: Django, third-party, project.
- Follow naming conventions for custom (project) settings