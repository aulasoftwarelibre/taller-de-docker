# Introducción

Según la Wikipedia: "Docker es un proyecto de código abierto que automatiza el despliegue de aplicaciones dentro de contenedores de software, proporcionando una capa adicional de abstracción y automatización de virtualización de aplicaciones en múltiples sistemas operativos. Docker utiliza características de aislamiento de recursos del kernel Linux, tales como cgroups y espacios de nombres (namespaces) para permitir que 'contenedores' independientes se ejecuten dentro de una sola instancia de Linux, evitando la sobrecarga de iniciar y mantener máquinas virtuales."[^1].

[^1]: Wikipedia: [https://es.wikipedia.org/wiki/Docker_(software)](https://es.wikipedia.org/wiki/Docker_(software))

Esto es una descripción formal, pero para enterarte de qué es docker sigue leyendo:

## ¿A quién le puede interesar usar docker?

Docker es útil a administradores de sistemas, pero también a desarrolladores. Uno de los problemas que se presentan durante el desarrollo y despliegue de aplicaciones es encontrarnos con sistemas heterogéneos, no ya entre los desarrolladores, también entre los sistemas de pruebas, pre-producción y producción. Es decir, que los desarrolladores y los sistemas donde se ejecuta la aplicación tienen librerías y sistemas operativos diferentes. ¿Y por qué es un problema? Pues porque la aplicación puede funcionar bien en una distribución de GNU/Linux pero no bien en otra, o ejecutarse bien con la versión de un lenguaje pero no con otra. Para asegurar la calidad de desarrollo tenemos que asegurar que todo el mundo usa las mismas versiones de todas las aplicaciones y librerías necesarios.

Esto es más complicado de lo que parece, porque hay desarrolladores que prefieron una distribución concreta, o incluso sistemas privativos. Incluso los sistemas de pruebas, pre-producción y producción suelen ser distintos. Los sistemas de producción suelen ser más nuevos y potentes y los antiguos se dejan para pruebas y pre-producción.

Otro problema es que un mismo desarrollador o un mismo sistema de despliegue tenga que trabajar en más de un proyecto que requiera versiones distintas de librerías, complicándolo aún más.

Docker viene a solucionar todos estos problemas, tanto para los desarrolladores como para los administradores de sistemas. Con Docker podemos crear entornos aislados con configuraciones que serán exactamente igual siempre.

## ¿Docker es virtualización?

En GNU/Linux Docker no es virtualizado, no hay un hipervisor. Los procesos que corren dentro de un contenedor de docker se ejecutan con el mismo kernel que la máquina anfitrión. Linux lo que hace es aislar esos procesos del resto de procesos del sistema, ya sean los propios de la máquina anfitrión o procesos de otros contenedores. Además, es capaz de controlar los recursos que se le asignan a esos contenedores (cpu, memoria, red, etc.). Internamente, el contenedor no sabe que lo es y a todos los efectos es una distribución GNU/Linux independiente, pero sin la penalización de rendimiento que tienen los sistemas virtualizados.

Así que, cuando ejecutamos un contenedor, estamos ejecutando un servicio dentro de una distribución construida a partir de una "receta". Esa receta permite que el sistema que se ejecuta sea siempre el mismo, independientemente de si estamos usando Docker en Ubuntu, Fedora o, incluso, sistemas privativos compatibles con Docker. De esa manera podemos garantizar que estamos desarrollando o desplegando nuestra aplicación, siempre con la misma versión de todas las dependencias.

Obviamente, si ejecutamos contenedores GNU/Linux dentro de sistemas privativos, sí habrá virtualización.

## Conceptos básicos

Antes de comenzar a instalar y usar docker es importante tener una serie de conceptos claros[^2]:

Demonio de docker (docker daemon)
: Es el proceso principal de docker. Escucha peticiones a la API y maneja los objetos de docker: imágenes, contenedores, redes, volúmenes. También es capaz de comunicarse con otros demonios para controlar servicios docker.

Cliente de docker (docker client)
: Es la principal herramienta que usan los administradores de sistema para interaccionar con el sistema Docker.

Registro de docker (docker registry)
: Es el lugar donde se almacenan las imágenes de Docker y poder descargarlas para reutilizarlas. [Docker Hub](https://hub.docker.com/) es el principal registro público de Docker y contiene ya un montón de imágenes listas para ser usadas de multitud de servicios (mysql, wordpress, etc).

[^2]: [Official Docker Documentation: Docker overview](https://docs.docker.com/engine/docker-overview/#docker-objects).

### Objetos de docker

Cuando usamos Docker, estamos creando y usando imágenes, contenedores, redes o volúmenes, entre otros. A todo esto se le denominan objetos. Veamos los más importantes:

Imagen (image)
: Plantilla de solo lectura que contiene las instrucciones para crear un contenedor Docker. Pueden estar basadas en otras imagenes, lo cual es habitual.

Contenedor (container)
: Es una instancia ejecutable de una imagen. Esta instancia puede ser creada, iniciada, detenida, movida o eliminada a través del cliente de Docker o de la API. Las instancias se pueden conectar a una o más redes, sistemas de almacenamiento, o incluso se puede crear una imagen a partir del estado de un contenedor. Se puede controlar cómo de aislado está el contenedor del sistema anfitrión y del resto de contenedores.
    El contenedor está definido tanto por la imagen de la que procede como de las opciones de configuración que permita. Por ejemplo, la imagen oficial de _MariaDb_ permite configurar a través de opciones la contraseña del administrador, de la primera base de datos que se cree, del usuario que la maneja, etc.

Servicios (services)
: Los servicios permiten escalar contenedor a través de múltiples demonios de Docker, los cuales trabajarán conjuntamente como un enjambre (swarm).

## Objetivos del taller

En este taller aprenderemos:

* A usar el registro oficial de Docker, a descargar y usar imágenes del mismo creando contenedores que nos puedan ser útiles.
* A crear una imagen a partir de otra.
* A automatizar la creación de un conjunto de contenedores que interaccionan entre sí.

En este taller no aprenderemos:

* A crear un cluster con Docker (Docker Swarm).
* A administrar sistemas. Se darán ya las instrucciones necesarias para crear servicios pero explicarlos está más alla del ámbito de este taller. Se recomienda cursar la asignatura: "Programación y Administración de Sistemas" del Grado de Ingeniería Informática.
