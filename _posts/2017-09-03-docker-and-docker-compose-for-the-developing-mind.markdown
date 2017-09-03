---
title: "Docker and Docker-Compose for the developing mind"
layout: post
date: 2017-09-03 10:44
headerImage: false
tag:
- docker
- docker-compose
star: true
category: blog
author: mattouille
description: Introducing docker and docker-compose to developers for developing powerful local development environments
---

**What's docker?**
I think a lot of folks that read my blog will already know what Docker is, however, I'll give a brief explanation anyway. Docker is a wrapper around Linux Containers (LXC) written in Go. It uses some REST API's to communicate with the docker daemon to be able to start up containers and relies on IPTables for networking along with the docker bridge.

Docker is a powerful tool because the same image I generate locally on my machine can be deployed into production, or the code can be committed and be made part of your _CI/CD Pipeline_. Other local development environment tools such as vagrant use virtual machines which are based on _Xen_, _KVM_, or _VMWare_. Unfortunately, building a vagrant box doesn't really translate to building an image that can be deployed to AWS or Google Cloud. There are products that facilitate this like _Packer_ (From HashiCorp as well), however, it's a bit of a stretch to get developers to start using this, much less to start using it to bring up local environments.

**Terminology**

| Term | Definition |
| --- | --- |
| Host OS | Relatively, this is your machine or the server that has the container running on it. |
| Guest OS | This usually is reserved for virtual machines, but it's fair to use the term when describing the operating system the container image is based on.|
| Ephemeral | Temporary or short lived. |
| Immutable | Indicates a lack of data persistence. No data is stored on something that is immutable. The underlying term actually means '_replaceable_'. |

**Dockerfile**
The Dockerfile basically describes how to build your image. Dockerfiles are relatively simple. In fact, [here's the entire reference for Dockerfile](https://docs.docker.com/engine/reference/builder/), take a quick glance over it. You can do simple things like exposing ports to the host OS (making things externally accessible), you can attach volumes that can add persistence to Docker (Docker and Linux Containers are ephemeral and immutable by nature), you can add existing directories, etc... The Dockerfile is the quintessential for building custom images that run on Docker containers.

**docker-compose**
Docker has an accompanying application called **docker-compose**. If you're using OSX then the install for Docker already comes with it, if you're using Linux or Windows you'll need to download it separately.

Rarely do we ever just run one application by itself. Frequently we need to run an application, a data source, and any other piece of infrastructure it may be dependent on. In this example I'm going to use a real Dockerfile and docker-compose that I wrote for a FOSS project I've been working on. The docker-compose is yaml based and represents various container and images pairs as _services_.

**Building our Dockerfile**
The framework I'm using is called Django which is python based. All I need is an OS with Python 3. All publicly available Docker images are based on some type of OS with all the non-essentials removed.

**Dockerfile**

{% highlight shell %}
FROM python:3.6

ENV PYTHONUNBUFFERED 1

RUN mkdir /code

WORKDIR /code

ADD requirements.txt /code/

RUN pip install -r requirements.txt

ADD . /code/
{% endhighlight %}

* **FROM** we're using the _python_ image from [Docker Hub](https://hub.docker.com) with a [label of 3.6](https://hub.docker.com/_/python/).

* **ENV** sets an environment variable in <Key> <Value> format. If you had multiple you'd use multiple **ENV**'s.

* **RUN** This indicates executes a command using the default shell. We're making the preliminary directory called _code_ in the root of the guest OS (note the prefixed slash).

* **WORKDIR** Changes the working directory to `/code`. The commands using this context are specific to `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD`.

*  **ADD** Copies a file to your Docker image. You might've noticed there's a command called `COPY` that is remarkably similar to `ADD`. There's two key differences.

 * Add allows the <src> to be a url.
 * Add will decompress recognized compressed files.

Choose wisely!

* **RUN** I need pip to process my requirements.txt. We're executing a normal shell command here, just as if we were sitting at the terminal.

* **ADD** We're now copying my existing code repository to Docker image.

That's it! If all we wanted to do was build the image and run the container then Django would come up. You'd simply:

`docker build .`

_This indicates to docker: Build my image, search for the Dockerfile in this directory (.)_

`docker create <image id>`

At this point your docker container would run but would not be externally accessible (we never specified `EXPOSE`).

**docker-compose.yml**

{% highlight yaml %}
version: '3.3'

services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: theseus
  web:
    build: .
    command: python code/manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    depends_on:
      - db
{% endhighlight %}

* **version** specifies which version of the [docker-compose reference](https://docs.docker.com/compose/compose-file/) to use.

* **services** Each top level is essentially a different container. Services are special because they can resolve each other by their service name. For instance, I can reach _db_ from _web_ by `ping db`.

* **db/image** In this case I'm telling docker-compose to use the Docker Hub image for postgres. Notice there's no label selected, Docker will assume _latest_.

* **db/environment** I'm setting an environment variable in a Key: Value format. The postgres page on Docker Hub has some nice instructions on features they've put into the postgres image.

* **web/build** I'm indicating to docker-compose to run a docker build in '.'. This will build my Docker image. If I had multiple custom images I'd just place a Dockerfile in different directories.

* **web/command** this is the command to run when the container starts. Notice I lost my context behind `WORKDIR` in the Dockerfile. There's a way to fix this, but I wanted to point it out.

* **web/volumes** I'm telling docker-compose that I want to mount /code as a volume. Changes I make in my code will be reflected into the container **AND** vice versa. I'll get into this later.

* **web/ports** This is the exposed port on the Host OS to the Guest OS.

* **web/depends_on** This tells docker-compose that it can't start the web container until the db container is online.

**Using docker-compose**. Before we were able to use the `docker` command to build, create, run etc... Now that we're using docker-compose though, we have to use the `docker-compose` command. If you don't, you can still reach your containers but they'll be without some of the essential networking that docker-compose gives you.

You'll need to create your code folder and generate a Django app, but after that simply run `docker-compose up -d`. This launches docker-compose in daemon mode. Leave out `-d` when you're troubleshooting. You can run `docker-compose ps` to get the names of your containers and run commands on them like this: `docker-compose run -it <container name> <command>`.

Technically we're all done, but I wanted to show you something neat. Remember how I said in docker-compose you can reference services by name and they'll resolve? In case it didn't click, here's an excerpt from my Django settings.py:

{% highlight python %}
# Database
# https://docs.djangoproject.com/en/1.11/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'postgres',
        'USER': 'postgres',
        'PASSWORD': 'theseus',
        'HOST': 'db',
        'PORT': 5432,
    }
}

{% endhighlight %}

My host is using '_db_'. That's all for today, hopefully that's helped get the gears spinning. There's a lot more you can do with docker-compose and docker, so be sure to explore!
