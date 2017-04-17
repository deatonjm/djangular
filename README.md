# djangular
My first attempt at setting up a Django/Angular project from scratch. The setup instructions are based on the [instructions here](http://gregblogs.com/how-the-do-i-build-a-django-django-rest-framework-angular-1-1-x-and-webpack-project/#prereqs), but include additional notes and changes to some of the instructions that I had to make in order to get things setup without error.

## Installation
All of the following was done using Git Bash. Software that was already installed before starting: Pyhon 2.x and Node.js. and virtualenv.

1. Create a new virtualenv called 'djangular' in your virtualenvs dir
```
$ virtualenv -p C:/Python27/python.exe djangular
```
2. Create a new directory for your project to live in called 'djangular'
```
Tree:
c/
  Users/
      me/
          djangular/
          virtualenvs/
              djangular/
```
3. Activate your 'djangular' virtualenv by running the following in /me/virtualenvs/djangular/
```
$ . Scripts/activate
```
4. Navigate into your project folder at /me/djangular/ and pip install the following:
* Django `$ pip install django==1.9`
* ESLint `$ npm install -g eslint`
* Webpack `$ npm install --save-dev webpack`
* Django-environ `$ pip install django-environ`
* Django REST Framework `$ pip install djangorestframework`

5. Navigate back to your project folder at /me/djangular/ and create the following scaffolding (hit return when line ends):
``` 
$ mkdir -p backend/docs/ backend/requirements/ \  
         frontend/app/shared/ \
         frontend/app/components/ \
         frontend/config \
         frontend/assets/img/ frontend/assets/css/ \
         frontend/assets/js/ frontend/assets/libs/ \
         frontend/dist/js/
```
```
your Tree should now look like:

c/
  Users/
      me/
          djangular/
              backend/
                  docs/
                  requirements/
              frontend/
                  app/
                      components/
                      shared/
                  assets/
                      css/
                      img/
                      js/
                      libs/
                  config/
                  dist/
                      js/
```
5. Navigate into /djangular/backend/ and create a project called mysite
```
$ django-admin.py startproject mysite .
```
6. Still within /djangular/backend/, create a requirements.txt file in the requirements dir
```
$ pip freeze > requirements/requirements.txt  
```
7. Go into /djangular/backend/mysite/ and scaffold your API dirs (hit return when line ends):
```
$ mkdir -p applications/api/v1/  
$ touch applications/__init__.py applications/api/__init__.py \  
        applications/api/v1/__init__.py applications/api/v1/routes.py \
        applications/api/v1/serializers.py applications/api/v1/viewsets.py
```
8. Within /djangular/backend/mysite/ scaffold your settings dirs (hit enter when line ends):
```
$ mkdir -p mysite/settings/  
$ touch mysite/settings/__init__.py \  
        mysite/settings/base.py mysite/settings/dev.py mysite/settings/prod.py \
        mysite/dev.env mysite/prod.env
```
9. Edit your /mysite/dev.env file to include the following:
```
DATABASE_URL=sqlite:///mysite.db  
DEBUG=on  
FRONTEND_ROOT=path/to/mysite/frontend/  
SECRET_KEY=_some_secret_key 
```
10. In your  djangular/backend/mysite/settings/base.py file, add the following:
```
import environ

project_root = environ.Path(__file__) - 3  
env = environ.Env(DEBUG=(bool, False),)  
CURRENT_ENV = 'dev' # 'dev' is the default environment

# read the .env file associated with the settings that're loaded
env.read_env('./mysite/{}.env'.format(CURRENT_ENV)) 

DATABASES = {  
    'default': env.db()
}

SECRET_KEY = env('SECRET_KEY')  
DEBUG = env('DEBUG')  

# Application definition
INSTALLED_APPS = [

    # Django Packages
    'rest_framework',
]

ROOT_URLCONF = 'mysite.urls'  

STATIC_URL = '/static/'  
STATICFILES_FINDERS = [  
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
]
STATICFILES_DIRS = [  
    env('FRONTEND_ROOT')
]

TEMPLATES = [  
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [env('FRONTEND_ROOT')],
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
```
11. In your /djangular/backend/mysite/settings.dev.py file, add the following:
```
from mysite.settings.base import *


CURRENT_ENV = 'dev'
```
11. Add a directory to your djangular/backend/applications dir called "games"
```
$ mkdir games
```
12. Back out to /djangular/backend and create the games application in th new games dir
```
$ winpty python.exe manage.py startapp games mysite/applications/games
```
* note: if the above command doesn't work, you might need to delete all of the compiled python files (.pyc) from your project and try again.
13. Add the 'applications.game' app to your `INSTALLED_APPS` list in /djangular/backend/mysite/settings/base.py
14. Add a new model class to your models.py file and give it two fields: title and description:
```
class Game(models.model):
  title = models.CharField(max_length=255)
  description = models.CharField(max_length=750)
```
15. Create a DRF serializer for the Game model in /djangular/backend/mysite/applications/api/v1/serializers.py
```
from rest_framework.serializers import ModelSerializer
from applications.games.models import Game


class GameSerializers(ModelSerializer):
  
  class Meta:
    model = Game
```
16. Register your routes using DRF's router registration features in /djangular/backend/mysite/applications/api/v1/routes.py
```
from applications.api.v1.viewsets impot GameViewSet


api_router = routers.SimpleRouter()
api_router.register('games', GameViewSet)
```
17. Map URLs to registered DRF route inside of /djangular/backend/mysite/urls.py
```
from django.contrib import admin
from django.conf.urls import include, url
from applications.api.v1.routes import api_router


urlpatterns = [
  url(r'^admin/', admin.site.urls),
  
  # API: V1
  url(r'^api/v1/', include(api_router.urls)),
]
```



---- right around here everything started screwing up. it turns out that git bash is not the holy grail of command lines on windows - in fact, it will straight up not execute your python manage.py runserver commands. you have to type '''winpty python.exe manage.py runserver''' to get it to work. literally wasted 10 hours on this. still not even sure if this will resolve everything. FML.
