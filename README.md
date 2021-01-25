# Build Steps

> poetry init -n
> poetry add django djangorestframework djangorestframework-simplejwt gunicorn django-environ psycopg2-binary django-cors-headers whitenoise
> poetry add --dev black flake8
> poetry shell
> django-admin startproject snack_tracker_api_project .
> touch `heroku.yml`

```yaml
build:
  docker:
    web: Dockerfile
release:
  image: web
run:
  web: gunicorn snack_tracker_api_project.wsgi
```

- **NOTE:** Make sure **.wsgi line matches your project.

> touch Dockerfile

```Dockerfile
# Python version
FROM python:3

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Set work directory
WORKDIR /code

# Install dependencies
COPY requirements.txt /code/
RUN pip install -r requirements.txt

# Copy project
COPY . /code/
```

> poetry export -o requirements.txt --without-hashes

Should look something close to ...

```text
asgiref==3.3.1; python_version >= "3.6"
django-cors-headers==3.6.0; python_version >= "3.6"
django-environ==0.4.5
django==3.1.5; python_version >= "3.6"
djangorestframework-simplejwt==4.6.0; python_version >= "3.7"
djangorestframework==3.12.2; python_version >= "3.5"
gunicorn==20.0.4; python_version >= "3.4"
psycopg2-binary==2.8.6; (python_version >= "2.7" and python_full_version < "3.0.0") or (python_full_version >= "3.4.0")
pyjwt==2.0.1; python_version >= "3.7"
pytz==2020.5; python_version >= "3.6"
sqlparse==0.4.1; python_version >= "3.6"
whitenoise==5.2.0; python_version >= "3.5" and python_version < "4"
```

## project urls.py

```python
from django.contrib import admin
from django.urls import include, path

from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/snacks/', include('snacks.urls')),
    path('api-auth/', include('rest_framework.urls')),
    path('api/v1/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/v1/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

## settings.py

### Installed Apps

```python
INSTALLED_APPS = [

    ...

    #3rd party
    'rest_framework',
    'corsheaders', # new for deployment lab

    # local
    'snacks',

]
```

### Middleware

Below SessionMiddleware and CommonMiddleware

```python
'corsheaders.middleware.CorsMiddleware',
'whitenoise.middleware.WhiteNoiseMiddleware',
```

### REST_FRAMEWORK

```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    )
}
```

### Cors Origins

```python
CORS_ALLOWED_ORIGINS=tuple(env.list('CORS_ALLOWED_ORIGINS', default=[]))
CORS_ALLOW_ALL_ORIGINS=env.bool('CORS_ALLOW_ALL_ORIGINS', default=0)
```

### Database

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': env('DATABASE_NAME'),
        'USER': env('DATABASE_USER'),
        'PASSWORD':  env('DATABASE_PASSWORD'),
        'HOST':  env('DATABASE_HOST'),
        'PORT':  env('DATABASE_PORT')
    }
}
```

### Env stuff

```python
import environ

env = environ.Env(
    DEBUG=(bool, False)
)

# read .env file
environ.Env.read_env()

...

SECRET_KEY = env.str('SECRET_KEY')
DEBUG = env.bool('DEBUG')
ALLOWED_HOSTS = tuple(env.list('ALLOWED_HOSTS'))
```

## .env file

**Note** It goes in project folder NOT root.

```text
DEBUG=on
SECRET_KEY=your_secret_key
DATABASE_NAME=your_database_name
DATABASE_USER=your_database_user
DATABASE_PASSWORD=your_database_password
DATABASE_HOST=your_database_host
DATABASE_PORT=your_database_port
ALLOWED_HOSTS=one_approved_host,another_approved_host
CORS_ALLOWED_ORIGINS=your_origin,your_other_origin
CORS_ALLOW_ALL_ORIGINS=1_OR_0
```

## Snacks Client

Copy in Snacks app from previous snacks-api lab
