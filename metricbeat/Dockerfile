# Usa la imagen base oficial de Metricbeat
FROM docker.elastic.co/beats/metricbeat:8.11.1

# Copia tu configuración personalizada de Metricbeat en la imagen
COPY ./metricbeat.yml /usr/share/metricbeat/metricbeat.yml

# Copia las configuraciones de módulo personalizadas
COPY ./nginx.yml /usr/share/metricbeat/modules.d/nginx.yml
COPY ./mysql.yml /usr/share/metricbeat/modules.d/mysql.yml

# Establece los permisos adecuados para los archivos de configuración
USER root
RUN chown root:metricbeat /usr/share/metricbeat/metricbeat.yml
RUN chown root:metricbeat /usr/share/metricbeat/modules.d/nginx.yml
RUN chown root:metricbeat /usr/share/metricbeat/modules.d/mysql.yml

# El usuario por defecto del contenedor es metricbeat
USER metricbeat

# Define el punto de entrada
ENTRYPOINT ["/usr/share/metricbeat/metricbeat"]
