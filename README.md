# ELK Stack Docker Configuration

Este archivo `README.md` proporciona instrucciones para el despliegue de un contenedor Docker que ejecuta el ELK Stack (Elasticsearch, Logstash, Kibana) utilizando `docker-compose`.

## Requisitos Previos

- Docker instalado en el equipo.
- Docker Compose instalado en el equipo.

## Configuración

El archivo `docker-compose.yml` está configurado para lanzar un contenedor que ejecuta ELK Stack versión 7.16.3. Las siguientes son las configuraciones:

- **Imagen Docker**: `sebp/elk:7.16.3`.
- **Puertos**: Los puertos 5601, 9200 y 5044 del contenedor están mapeados a los mismos puertos en el host.
- **Red Docker**: Una red de tipo bridge `elk-net` con un rango de IP `172.20.0.0/24`.
- **Dirección IP del Contenedor**: `172.20.0.10`.

## Instrucciones de Uso

1. **Inicio del Contenedor**: Para iniciar el contenedor ELK, ejecuta el siguiente comando en el directorio donde se encuentra el archivo `docker-compose.yml`:

    ```bash
    docker-compose up -d
    ```

    Este comando descargará la imagen (si aún no lo está) y arrancará el contenedor.

2. **Acceso a los Servicios**: Una vez que el contenedor esté en arrancado, se puede acceder a:
    - Kibana: [http://localhost:5601](http://localhost:5601)
    - Elasticsearch: [http://localhost:9200](http://localhost:9200)

3. **Detención del Contenedor**: Para detener y eliminar el contenedor y la red, utilizar:

    ```bash
    docker-compose down
    ```

## Mantenimiento y Soporte

Este contenedor es adecuado para propósitos de desarrollo y prueba. Para ambientes de producción, se debe considerar instalar y configurar el slack elk cada uno por separado e implementar todas las medidas de seguridad necesarias.
