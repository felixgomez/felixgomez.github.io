---
title: Entorno de desarrollo con Docker para Symfony4
categories:
- Devops
- PHP
tags:
    - Docker
    - docker-compose
thumbnail: /images/docker-logo.png
---

Ha quedado atrás el tiempo en que para poder desarrollar en PHP había que instalar un entorno en función del sistema operativo, necesidades de versiones de los diferentes elementos de infraestructura y además hacerlo para cada proyecto.

La necesidad de sincronizar la información de infraestructura como se puede hacer con el código da pie al concepto de **"Infrastructure as code"**.

{% blockquote Kief Morris @kief, Author of Infrastructure as Code (O'Reilly)%}
The enabling idea of infrastructure as code is that the systems and devices wich are used to run software can be treated as if they, themselves, are software.
{% endblockquote %}

Veremos como como configurar el entorno local para programar con Symfony4 con **Docker**.

El código se encuentra en https://github.com/felixgomez/docker-symfony4.
<!-- more -->

## Instalación

Se puede descargar un instalador desde [Docker CE (_Community Edition_)](https://www.docker.com/community-edition). En mi caso una vez instalado y arrancado tengo:

![](/images/docker-about.png).

Y desde consola tenemos acceso tanto a `docker` como a `docker-compose` y `docker-machine`. 

## Imágenes y configuración

Podemos configurar un contenedor de docker a partir de una imagen base y hacer modificaciones en base a un _Dockerfile_, que es un archivo de texto plano con una sintaxis específica. 

Estas imágenes modificadas pueden ser base a su vez para la generación de nuevos contenedores.

### Imágenes

Podemos utilizar [Docker Hub](https://hub.docker.com/) para localizar imágenes. En este ejemplo tiraremos de imágenes **oficiales**.

Para este ejemplo configuraremos 4 contenedores orquestados con `docker-compose`:

- **webserver**, basado en imagen oficial (https://hub.docker.com/_/nginx/)
- **mysql**, basado en imagen oficial (https://hub.docker.com/_/mysql/)
- **redis**, basado en imagen oficial (https://hub.docker.com/_/redis/)
- **php-fpm**, basado en imagen oficial (https://hub.docker.com/_/php/)

Y como aplicación PHP utilizaremos el esqueleto de instalación de [Symfony 4](https://symfony.com/download).
 
 
### Configuración

Para este ejemplo sencillo utilizaremos la siguiente estructura:
```bash
.
├── application # Proyecto PHP con Symfony 4
│   ├── bin
│   ├── composer.json
│   ├── composer.lock
│   ├── config
│   ├── public
│   ├── src
│   ├── symfony.lock
│   ├── var
│   └── vendor
└── infrastructure # Contenedores y configuración
    ├── .env
    ├── nginx
    ├── php-fpm
    └── docker-compose.yml
```

** El código para la parte de infraestructura se encuentra en [Github](https://github.com/felixgomez/docker-symfony4) **

Para cada uno de los servicios únicamente se utilizan las siguientes opciones de configuración:

- **image / build**: Para utilizar una imagen o para construir una en base a un `Dockerfile`
- **container_name**: Nombre del contenedor
- **working_dir**: Directorio de trabajo
- **environment**: Establecimiento de variables de entorno
- **volumes**: Establecimiento de puntos de montaje
- **port**: Mapeo de puertos host <-> container

Con estas opciones el archivo `docker-compose.yml` podría ser

```yml
version: "3.5"
services:

    redis:
      image: redis:alpine
      container_name: php-redis

    mysql:
      image: mysql:8.0
      container_name: php-mysql
      volumes:
        - "./database:/var/lib/mysql"
      environment:
        - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
        - MYSQL_DATABASE=${MYSQL_DATABASE}
        - MYSQL_USER=${MYSQL_USER}
        - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      ports:
        - "8082:3306"

    webserver:
      image: nginx:alpine
      container_name: php-webserver
      volumes:
          - ../infrastructure/nginx/nginx.conf:/etc/nginx/conf.d/default.conf
      ports:
       - "8080:80"

    php-fpm:
      build: php-fpm
      container_name: php-fpm
      volumes:
        - ${SYMFONY_APP_PATH}:/application
        - ../infrastructure/php-fpm/php-ini-overrides.ini:/etc/php/7.2/fpm/conf.d/99-overrides.ini
        - ./var:/application/var

    phpmyadmin:
        image: phpmyadmin/phpmyadmin
        container_name: phpmyadmin
        ports:
            - "8081:80"
```

## Ejecución

Para descargar las imágenes y construir los contenedores ejecutamos
```bash
$ docker-compose build
```

Y para levantar los servicios:

```bash
$ docker-compose up -d # Detached mode
Recreating php-mysql ...
phpmyadmin is up-to-date
php-webserver is up-to-date
php-redis is up-to-date
Recreating php-fpm ... done
```

## Referencias

- https://github.com/felixgomez/docker-symfony4
- https://www.docker.com/
- https://hub.docker.com/
- https://docs.docker.com/compose/
