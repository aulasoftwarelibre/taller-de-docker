# Crear imágenes propias

Ya hemos visto como usar imágenes de terceros para crear aplicaciones y servicios. Pero, ¿si no hay ninguna imagen que tenga lo que queremos? ¿O si queremos hacer una imagen de nuestra aplicación para distribuirla?

Docker permite crear imagenes propias. Aunque podríamos hacerla partiendo de cero, es un esfuerzo que no tiene sentido. Existe ya imágenes base para crear las nuestras y es mucho más fácil crear una imagen basándose en otra que hacerlo todo nosotros.

Podemos partir de una imagen base que parte de un lenguaje de programación (_python_, _php_) o de alguna distribución (_ubuntu_, _debian_).

## Mi primer Dockerfile

Los _Dockerfile_ son los archivos que contienen las instrucciones que crean las imagenes. Deben estar guardados dentro de un _build context_, es decir, un directorio. Este directorio es el que contiene todos los archivos necesarios para construir nuestra imagen, de ahí lo de _build context_.

Creamos nuestro _build context_

    mkdir -p  ~/Sites/hello-world
    cd ~/Sites/hello-world
    echo "hello" > hello

Dentro de este directorio crearemos un archivo llamado _Dockerfile_ con este contenido:

    :::env
    FROM busybox
    COPY /hello /
    RUN cat /hello

| Directiva | Explicación |
|--|--|
| FROM | Indica la imagen base sobre la que se basa esta imagen |
| COPY | Copia un archivo del _build context_ y lo guarda en la imagen |
| RUN | Ejecuta el comando indicado durante el proceso de creación de imagen. |

Ahora para crear nuestra imagen usaremos `docker build`.

    docker build -t helloapp:v1 .

El parámetro `-t` nos permite etiquetar la imagen con un nombre y una versión. El `.` indica que el _build context_ es el directorio actual.

El resultado de ejecutar lo anterior sería:

    $ docker build -t helloapp:v1 .
    Sending build context to Docker daemon  3.072kB
    Step 1/3 : FROM busybox
    latest: Pulling from library/busybox
    8c5a7da1afbc: Pull complete 
    Digest: sha256:cb63aa0641a885f54de20f61d152187419e8f6b159ed11a251a09d115fdff9bd
    Status: Downloaded newer image for busybox:latest
     ---> e1ddd7948a1c
    Step 2/3 : COPY /hello /
     ---> 8a092965dbc9
    Step 3/3 : RUN cat /hello
     ---> Running in 83b5498790ca
    hello
    Removing intermediate container 83b5498790ca
     ---> f738f117d4b6
    Successfully built f738f117d4b6
    Successfully tagged helloapp:v1

Y podremos ver que una nueva imagen está instalada en nuestro equipo:

    $ docker images
    REPOSITORY   TAG  IMAGE ID      CREATED         SIZE
    helloapp     v1   f738f117d4b6  40 seconds ago  1.16MB

## Creando aplicaciones en contenedores

Vamos a crear un aplicación en python y la vamos a guardarla en un contenedor. Comenzamos creando un nuevo _build context_:

    mkdir -p  ~/Sites/friendlyhello
    cd ~/Sites/friendlyhello

El código de la aplicación es el siguiente, lo guardaremos en un archivo llamado `app.py`:

    :::python
    from flask import Flask
    from redis import Redis, RedisError
    import os
    import socket

    # Connect to Redis
    redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

    app = Flask(__name__)

    @app.route("/")
    def hello():
        try:
            visits = redis.incr("counter")
        except RedisError:
            visits = "<i>cannot connect to Redis, counter disabled</i>"

        html = "<h3>Hello {name}!</h3>" \
               "<b>Hostname:</b> {hostname}<br/>" \
               "<b>Visits:</b> {visits}"
        return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(),  visits=visits)

    if __name__ == "__main__":
        app.run(host='0.0.0.0', port=80)

Nuestra aplicación tiene una serie de dependencias (librerías de terceros) que guardaremos en el archivo _requirements.txt_:

    Flask
    Redis

Y por último definimos nuestro _Dockerfile_:

    :::env
    # Partimos de una base oficial de python
    FROM python:2.7-slim

    # El directorio de trabajo es desde donde se ejecuta el contenedor al iniciarse
    WORKDIR /app

    # Copiamos todos los archivos del build context al directorio /app del contenedor
    COPY . /app

    # Ejecutamos pip para instalar las dependencias en el contenedor
    RUN pip install --trusted-host pypi.python.org -r requirements.txt

    # Indicamos que este contenedor se comunica por el puerto 80/tcp
    EXPOSE 80

    # Declaramos una variable de entorno
    ENV NAME World

    # Ejecuta nuestra aplicación cuando se inicia el contenedor
    CMD ["python", "app.py"]

En total debemos tener 3 archivos:

    $ ls
    app.py  Dockerfile  requirements.txt

Ahora construimos la imagen de nuestra aplicación:

    docker build -t friendlyhello .

Y comprobamos que está creada:

    $ docker image ls
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    friendlyhello       latest              88a822b3107c        56 seconds ago      132MB

### Probar nuestro contenedor

Vamos a arrancar nuestro contenedor y probar la aplicación:

    docker run --rm -p 4000:80 friendlyhello

!!! tip
    Normalmente los contenedores son de usar y tirar, sobre todo cuando hacemos pruebas. El parámetro `--rm` borra automáticamente un contenedor cuando se para. Recordemos que los datos volátiles siempre se deben guardar en volúmenes.

Lo que arranca la aplicación Flask:

    $ docker run --rm -p 4000:80 friendlyhello
     * Serving Flask app "app" (lazy loading)
     * Environment: production
       WARNING: Do not use the development server in a production environment.
       Use a production WSGI server instead.
     * Debug mode: off
     * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)

Comprobamos en el puerto 4000 si efectivamente está iniciada o no: [http://localhost:4000](http://localhost:4000)

Obtendremos un mensaje como este:

    Hello World!

    Hostname: 0367b056e66e
    Visits: cannot connect to Redis, counter disabled

### Creando la aplicación

En este caso nuestro contenedor no funciona por sí mismo. Es muy habitual que dependamos de servicios para poder iniciar la aplicación, habitualmente bases de datos. En este caso necesitamos una base de datos _Redis_ que no tenemos.

Como vimos en el apartado anterior, vamos a aprovechar las características de _Compose_ para levantar nuestra aplicación.

Vamos a crear el siguiente archivo _docker-compose.yaml_:

    :::yaml
    version: "3"
    services:
        web:
            build: .
            ports:
                - "4000:80"
        redis:
            image: redis
            ports:
                - "6379:6379"
            volumes:
                - "./data:/data"
            command: redis-server --appendonly yes

### Balanceo de carga

Vamos a modificar nuestro _docker-compose.yaml_:

    :::yaml
    version: "3"
    services:
        web:
            build: .
        redis:
            image: redis
            ports:
                - "6379:6379"
            volumes:
                - "./data:/data"
            command: redis-server --appendonly yes
        lb:
            image: dockercloud/haproxy
            ports:
                - 80:80
            links:
                - web
            volumes:
                - /var/run/docker.sock:/var/run/docker.sock 


    networks:
        front-tier:
            driver: bridge
        back-tier:
            driver: bridge
