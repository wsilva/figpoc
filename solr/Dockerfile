FROM  debian
MAINTAINER  Wellington Silva <wsilva@usp.br>

COPY start-solr.sh /start-solr.sh

RUN useradd -u 1000 docker && mkdir /src && chown docker:docker /src

RUN apt-get update -y \
    && apt-get install -y openjdk-7-jre-headless \
    && useradd -u 1000 docker \
    && mkdir /src \
    && chown docker:docker /src \
    && chmod a+x /start-solr.sh

EXPOSE 8983

USER docker

ENTRYPOINT ["/start-solr.sh"]