# Levantar un WordPress con Docker

Para crear un _blog_ con _WordPress_ necesitamos tener una base de datos dónde almacenar las entradas. Así que empezaremos creándola y después crearemos el contenedor de nuestro _blog_.

## Crear un contenedor con _MariaDB_.

_WordPress_ soporta los motores relaciones _MySQL_ y _MariaDB_. Usaremos este último.

!!! example
    Vamos a crear nuestra base de datos usando este volumen.
    
        docker run -d --name wordpress-db \
            --mount source=wordpress-db,target=/var/lib/mysql \
            -e MYSQL_ROOT_PASSWORD=secret \
            -e MYSQL_DATABASE=wordpress \
            -e MYSQL_USER=manager \
            -e MYSQL_PASSWORD=secret mariadb:10

La imagen se descargará, si no lo estaba ya, y se iniciará nuestro contenedor de _MariaDB_:

```console hl_lines="1-6"
$ docker run -d --name wordpress-db \
    --mount source=wordpress-db,target=/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=wordpress \
    -e MYSQL_USER=manager \
    -e MYSQL_PASSWORD=secret mariadb:10
Unable to find image 'mariadb:10' locally
10: Pulling from library/mariadb
124c757242f8: Pull complete 
9d866f8bde2a: Pull complete 
fa3f2f277e67: Pull complete 
398d32b153e8: Pull complete 
afde35469481: Pull complete 
31f2ae82b3e3: Pull complete 
3eeaf7e45ea6: Pull complete 
716982328e17: Pull complete 
34ce605c9036: Pull complete 
4502ed9073c0: Pull complete 
2afafbdf5a96: Pull complete 
43d52b11dd31: Pull complete 
30c7b70556f3: Pull complete 
8b1b39f2f89a: Pull complete 
41480b9319d7: Pull complete 
Digest: sha256:b7894bd08e5752acdd41fea654cb89467c99e67b8293975bb5d787b27e66ce1a
Status: Downloaded newer image for mariadb:10
30634831d17108aa553a5774e27f398760bdbdf32debc3179843e73aa5957956

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
30634831d171        mariadb:10      "docker-entrypoint.s…"   20 seconds ago      Up 16 seconds       3306/tcp            wordpress-db
```

El principal cambio en `docker run` con respecto a la última vez es que no hemos usado
`-p` (el parámetro para publicar puertos) y hemos añadido el parámetro `-d`.

Lo primero que habremos notado es que el contenedor ya no se queda en primer plano. El parámetro `-d` indica que debe ejecutarse como un proceso en segundo plano. Así no podremos pararlo por accidente con ++ctrl+c++.

Lo segundo es que vemos que el contenedor usa un puerto, el `3306/tcp`, pero no está linkado a la máquina anfitrión. No tenemos forma de acceder a la base de datos directamente. Nuestra intención es que solo el contenedor de _WordPress_ pueda acceder.

Luego una serie de parámetros `-e` que nos permite configurar nuestra base de datos.

!!! info
    Los contenedores se configuran a través de variables de ambiente, que podemos configurar con el parámetro `-e` que vemos en la orden anterior. Gracias a ellos hemos creado una base de datos, un usuario y configurado las contraseñas.

    Se recomienda buscar en el registro de _Docker_ la imagen oficial de [MariaDB](https://hub.docker.com/_/mariadb/) para entender el uso de los parámetros.

Por último, el parámetro `--mount` nos permite enlazar el volumen que creamos en el paso anterior con el directorio `/var/lib/mysql` del contenedor. Ese directorio es donde se guardan los datos de _MariaDB_. Eso significa que si borramos el contenedor, o actualizamos el contenedor a una nueva versión, no perderemos los datos porque ya no se encuentran en él, si no en el volumen. Solo lo perderíamos si borramos explícitamente el volumen.

!!! warning
    Cada contendor que usemos tendrá uno o varios directorios donde se deben guardar los datos no volátiles. Nos corresponde a nosotros conocer la herramienta y saber de qué directorios se tratan. Usualmente están en la documentación del contenedor, pero no siempre.

!!! info
    El parámetro `--mount` se empezó a utilizar desde la versión `17.06` para contenedores independientes (los que no pertenecen a un enjambre o _swarm_). Los que conozcan _Docker_ de versiones más antiguas estarán más acostumbrados a usar el parámetro `--volume` que hace algo similar. Sin embargo la documentación aconseja usar ya `--mount`, sobre todo para nuevos usuarios.

    Nosotros somos muy obedientes así que en este taller usaremos `--mount`.

## Creando nuestro blog

Vamos a crear otra vez nuestro contenedor de _WordPress_, pero esta vez vamos a conectarlo con nuestra base de datos. Además, queremos poder editar los ficheros de las plantillas, por si tenemos que modificar algo, así que necesitaremos montar el directorio del contenedor donde está instalado _WordPress_ con nuestra cuenta de usuario en la máquina anfitrión.

!!! example
    Vamos a crear el espacio de trabajo:

        mkdir -p ~/Sites/wordpress/target && cd ~/Sites/wordpress

!!! example
    Y dentro de este directorio arrancamos el contenedor:

        docker run -d --name wordpress \
            --link wordpress-db:mysql \
            --mount type=bind,source="$(pwd)"/target,target=/var/www/html \
            -e WORDPRESS_DB_USER=manager \
            -e WORDPRESS_DB_PASSWORD=secret \
            -p 8080:80 \
            wordpress:6

Cuando termine la ejecución, si accedemos a la dirección [http://localhost:8080/](http://localhost:8080/), ahora sí podremos acabar el proceso de instalación de nuestro WordPress. Si listamos el directorio target comprobaremos que tenemos todos los archivos de instalación accesibles desde el directorio anfitrión.

!!! note
    Ejercicios:

    1. Para los contenedores, tanto el de _WordPress_ como el _MariaDB_.
    2. Borra ambos.
    3. Vuelve a crearlos y mira como ya no es necesario volver a instalar _WordPress_.
    4. Vuelve a borrarlos y borra también el volumen.
    5. Vuelve a crear el volumen y los contenedores y comprueba que ahora sí hay que volver a instalar _WordPress_.

