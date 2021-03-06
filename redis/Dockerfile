FROM  debian
MAINTAINER  Wellington Silva <wsilva@usp.br>

RUN apt-get update && apt-get install -y wget make gcc \
    && cd /tmp \
    && useradd -u 1000 docker && mkdir /data && chown docker:docker /data \
    && wget http://download.redis.io/redis-stable.tar.gz \
    && tar xvzf redis-stable.tar.gz \
    && cd redis-stable \
    && make \
    && make install \
    && cp -f src/redis-sentinel /usr/local/bin \
    && mkdir -p /etc/redis \
    && cp -f *.conf /etc/redis \
    && rm -rf /tmp/redis-stable* \
    && sed -i 's/^\(bind .*\)$/# \1/' /etc/redis/redis.conf \
    && sed -i 's/^\(daemonize .*\)$/# \1/' /etc/redis/redis.conf \
    && sed -i 's/^\(loglevel .*\)$/# \1\nloglevel debug/' /etc/redis/redis.conf \
    && sed -i 's/^\(dir .*\)$/# \1\ndir \/data/' /etc/redis/redis.conf \
    && sed -i 's/^\(logfile .*\)$/# \1/' /etc/redis/redis.conf

# Define mountable directories.
VOLUME ["/data"]

# Define working directory.
WORKDIR /data

# run as docker user to be able to read dump file
USER docker

# Define default command.
ENTRYPOINT ["redis-server"]

# default parameter
CMD ["/etc/redis/redis.conf"]

# Expose ports.
EXPOSE 6379