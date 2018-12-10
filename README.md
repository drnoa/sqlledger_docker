SQL-Ledger_docker
================

Docker Build for SQL-Ledger a erp solution for small businesses


# Table of Contents

- [Introduction](#introduction)
- [Contributing](#contributing)
- [Reporting Issues](#reporting-issues)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Creating User and Database at Launch](creating-user-and-database-at-launch)
- [Configuration](#configuration)
    - [Data Store](#data-store)
- [Upgrading](#upgrading)

# Introduction

Dockerfile to build a SQL-Ledger container image which can be linked to other containers.
Will install Postgres and Apache2 and all the necessary packages for SQL-Ledger.

# Contributing

If you find this image useful here's how you can help:

- Send a Pull Request with your awesome new features and bug fixes
- Help new users with [Issues](https://github.com/drnoa/SQLLedger_docker/issues) they may encounter

# Reporting Issues

Docker is a relatively new project and is active being developed and tested by a thriving community of developers and testers and every release of docker features many enhancements and bugfixes.

Given the nature of the development and release cycle it is very important that you have the latest version of docker installed because any issue that you encounter might have already been fixed with a newer docker release.

For ubuntu users I suggest [installing docker](https://docs.docker.com/installation/ubuntulinux/) using docker's own package repository since the version of docker packaged in the ubuntu repositories are a little dated.

Here is the shortform of the installation of an updated version of docker on ubuntu.

```bash
sudo apt-get remove docker docker-engine docker.io
sudo apt-get update
sudo apt-get install docker-ce
```

# Installation

Pull the latest version of the image from the docker index. This is the recommended method of installation as it is easier to update image in the future. These builds are performed by the **Docker Trusted Build** service.

```bash
docker pull drnoa/sqlledger-docker
```

Alternately you can build the image yourself.

```bash
git clone https://github.com/drnoa/SQLLedger_docker.git
cd SQLLedger_docker
docker build -t="<name_of_your_container>" .
```

# Quick Start

Run the SQL-Ledger image

```bash
docker run --name sqlledger_docker -d drnoa/sqlledger-docker:latest
```
Check the ip of your docker container
```bash
docker ps -q | xargs docker inspect | grep IPAddress | cut -d '"' -f 4
```

Got to the administrative interface of SQL-Ledger (e.g. http://172.17.0.3/ledger123/admin.pl) using the password: admin123 and configure the database. All database users (SQL-Ledger and docker) use docker as password.

Alternately you can fetch the password set for the `postgres` user from the container logs.

```bash
docker logs <container-id>
```


To test if the postgresql server is working properly, try connecting to the server.

```bash
psql -U postgres -h $(docker inspect --format {{.NetworkSettings.IPAddress}} sqlledger_docker)
```

# Configuration

## Data Store

For data persistence a volume should be mounted at `/var/lib/postgresql`.

The updated run command looks like this.

```bash
docker run --name <name_of_your_container> -d \
  -v /opt/postgresql/data:/var/lib/postgresql drnoa/sqlledger-docker:latest
```

This will make sure that the data stored in the database is not lost when the image is stopped and started again.

## Securing the server

By default 'docker' is assigned as password for the postgres user. 

You can change the password of the postgres user
```bash
psql -U postgres -h $(docker inspect --format {{.NetworkSettings.IPAddress}} <name_of_your_container>)
\password postgres
```

## Build container from Dockerfile
You can build the container from the Dockerfile in
https://github.com/drnoa/SQLLedger_docker

simply clone the git repo localy and then build
```bash
git clone https://github.com/drnoa/SQLLedger_docker.git
cd SQLLedger_docker
sudo docker build .
```

When you build the container using the Dockerfile you have the possibility to change some parameters
for example the used postgressql version or the database locale (default ist de_DE).
Its also possible to change the postgred passwords.
To change this paramters simply edit the Dockerfile and edit the following values:
```bash
ENV postgresversion 9.1
ENV locale de_CH
ENV postrespassword docker
```


# Upgrading

To upgrade to newer releases, simply follow this 3 step upgrade procedure.

- **Step 1**: Stop the currently running image

```bash
docker stop <name_of_your_container>
```

- **Step 2**: Update the docker image.

```bash
docker pull drnoa/sqlledger_docker:latest
```

- **Step 3**: Start the image

```bash
docker run --name <name_of_your_container> -d [OPTIONS] drnoa/sqlledger_docker:latest
```
