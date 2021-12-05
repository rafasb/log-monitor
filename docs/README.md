# Monitorización de logs

En este conjunto de contenedores se emplea:

* **fluentd** para recoger los logs de los diferentes equipos, sistemas o aplicaciones.

* **graylog** analizar, presentar gráficas y generar alarmas en base a los logs recibidos.

* **elasticsearch** lo emplea graylog para almacenamiento e indexado de los logs.

* **mongodb** lo emplea graylog para almacenamiento de datos de la aplicación.

Aunque este ejemplo está pensado para un solo host, cada una de las herramientas empleadas puede escalar en función de las necesidades, para lo que sería necesario adaptar los diferentes ficheros de configuración.

## Pre-requisitos

En el equipo o máquina virtual donde vamos a instalar el servidor wordpress:

1) Instalar docker: https://docs.docker.com/engine/install/ y https://docs.docker.com/engine/install/linux-postinstall/ 

2) Instalar docker-compose: https://docs.docker.com/compose/install/

Asegurate de:

* El usuario debe disponer de permisos de `sudo`.

* Crear grupo docker y añadir al usuario al grupo `usermod -a -G docker $USER`. Hacer logout y login.

* Si vas a usar GIT no solo para clonar: Instalar y configurar GIT (Claves, user.email y user.name). 

## Instrucciones básicas

1) Clonar el directorio con: `git clone https://github.com/rafasb/log-monitor.git`

2) Cambiar el directorio con `cd log-monitor`

## Parametrización

En este ejemplo, el fichero `docker-compose.yml` establece que algunas aplicaciones (fluentd, graylog y elastichsearch) generarán las imágenes en base a diferentes ficheros de definición (`Dockerfile`) y de los ficheros de configuración de dichas aplicaciones. Esto nos aporta mayor capacidad de personalización de dichas aplicaciones.

Veamos como se parametrizan cada una de las aplicaciones.

### Fluentd

En el fichero docker-compose.yml disponemos de los puertos que se abrirán para recibir los logs de las diversas fuentes. Los puertos internos (puertosExterno:puertoInterno) deben de coincidir con los puertos establecidos en la configuración almacenada en el fichero `./Dockerfiles/fluentd/config/fluentd.conf`

**Configuración de fluentd**: Se establece en el fichero `fluentd.conf`. Se observan en el ejemplo dos tipos de cláusula:

1) **source**: Establece los puertos de entrada de los logs, marca las entradas con una etiqueta e incluso permite manipular (parse) los logs para asociarle campos mediante el uso de expresiones regulares.

2) **match**: Establece donde entregar los flujos de logs. En base a las etiquetas definidas en las clausulas `source` se determina el tipo de registro y el host de destino.

### Elasticsearch

En este caso se han dejado los parámetros por defecto, tanto en la imagen a generar como en los puertos disponibles. Estos puertos se emplearán en el fichero de configuración de `graylog`.

### Graylog

Este es la aplicación más compleja de configurar. Además de configurar antes de arrancar los contenedores, debemos de configurarla mediante la interfaz gráfica para incluir los orígenes de datos, como se analizan y las alertas.

**docker-compose.yml**
En el apartado de la aplicación dentro del fichero docker-compose.yml encontramos el puerto donde se muestra el interfaz gráfico (9000) y puertos de entrada de logs. En este ejemplo emplearemos el puerto **12201/udp** que hemos definido en la aplicación `fluentd`. Estándo en el mismo host, no sería necesario exponer los puertos, pero en el ejemplo se exponen por si se prefiere dividir los contenedores en diferentes máquinas, para escalar la solución. Un ejemplo es el uso que hace graylog de mongodb. Como se puede observar, mongodb no expone ningún puerto y graylog tiene establecido un enlace con este contenedor.

Es muy importante establecer el nombre correcto del host (FQDN) en el parámetro GRAYLOG_WEB_ENDPOINT_URI 

También deberías asegurarte que los parámetros y en especial GRAYLOG_ROOT_PASSWORD_SHA2 coinciden con los establecidos en el fichero de configuración **graylog.conf** descrito en el siguiente apartado.

**graylog.conf**
El fichero se encuentra en `Dockerfiles/graylog/config/graylog.conf` es conveniente revisar el contenido y personaliza al menos los siguientes campos:
* password_secret 
* root_password_sha2: Debe coincidir con el establecido en el parámetro GRAYLOG_ROOT_PASSWORD_SHA2 del docker-compose.yml

Si modificados los puertos de elasticsearch o trasladamos el contenedor a otro equipo, será necesario personalizar los campos relativos a esta aplicación en este fichero.

## Arranque la solución

Como siempre, arrancar la aplicación es extremadamente fácil. Usamos el comando `docker-compose up -d` en el directorio clonado del repositorio git. Una vez arrancada debemos acceder a Graylog mediante `http://<IPdelHost>:9000`

**Interfaz gráfico**

En el primer acceso el usuario por defecto es `admin` y la contraseña por defecto es `admin`

Ahora debemos configurar la entrada de datos desde fluentd, para ello accedemos a **System** > **Inputs** y añadimos el tipo **GELF UDP**, ya que así lo hemos definido en el fichero de configuración de fluentd. Es importante que modifiquemos el valor de **Receive Buffer Size** para establecerlo en **212992**

Realizados estos pasos, ya podemos visualizar los datos y empezar a gestionar los logs.


## Referencias
https://github.com/greggilbert/fluentd-graylog

https://docs.fluentd.org/input/syslog#source_hostname_key

https://docs.graylog.org/docs/docker

https://marketplace.graylog.org/addons?search=


## To Do
Segregar las redes contenedores para aislarlos del exterior. 


