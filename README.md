# ELK Stack Docker Configuration

Este archivo `README.md` proporciona instrucciones para el despliegue de un contenedor Docker que ejecuta el ELK Stack (Elasticsearch, Logstash, Kibana) utilizando dockers.

## Requisitos Previos

- Docker instalado en el equipo.
- Docker Compose instalado en el equipo.

## Estructura del proyecto

```
incid-elk/
│
├── elk/
│   ├── etc/
│   │   └── logstash/
│   │       └── conf.d/
│   │           └── 02-beats-input.conf   # Archivo de configuración de Logstash para los Beats.
│   │
│   └── Dockerfile          # Dockerfile para construir la imagen de ELK.
│
├── metribeat/
│   ├── Dockerfile    
│   ├── metricbeat.yml      # Fichero de configuración de metribeat
│   ├── mysql.yml           # Fichero de configuración del módulo de metribeat mysql.yml
│   └── nginx.yml           # Fichero de configuración del módulo de metritbeat nginx.yml
├── nginx-filebeat/
│   ├── html/
│   │   ├── favicon.ico     # Favicon para el sitio web servido por Nginx.
│   │   └── index.html      # Página de inicio HTML para el servidor web Nginx.
│   │
│   ├── Dockerfile          # Dockerfile para construir la imagen de Nginx con Filebeat.
│   ├── filebeat.yml        # Archivo de configuración de Filebeat.
│   ├── logstash-beats.crt  # Certificado SSL para la comunicación segura con Logstash.
│   ├── start.sh            # Script de inicio para ejecutar el servicio ngix con Filebeat
│   ├── docker-compose.yml  # Docker Compose para definir y ejecutar aplicaciones multi-contenedor.
│   ├── LICENSE             # Archivo de licencia del proyecto.
└── └── README.md                           
```


## Documentación del Docker Compose del Proyecto ELK con Nginx, Filebeat y Metricbeat

El archivo `docker-compose.yml` establece la infraestructura necesaria para ejecutar un stack ELK (Elasticsearch, Logstash, Kibana) junto con un servidor web Nginx que también tiene Filebeat instalado para el envío de logs.

### Versión

Se utiliza la versión `3.8` de Docker Compose, asegurando compatibilidad con características recientes de Docker.

### Servicios

#### ELK

El servicio `elk` construye una imagen personalizada a partir de un `Dockerfile` en la carpeta `./elk`.

- **Nombre del contenedor**: `elk`
- **Red**: `elk-net` con una dirección IP estática `172.20.0.10`.
- **Puertos**:
  - `5601`: Puerto para Kibana, permitiendo el acceso a la interfaz de usuario de Kibana desde el host.
  - `9200`: Puerto para Elasticsearch, permitiendo la interacción con el cluster de Elasticsearch desde el host.
  - `5044`: Puerto para Logstash Beats input, utilizado para recibir datos de Filebeat.

### Nginx-Filebeat

El servicio `nginx-filebeat` construye una imagen personalizada a partir de un `Dockerfile` en la carpeta `./nginx-filebeat`.

- **Nombre del contenedor**: `nginx-filebeat`
- **Red**: Se conecta a la red `elk-net` definida.
- **Puertos**:
  - `80`: Puerto para el servidor web Nginx, permitiendo el acceso al sitio web desde el host.

## Redes

### elk-net

Define una red personalizada tipo `bridge` con el siguiente rango de subnet: `172.20.0.0/24`.

- **Driver**: `bridge` para la comunicación entre contenedores.
- **IPAM Config**:
  - `subnet`: "172.20.0.0/24" define la red en la que los servicios pueden comunicarse entre sí.

## Configuración Detallada

### Servicio ELK

La imagen de ELK se construye a partir de la imagen base `sebp/elk:8.11.1`. Esta imagen contiene Elasticsearch, Logstash y Kibana, las cuales son configuradas para trabajar juntas dentro de un solo contenedor Docker.

#### Dockerfile de ELK

El `Dockerfile` especifica lo siguiente:

- **Imagen Base**: `sebp/elk:8.11.1` es la imagen base que ya incluye las aplicaciones ELK.
- **Configuración de Logstash**: Se copia una configuración personalizada de Logstash desde el directorio del host `./etc/logstash/conf.d/` al directorio correspondiente dentro del contenedor. Esta configuración está diseñada para recibir logs de Filebeat.

#### Configuración de Logstash

La configuración personalizada de Logstash (`02-beats-input.conf`) define un input para Beats en el puerto `5044` y deshabilita SSL (para este ejemplo, aunque no es lo recomedado):

```ruby
input {
    beats {
        port => 5044
        ssl => false  # En esta versión se deshabilita para evitar problemas de certificados inválidos.
        # ssl_certificate => "/etc/pki/tls/certs/logstash-beats.crt"
        # ssl_key => "/etc/pki/tls/private/logstash-beats.key"
    }
}
```

### Servicio Nginx con Filebeat

El `Dockerfile` está diseñado para construir una imagen Docker que ejecuta Nginx junto con Filebeat, permitiendo la recopilación y el envío de logs de Nginx a Logstash.

#### Imagen Base

`FROM nginx`: Esta imagen usa la imagen oficial de Nginx como base.

#### Mantenedor

MAINTAINER [Sebastien Pujadas](https://github.com/spujadas/elk-docker/tree/oss-7.13.3/nginx-filebeat): Información sobre el autor y mantenedor de la imagen Docker.

#### Variables de Entorno

`ENV REFRESHED_AT`: Se utiliza para invalidar la caché del Dockerfile.
`ENV FILEBEAT_VERSION`: Define la versión de Filebeat que se instalará. **Es muy importante que coincida con la de ELK**

#### Instalación

- Se actualizan los paquetes y se instala `curl`.
- Se descarga e instala Filebeat utilizando la versión especificada en la variable de entorno.

#### Configuración

- Se eliminan los enlaces simbólicos de los logs de Nginx para permitir que Filebeat los monitoree directamente.
- Se añade el archivo `filebeat.yml` al contenedor y se establecen los permisos adecuados.
- Se crea una carpeta para los certificados SSL y se añade el certificado `logstash-beats.crt`.
- Se exporta la plantilla de Filebeat basada en la versión de Elasticsearch *asumiendo que es la misma que la de Filebeat*

#### Datos

- Se copia una estructura de directorio `html` en la ubicación de Nginx para servir archivos estáticos.

#### Inicio

- Se añade el script `start.sh` al contenedor y se le dan permisos de ejecución.
- `CMD [ "/usr/local/bin/start.sh" ]`: Se establece el comando por defecto para ejecutar el script de inicio.


#### Script de Inicio (`start.sh`)

El script realiza las siguientes acciones:

- Carga la plantilla de Filebeat en Elasticsearch.
- Inicia el servicio de Filebeat.
- Inicia el servidor web Nginx.
- Mantiene abierta la salida del log con `tail` para `access.log` y `error.log`.


#### Fichero de Configuración de Filebeat (`filebeat.yml`)

El archivo `filebeat.yml` configura Filebeat para enviar logs a Logstash:

- Define la salida hacia Logstash, especificando la dirección y el puerto del servicio de Logstash.
- Configura dos entradas de Filebeat:
  - Logs del sistema (`syslog` y `auth.log`).
  - Logs de Nginx (`/var/log/nginx/*.log`), marcando los campos como parte de la raíz del documento.

La sección de SSL está comentada, porque en esta versión no se utiliza SSL/TLS para la conexión a Logstash en esta configuración.

## Instrucciones de Uso

1. **Inicio del Contenedor**: Para iniciar el contenedor ELK, ejecuta el siguiente comando en el directorio donde se encuentra el archivo `docker-compose.yml`:

    ```bash
    docker-compose up -d
    ```

    Este comando descargará la imagen (si aún no lo está) y arrancará el contenedor.

2. **Acceso a los Servicios**: Una vez que el contenedor esté en arrancado, se puede acceder a:
    - Kibana: [http://localhost:5601](http://localhost:5601)
    - Elasticsearch: [http://localhost:9200](http://localhost:9200)

3. **Detener el Contenedor**: Para detener y eliminar el contenedor y la red, utilizar:

    ```bash
    docker-compose down
    ```

4. **Comprobar parámetros de la red**: Para obtener información detallada de la red, utilizar:

    ```bash
    docker network inspect incid-elk_elk-net
    ```

## Mantenimiento y Soporte

Este contenedor es adecuado para propósitos de desarrollo y prueba. Para ambientes de producción, se debe considerar instalar y configurar el slack elk cada uno por separado e implementar todas las medidas de seguridad necesarias.
