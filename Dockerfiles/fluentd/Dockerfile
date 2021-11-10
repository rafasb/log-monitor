FROM fluent/fluentd:v1.6-debian-1
USER root

#instalación del plugin elasticsearch
# RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-document", "--version", "3.5.2"]

#Instalación del plugin GELF
RUN apt-get update && \
    apt-get install -y ca-certificates openssl wget && \
    gem install gelf

RUN cd /fluentd/plugins && \
    wget https://raw.githubusercontent.com/emsearcy/fluent-plugin-gelf/master/lib/fluent/plugin/out_gelf.rb

USER fluent