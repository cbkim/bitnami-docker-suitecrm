[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-suitecrm/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-suitecrm/tree/master)
[![Slack](http://slack.oss.bitnami.com/badge.svg)](http://slack.oss.bitnami.com)
[![Kubectl](https://img.shields.io/badge/kubectl-Available-green.svg)](https://raw.githubusercontent.com/bitnami/bitnami-docker-suitecrm/master/kubernetes.yml)

# What is SuiteCRM?

> SuiteCRM is a completely open source enterprise-grade Customer Relationship Management (CRM) application. SuiteCRM is a software fork of the popular customer relationship management (CRM) system SugarCRM.

https://www.suitecrm.com/

# TL;DR;

## Docker Compose

```bash
$ curl -LO https://raw.githubusercontent.com/bitnami/bitnami-docker-suitecrm/master/docker-compose.yml
$ docker-compose up
```

## Kubernetes

> **WARNING**: This is a beta configuration, currently unsupported.

Get the raw URL pointing to the kubernetes.yml manifest and use kubectl to create the resources on your Kubernetes cluster like so:

```bash
$ kubectl create -f https://raw.githubusercontent.com/bitnami/bitnami-docker-suitecrm/master/kubernetes.yml
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* Bitnami images are built on CircleCI and automatically pushed to the Docker Hub.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recommended with a version 1.6.0 or later.

# How to use this image

## Run SuiteCRM with a Database Container

Running SuiteCRM with a database server is the recommended way. You can either use docker-compose or run the container manually.

### Run the application using Docker Compose

This is the recommended way to run SuiteCRM. You can use the following docker compose template:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    volumes:
      - 'mariadb_data:/bitnami/mariadb'
  suitecrm:
    image: 'bitnami/suitecrm:latest'
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - 'suitecrm_data:/bitnami/suitecrm'
      - 'php_data:/bitnami/php'
      - 'apache_data:/bitnami/apache'
    depends_on:
      - mariadb

volumes:
  mariadb_data:
    driver: local
  suitecrm_data:
    driver: local
  apache_data:
    driver: local
  php_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create suitecrm-tier
  ```

2. Start a MariaDB database in the network generated:

  ```bash
   $ docker run -d --name mariadb -e ALLOW_EMPTY_PASSWORD=yes --net=suitecrm-tier bitnami/mariadb
  ```

  *Note:* You need to give the container a name in order to SuiteCRM to resolve the host

3. Run the SuiteCRM container:

  ```bash
  $ docker run -d -p 80:80 --name suitecrm --net=suitecrm-tier bitnami/suitecrm
  ```

Then you can access your application at http://your-ip/

## Persisting your application

If you remove every container and volume all your data will be lost, and the next time you run the image the application will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence of the SuiteCRM deployment, the above examples define docker volumes namely `mariadb_data`, `suitecrm_data`, `apache_data` and `php_data`. The SuiteCRM application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

> **Note!** If you have already started using your application, follow the steps on [backing](#backing-up-your-application) up to pull the data from your running container down to your host.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the `docker-compose.yml` template previously shown:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    volumes:
      - '/path/to/mariadb-persistence:/bitnami/mariadb'
  suitecrm:
    image: 'bitnami/suitecrm:latest'
    depends_on:
      - mariadb
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - '/path/to/suitecrm-persistence:/bitnami/suitecrm'
      - '/path/to/php-persistence:/bitnami/php'
      - '/path/to/apache-persistence:/bitnami/apache'
```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create suitecrm-tier
  ```

2. Create a MariaDB container with host volume:

  ```bash
  $ docker run -d --name mariadb -e ALLOW_EMPTY_PASSWORD=yes \
    --net suitecrm-tier \
    --volume /path/to/mariadb-persistence:/bitnami/mariadb \
    bitnami/mariadb:latest
  ```
   *Note:* You need to give the container a name in order to SuiteCRM to resolve the host

3. Create the SuiteCRM container with host volumes:

  ```bash
  $ docker run -d --name suitecrm -p 80:80 -p 443:443 \
    --net suitecrm-tier \
    --volume /path/to/suitecrm-persistence:/bitnami/suitecrm \
    --volume /path/to/apache-persistence:/bitnami/apache \
    --volume /path/to/php-persistence:/bitnami/php \
    bitnami/suitecrm:latest
  ```

# Upgrade this application

Bitnami provides up-to-date versions of MariaDB and SuiteCRM, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the SuiteCRM container. For the MariaDB upgrade you can take a look at https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```bash
  $ docker pull bitnami/suitecrm:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop suitecrm`
 * For manual execution: `$ docker stop suitecrm`

3. (For non-compose execution only) Create a [backup](#backing-up-your-application) if you have not mounted the suitecrm folder in the host.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm -v suitecrm`
 * For manual execution: `$ docker rm -v suitecrm`

5. Run the new image

 * For docker-compose: `$ docker-compose start suitecrm`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name suitecrm bitnami/suitecrm:latest`

# Configuration
## Environment variables
 When you start the SuiteCRM image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:

```yaml
suitecrm:
  image: bitnami/suitecrm:latest
  ports:
    - 80:80
  environment:
    - SUITECRM_PASSWORD=my_password
  volumes_from:
    - suitecrm_data
```

 * For manual execution add a `-e` option with each variable and value:

  ```bash
  $ docker run -d -p 80:80 -p 443:443 --name suitecrm  \
    -e SUITECRM_PASSWORD=my_password \
    --net suitecrm-tier \
    --volume /path/to/suitecrm-persistence:/bitnami/suitecrm \
    --volume /path/to/apache-persistence:/bitnami/apache \
    --volume /path/to/php-persistence:/bitnami/php \
    bitnami/suitecrm:latest
  ```

Available variables:

 - `SUITECRM_USERNAME`: SuiteCRM application username. Default: **User**
 - `SUITECRM_PASSWORD`: SuiteCRM application password. Default: **bitnami**
 - `SUITECRM_EMAIL`: SuiteCRM application email. Default: **user@example.com**
 - `SUITECRM_LASTNAME`: SuiteCRM application last name. Default: **Name**
 - `SUITECRM_HOST`: Host domain or IP.
 - `MARIADB_USER`: Root user for the MariaDB database. Default: **root**
 - `MARIADB_PASSWORD`: Root password for the MariaDB.
 - `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
 - `MARIADB_PORT_NUMBER`: Port used by MariaDB server. Default: **3306**

### SMTP Configuration

To configure SugarCMR to send email using SMTP you can set the following environment variables:

 - `SUITECRM_SMTP_HOST`: SugarCRM SMTP host.
 - `SUITECRM_SMTP_PORT`: SugarCRM SMTP port.
 - `SUITECRM_SMTP_USER`: SugarCRM SMTP account user.
 - `SUITECRM_SMTP_PASSWORD`: SugarCRM SMTP account password.
 - `SUITECRM_SMTP_PROTOCOL`: SugarCRM SMTP protocol to use.

This would be an example of SMTP configuration using a Gmail account:

 * docker-compose:

```yaml
  suitecrm:
    image: bitnami/suitecrm:latest
    ports:
      - 80:80
    environment:
      - SUITECRM_SMTP_HOST=smtp.gmail.com
      - SUITECRM_SMTP_USER=your_email@gmail.com
      - SUITECRM_SMTP_PASSWORD=your_password
      - SUITECRM_SMTP_PROTOCOL=TLS
      - SUITECRM_SMTP_PORT=587
```

 * For manual execution:

  ```bash
  $ docker run -d -p 80:80 -p 443:443 --name suitecrm  \
    -e SUITECRM_SMTP_HOST=smtp.gmail.com \
    -e SUITECRM_SMTP_PROTOCOL=TLS \
    -e SUITECRM_SMTP_PORT=587 \
    -e SUITECRM_SMTP_USER=your_email@gmail.com \
    -e SUITECRM_SMTP_PASSWORD=your_password
    --net suitecrm-tier \
    --volume /path/to/suitecrm-persistence:/bitnami/suitecrm \
    --volume /path/to/apache-persistence:/bitnami/apache \
    --volume /path/to/php-persistence:/bitnami/php \
    bitnami/suitecrm:latest
  ```
# Backing up your application

To backup your application data follow these steps:

1. Stop the running container:

  * For docker-compose: `$ docker-compose stop suitecrm`
  * For manual execution: `$ docker stop suitecrm`

2. Copy the SuiteCRM data folder in the host:

  ```bash
  $ docker cp /path/to/suitecrm-persistence:/bitnami/suitecrm
  ```

# Restoring a backup

To restore your application using backed up data simply mount the folder with SuiteCRM data in the container. See [persisting your application](#persisting-your-application) section for more info.

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an
[issue](https://github.com/bitnami/bitnami-docker-suitecrm/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-suitecrm/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an
[issue](https://github.com/bitnami/bitnami-docker-suitecrm/issues). For us to provide better support,
be sure to include the following information in your issue:

- Host OS and version
- Docker version (`$ docker version`)
- Output of `$ docker info`
- Version of this container (`$ echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive
information)

# Community

Most real time communication happens in the `#containers` channel at [bitnami-oss.slack.com](http://bitnami-oss.slack.com); you can sign up at [slack.oss.bitnami.com](http://slack.oss.bitnami.com).

Discussions are archived at [bitnami-oss.slackarchive.io](https://bitnami-oss.slackarchive.io).

# License

Copyright 2016 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

 <http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
