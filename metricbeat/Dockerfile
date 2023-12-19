# Usar la imagen base oficial de Metricbeat
FROM docker.elastic.co/beats/metricbeat:8.11.1

# Cambiar al usuario root para poder cambiar los permisos
USER root

# Copiar la configuración personalizada de Metricbeat en la imagen
COPY metricbeat.yml /usr/share/metricbeat/metricbeat.yml
COPY nginx.yml /usr/share/metricbeat/modules.d/nginx.yml
COPY mysql.yml /usr/share/metricbeat/modules.d/mysql.yml

# Cambiar los permisos del archivo de configuración
RUN chmod go-w /usr/share/metricbeat/metricbeat.yml
RUN chmod go-w /usr/share/metricbeat/modules.d/nginx.yml
RUN chmod go-w /usr/share/metricbeat/modules.d/mysql.yml

# Vuelve al usuario metricbeat
USER metricbeat


# Define el punto de entrada
ENTRYPOINT ["/usr/share/metricbeat/metricbeat"]
