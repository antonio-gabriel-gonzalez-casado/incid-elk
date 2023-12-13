# Extend from the official ELK image
FROM sebp/elk:7.16.3

# Copy custom Logstash configuration
COPY ./etc/logstash/conf.d/02-beats-input.conf /etc/logstash/conf.d/02-beats-input.conf
