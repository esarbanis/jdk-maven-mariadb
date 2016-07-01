# Docker image jdk, maven, mariadb

## TL;DR
This docker image is intended to be used in Bitbucket Pipelines where we can do full integration tests on application (from backend to database).

This image contains:
* JDK 8u91
* Maven 3.3.3
* MariaDB 10.1

Bitbucket Pipelines doesn't support multiple (linking) containers on commit, so we need single container that can give us proper technology basics where we can do all our tests.

Currently we are mostly working with Java (JDK8) environment, using Maven to do builds (tests), and MariaDB for database.

Look below for sample `bitbucket-pipelines.yml` file on how to use this image.

## Why this image ?
On the first look it might seem crazy to combine "full stack" in signle Docker image. Main idea behind Docker containers is to split responsabilities so application would have its own container and database would have separate containser. So, why would anybody have container to mix app and database?

Bitbucket is introducing new feature [Pipelines](https://bitbucket.org/product/features/pipelines), still in beta phase, that enables you to run docker containers on bitbucket with every commit and test your code in container. 

Problem with current stage of Pipelines feature is that it doesn't support linking (multiple) containers. So for all our testing - we can only use one container - hence this image is created to contain all technologies that we use (JDK8, Maven, MariaDB).

## How is this image created ?
This image is blatant rip-off and merge of several official Docker images

Big thanks to authors of these images for sharing their work!

* [MariaDB Docker](https://hub.docker.com/_/mariadb/) - Official MariaDB Docker page
* [MariaDB GitHub](https://github.com/docker-library/mariadb) - Official Docker MariaDB GitHub repository
* [Maven Docker](https://hub.docker.com/_/maven/) - Official Maven Docker page
* [Maven GitHub](https://github.com/carlossg/docker-maven) - Official Docker Maven GitHub repository
* [Java/JDK Docker](https://hub.docker.com/_/java/) - Official Java Docker page
* [Java/JDK GitHub](https://github.com/docker-library/openjdk) - Official Docker Java GitHub repository

## Parameters

#### Database port
MariaDB is exposing port `3306` so when running container if you want to use database you should make proper port mapping.

#### Database parameters
When running container you can use several environment variables (expected by  `docker-entrypoint.sh` script):
* `MYSQL_ROOT_PASSWORD` - this will set root password of database
* `MYSQL_DATABASE` - this will create database schema with provided name
* `MYSQL_USER`, `MYSQL_PASSWORD` - with these variables you can setup user that will have grants on database created using `MYSQL_DATABASE`. If you already provided root password, then you don't need to specify these variables
* `MYSQL_ALLOW_EMPTY_PASSWORD` - set this variable to `yes` if you don't want to be bothered with root password (not recommended!)

## How to use

### Use public container in your Bitbucket Pipeline

In order to even start using Bitbucket Pipelines you must have `bitbucket-pipelines.yml` file in root of your repository - [official instructions on how to configure Bitbucket Pipelines](https://confluence.atlassian.com/bitbucket/bitbucket-pipelines-beta-792496469.html).

Sample yml file:

```
image: 
  name: lukastosic/maven-mariadb:1.2
  MYSQL_ROOT_PASSWORD: provide_root_password
  MYSQL_DATABASE: provide_database_name

pipelines:
  default:
    - step:
        script:
          - mvn --version
          - mvn clean test
```

In this sample we have used 2 database parameters described above. You can use other parameters, or you can use environment variables from Bitbucket - [more info about environment variables](https://confluence.atlassian.com/bitbucket/environment-variables-in-bitbucket-pipelines-794502608.html)


### Use docker container on your own server

For complete instructions on how to use Docker container you can refer to [MariaDB Docker page](https://hub.docker.com/_/mariadb/).

#### Sample run commands

Running with specified root password

```
docker run --name my_container_name -e MYSQL_ROOT_PASSWORD=my_password -d lukastosic/maven-mariadb:1.2
```

Running with specified database and username/password

```
docker run --name my_cintainer_name -e MYSQL_DATABASE=init_database -e MYSQL_USER=my_user -e MYSQL_PASSWORD=my_password -d lukastosic/maven-mariadb:1.2
```

## How to build docker image from source

I'm not sure why would you want to do this, since image is available as public repo so you can just `pull` the image, but if you really want to play:

* download this repository to some folder
* navigate to folder where you downloaded this repository
* execute `docker build -t image_name:tag .`