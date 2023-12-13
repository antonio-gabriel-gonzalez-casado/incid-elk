# Lo que se especifique en esta versión hay que tenerlo en cuenta para los beats que se instalen.
FROM sebp/elk:8.11.1

# Copiar la configuración particular de logstash para los beats
COPY ./etc/logstash/conf.d/02-beats-input.conf /etc/logstash/conf.d/02-beats-input.conf
