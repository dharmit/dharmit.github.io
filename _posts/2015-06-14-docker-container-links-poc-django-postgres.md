---
layout: post
title: Linking Docker containers- A POC using django and postgres
categories:
- blog
---


As I mentioned in my [earlier post](dharmit.github.io/blog/2015/06/13/moving-from-wordpress-to-jekyll.html){:target="_blank"}, I was working on writing a blog about linking docker containers running `django` and `postgresql`.

Docker container linking is not a new thing. It's been used widely by a lot of people out there. However, when I wanted to link a django and postgres container, I could only find posts that explained the linking using some existing django code.

What I was actually looking for was the modifications to be made to `settings.py`, way to correctly run a postgres container, and then link them correctly. Since I couldn't find such a post, I wrote one myself in the hope of it being useful for someone who is same situation as I was.

This post doesn't have any boilerplate code and walks you through steps right from creating a django project to creating the containers and then linking them in the end.

Software versions:

* Ubuntu 15.04 (64-bit)
* Docker 1.6.2
* Django 1.8.2
* Django Rest Framework 3.1.3
* psycopg2 2.6

Docker images:

* Ubuntu (latest)
* Postgres official image

First we will create a django project and see if things work well on the local system. Follow below steps to create a Django project and an app:

{% highlight sh %}

$ mkcd link_containers
$ virtualenv venv
$ source venv/bin/activate
$ pip install django djangorestframework psycopg2
$ django-admin startproject docker_links
$ cd docker_links
$ python manage.py startapp api

{% endhighlight %}

Modify `docker_links/settings.py` to have below information:

{% highlight python %}

INSTALLED_APPS = (
    ..
    'rest_framework',
    'api',
)

{% endhighlight %}

and

{% highlight python %}

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'docker_links',
        'USER': 'postgres',
        'PASSWORD': 'postgres123',
        'HOST': '127.0.0.1',
        'PORT': '5432'
    }
}

{% endhighlight %}

Make sure that `DATABASES` dictionary has settings that reflect configuration on your system!

To ensure that requests to api are redirected to api app, modify `docker_links/urls.py`:

{% highlight python %}

from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^api/', include('api.urls')),
]

{% endhighlight %}

Let `api/models.py` have very basic code like below:

{% highlight python %}

from django.db import models

class User(models.Model):

    name = models.CharField(max_length=255)
    class Meta:
        db_table = "users"
{% endhighlight %}

And apply migrations:

{% highlight sh %}

$ python manage.py makemigrations
$ python manage.py migrate

{% endhighlight %}

Next, have below code in `api/serializers.py`:

{% highlight python %}

from rest_framework.serializers import ModelSerializer
from api.models import User

class UserSerializer(ModelSerializer):

    class Meta:
        fields = ('id', 'name',)
        model = User

{% endhighlight %}

`api/urls.py`:

{% highlight python %}
from django.conf.urls import url, patterns

urlpatterns = patterns(
    'api.views',

    url(r'^users/(?P<pk>[0-9]+)',
        'user_detail', name='user_detail'),

    url(r'^users/new/$', 'user_create', name='user_create'),
)

{% endhighlight %}

and `api/views.py`:

{% highlight python %}

from api.models import User
from api.serializers import UserSerializer

from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

@api_view(['GET'])
def user_detail(request, pk):
    try:
        user = User.objects.get(id=pk)
    except User.DoesNotExist:
        return Response({"error": "User with id %s does not exist" % pk})
    else:
        serializer = UserSerializer(user)
        return Response(serializer.data, status=status.HTTP_200_OK)

@api_view(['POST'])
def user_create(request):
    try:
        serializer = UserSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
    except:
        return Response({"error": "Invalid data"},
                        status=status.HTTP_400_BAD_REQUEST)
    else:
        return Response(serializer.data, status=status.HTTP_201_CREATED)

{% endhighlight %}

I think that is it. You can now test `POST` and `GET` requests by running the django dev server:

{% highlight sh %}
$ python manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).
June 13, 2015 - 15:09:46
Django version 1.8.2, using settings 'docker_links.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.

{% endhighlight %}

I prefer using [Postman REST client](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm?hl=en "Postman REST Client"){:target="_blank"} to test REST APIs.

If everything works well, make sure to have a `requirements.txt` file created using below command:

{% highlight sh %}

$ pip freeze > requirements.txt

{% endhighlight %}

`requirements.txt` could be in the same directory as `manage.py`.

Now that we have everything working on the host which has django and postgresql configured on it, we will move towards dockerizing the setup and link the containers running django and postgresql.

Create a Dockerfile like below:

~~~
FROM ubuntu:latest

RUN apt-get update && apt-get upgrade
RUN apt-get -y install python-pip
RUN apt-get install python-dev postgresql-server-dev-all -y

RUN mkdir /code && cd /code
WORKDIR /code
ADD docker_links /code

RUN pip install -r requirements.txt
~~~

and build an image out of it:

{% highlight sh %}
$ sudo docker build -t django .
{% endhighlight %}

Going through Dockerfile line by line, `python-pip` package is required to install the project dependencies from earlier created `requirements.txt` file. `python-dev` and `postgresql-server-dev-all` packages are required to make sure that `psycopg2` has its dependencies preinstalled. Then we copy the code from host system to the container and install dependencies from `requirements.txt`.

Let's start the postgresql container:

{% highlight sh %}

$  sudo docker run --name db -p 5432:5432 -e POSTGRES_PASSWORD=postgres123 -d postgres

{% endhighlight %}

Create a database called `docker_links` in the postgres container:

{% highlight sh %}

$ psql -h <container-ip-address> -U postgres

{% endhighlight %}

Above command should work without specifying host with `-h` switch. I need to figure it out.

Create a throwaway container to apply migrations from our django app:

{% highlight sh %}

$ sudo docker run --rm --link db:db django python manage.py migrate

{% endhighlight %}

And finally create a container that would serve our REST APIs:

{% highlight sh %}

$ sudo docker run -d --name web -p 80:8000 --link db:db django python manage.py runserver 0.0.0.0:8000

{% endhighlight %}

You can now test API on localhost using the Postman REST client.

*For any suggestions on improving above workflow, please ping me on [twitter](http://twitter.com/dharm1t "Twitter"){:target="_blank"}.*












