# Contenedores

Los contenedores son instancias de las imágenes que hemos creado o hemos descargado que se ejecutan de forma aislada.

## Listado

La orden para ver el listado de contenedores del sistema es `docker container ls` o la forma abreviada `docker ps`. Si lo ejecutamos nos dará un listado vacío porque no hay ningún contenedor activo.

Probemos con el parámetro `--all` o `-a`.

    $ docker container ls -a
    CONTAINER ID  IMAGE        COMMAND     CREATED         STATUS      PORTS  NAMES
    4bd76e08b07f  wordpress    "docker-…"  11 minutes ago  Exited (0)         peaceful_murdock
    69a3c34c224d  hello-world  "/hello"    18 minutes ago  Exited (0)         blissful_goldwasser

Estos contenedores están parados y se pueden volver a ejecutar, con el mismo estado que tuviera el sistema de archivos cuando se detuvieron.

## Iniciar un contenedor

Con `docker start` podemos iniciar un contenedor parado:

    $ docker container start peaceful_murdock 
    peaceful_murdock
    $ docker ps
    CONTAINER ID  IMAGE      COMMAND    CREATED         STATUS  PORTS                 NAMES
    4bd76e08b07f  wordpress  "docker…"  14 minutes ago  Up      0.0.0.0:8080->80/tcp  peaceful_murdock

Veremos que la web de instalación de _WordPress_ está de nuevo disponible. Solo que ahora el contenedor se ejecuta en segundo plano y no lo podemos detener como antes.

## Detener un contenedor

Con `docker stop` podemos detener un contenedor iniciado, indicando su id o su nombre

    $ docker container stop 4bd76e08b07f
    4bd76e08b07f

!!! tip
    Podemos hacer referencia a los contenedores por su ID o por su nombre.

## Borrar un contenedor

Un contenedor detenido ocupa espacio. Si hemos dejado de necesitar un contenedor podemos borrarlo con `docker rm`. Igualmente hay que indicar id o nombre.

    $ docker container rm 4bd76e08b07f
    4bd76e08b07f

!!! danger
    Hay que tener cuidado al borrar contenedores. Cuando un contenedor se borra se elimina cualquier información que contenga y no esté almacenada en algún lugar externo al propio contenedor.

