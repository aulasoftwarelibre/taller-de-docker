# Levantar un WordPress con Docker Compose

El cliente de _Docker_ es engorroso para crear contenedores, así como para crear el resto de objetos y vincularlos entre sí.

Para automatizar la creación, inicio y parada de un contenedor o un conjunto de ellos, _Docker_ proporciona una herramiento llamada _Docker Compose_.

Para esta parte vamos a detener y borrar lo que hemos creado:

!!! example

    Borra el trabajo actual:

        docker container stop wordpress wordpress-db
        docker container rm wordpress wordpress-db
        docker volume rm wordpress-db

## Creación de contenedores automatizada

En el mismo directorio donde estábamos en el paso anterior (`~/Sites/wordpress`), vamos a crear un fichero llamado `docker-compose.yaml` con el siguiente contenido:

    :::yaml
    version: '3'

    services:
        db:
            image: mariadb:10.3.9
            volumes:
                - data:/var/lib/mysql
            environment:
                - MYSQL_ROOT_PASSWORD=secret
                - MYSQL_DATABASE=wordpress
                - MYSQL_USER=manager
                - MYSQL_PASSWORD=secret
        web:
            image: wordpress:4.9.8
            depends_on:
                - db
            volumes:
                - ./target:/var/www/html
            environment:
                - WORDPRESS_DB_USER=manager
                - WORDPRESS_DB_PASSWORD=secret
                - WORDPRESS_DB_HOST=db
            ports:
                - 8080:80

    volumes:
        data:
            driver: local

!!! info
    YAML es un lenguaje de serialización de datos diseñado para ser leído y escrito por personas. Se recomienda que sigas algún tutorial para entender su formato: [Aprende YAML en Y minutos](https://learnxinyminutes.com/docs/es-es/yaml-es/).

El 
