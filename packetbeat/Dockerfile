FROM docker.elastic.co/beats/packetbeat:7.12.0
COPY packetbeat.yml /usr/share/packetbeat/packetbeat.yml
USER root
RUN chown root:packetbeat /usr/share/packetbeat/packetbeat.yml
RUN chmod +x /usr/share/packetbeat/packetbeat
RUN mkdir -p /var/log/packetbeat && \
    chown packetbeat:packetbeat /var/log/packetbeat && \
    chmod 755 /var/log/packetbeat
USER packetbeat
