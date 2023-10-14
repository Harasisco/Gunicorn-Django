# WebServer_using_Postgresql

<p>This repository contains a Dockerized setup for running a Django web application using Gunicorn with a Postgresql database and an Nginx reverse proxy. This setup allows you to easily manage and deploy your Django application in a containerized environment.</p>

## Table of Contents

- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Check The Connectivity](#check-the-connectivity)
- [Compose File](#compose-file)
- [Configuration](#configuration)

## Prerequisites

Make sure you have the following prerequisites installed on your system:

- Docker: [Install Docker](https://docs.docker.com/get-docker/)
- Docker Compose: [Install Docker Compose](https://docs.docker.com/compose/install/)

## Getting Started

1. Clone this repository to your local machine:

```shell
   git clone https://github.com/Harasisco/WebServer_using_Postgresql.git
   cd Dockarizing-Django-nginx-PostgreSQL
```

2. Create your own docker network:

```shell
  docker network create --subnet=10.0.0.0/24 backend-network
```

3. Build the Django Docker image:

```shell
docker build -t django_image ./django_gunicorn
```

4. Start the PostgreSQL container:

```shell
docker run -d --name db --hostname=PostgreSQL --ip 10.0.0.15 -v ./postgresql/:/var/lib/postgresql/data/ --net backend-network -e POSTGRES_PASSWORD=harasisco -e POSTGRES_DB=my_database -e PGDATA=/tmp postgres
```

5. Start the Django container:

```shell
docker run -d --name web --hostname=Django_server --ip 10.0.0.10 -v ./django_gunicorn:/code/ --net backend-network --env-file ./django_gunicorn/.env --expose 8000 --link db django_image sh -c "cd /code/nginx_guni && gunicorn ngnix_guni.wsgi:application --bind 0.0.0.0:8000"
```

6. Start the Nginx container:

```shell
docker run -d --name nginx --hostname=nginx --ip 10.0.0.5  -p 80:80 -v ./nginx/conf.d/:/etc/nginx/conf.d/ --net backend-network --link web nginx:latest
```

7. Access your Django application in your web browser by navigating to http://localhost or using the IP 127.0.0.1.


## Check The Connectivity

![image](https://github.com/Harasisco/WebServer_using_Postgresql/assets/87074807/faf28c32-72e8-46dc-b99d-95e964df7ceb)

<p> You can check the network connectivity between nginx and Django by executing the django container and run the ``` ping ``` command </p>

```shell
$ docker exec -it <container ID> /bin/bash
```
```shell
root@Django_server:/code# ping nginx
PING nginx_net (10.0.0.5) 56(84) bytes of data.
64 bytes from nginx.backend-network (10.0.0.5): icmp_seq=1 ttl=64 time=0.174 ms
64 bytes from nginx.backend-network (10.0.0.5): icmp_seq=2 ttl=64 time=0.066 ms
64 bytes from nginx.backend-network (10.0.0.5): icmp_seq=3 ttl=64 time=0.261 ms
64 bytes from nginx.backend-network (10.0.0.5): icmp_seq=4 ttl=64 time=0.063 ms
64 bytes from nginx.backend-network (10.0.0.5): icmp_seq=5 ttl=64 time=0.066 ms
64 bytes from nginx.backend-network (10.0.0.5): icmp_seq=6 ttl=64 time=0.066 ms
64 bytes from nginx.backend-network (10.0.0.5): icmp_seq=7 ttl=64 time=0.067 ms
^C
--- nginx_net ping statistics ---
7 packets transmitted, 7 received, 0% packet loss, time 6119ms
rtt min/avg/max/mdev = 0.063/0.109/0.261/0.072 ms

```
### To check the PostgreSQL connectivity:

  - Firstly execute the PostgreSQL container using:
    ```shell
    docker exec -it <container ID>  /bin/bash
    ```
  - Secondery check the PostgreSQL data base info:
    ```shell
   docker exec -it postgresql psql -U root -d my_database
   
   \dt;
    ```
   - You will see that No relations shown.
     
   - In a new tab execute :
     ```shell
     docker container  exec django_server python /code/nginx_guni/manage.py migrate --noinput
     ```
   - Finally, Go back to the PostgreSQL tab and rerun the ``` \dt; ``` command to figoure out that the Django container and PostgreSQL container are linked correctly.

## Compose File

<p> I've included a Docker Compose file (docker-compose.yml) in this project to simplify the setup process and manage the different containers required for our Django application, PostgreSQL database, and Nginx reverse proxy. Docker Compose allows you to define and run multi-container Docker applications with ease.</p>

### To use it Follow these commands:

```shell
docker compose build
docker compose up
```
And when You finsh press ``` Ctrl + C ``` and
```shell
docker compose down
```

## Configuration
- Django settings can be configured in the ./django_gunicorn/mysite/settings.py file.
- PostgreSQL configuration is set in the ./django_gunicorn/.env file under the db service section.
- Nginx configuration files can be added/modified in the ./nginx/conf.d/ directory.
