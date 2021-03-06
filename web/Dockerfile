FROM    debian
MAINTAINER Wellington Silva <wsilva@usp.br>

# Keep upstart from complaining
RUN dpkg-divert --local --rename --add /sbin/initctl && ln -sf /bin/true /sbin/initctl

# Let the conatiner know that there is no tty
ENV DEBIAN_FRONTEND noninteractive

# we need docker user, install wget and path /src
RUN useradd -u 1000 docker && apt-get update && apt-get install -y wget && mkdir /src && chown docker:docker /src

# add dotdeb source list
RUN echo "deb http://packages.dotdeb.org wheezy all" > /etc/apt/sources.list.d/dotdeb.list \
    && echo "deb-src http://packages.dotdeb.org wheezy all" >> /etc/apt/sources.list.d/dotdeb.list \
    && wget -O - http://www.dotdeb.org/dotdeb.gpg |apt-key add -

# installing php stuff
RUN apt-get update && apt-get install -y cmake make g++ git whiptail mlocate net-tools curl procps \
    vim nmap re2c libsnappy1 libsnappy-dev php5-cli php5-fpm php5-mysql php5-intl php5-xdebug php5-recode \
    php5-snmp php5-mcrypt php5-memcache php5-memcached php5-imagick php5-curl php5-xsl php5-snmp \
    php5-dev php5-tidy php5-xmlrpc php5-gd php5-pspell php-pear php-apc nginx pkg-config librabbitmq-dev \
    librabbitmq0

# setting up php.ini, fpm pool conf and nginx.conf and installing composer
RUN sed -i "s/;date.timezone =/date.timezone = America\/Sao_Paulo/" /etc/php5/cli/php.ini \
    && sed -i "s/;date.timezone =/date.timezone = America\/Sao_Paulo/" /etc/php5/fpm/php.ini \
    && sed -i "s/short_open_tag = On/short_open_tag = Off/" /etc/php5/cli/php.ini \
    && sed -i "s/short_open_tag = On/short_open_tag = Off/" /etc/php5/fpm/php.ini \
    && sed -i "s/error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT/error_reporting = E_ALL/" /etc/php5/cli/php.ini \
    && sed -i "s/error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT/error_reporting = E_ALL/" /etc/php5/fpm/php.ini \
    && sed -i "s/display_errors = Off/display_errors = On/" /etc/php5/cli/php.ini \
    && sed -i "s/display_errors = Off/display_errors = On/" /etc/php5/fpm/php.ini \
    && sed -i "s/display_startup_errors = Off/display_startup_errors = On/" /etc/php5/cli/php.ini \
    && sed -i "s/display_startup_errors = Off/display_startup_errors = On/" /etc/php5/fpm/php.ini \
    && sed -i "s/www-data/docker/g" /etc/php5/fpm/pool.d/www.conf \
    && sed -i "s/www-data;/docker;\\ndaemon off;/g" /etc/nginx/nginx.conf \
    && curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/bin/composer

# setting xdebug
COPY xdebug.ini /etc/php5/mods-available/xdebug.ini
RUN echo "\n\nxdebug.remote_host="`/sbin/ip route|awk '/default/ { print $3 }'` >> /etc/php5/mods-available/xdebug.ini

# compiling rabbitmq lib and installing amqp, igbinary and php-ext-snappy extension
WORKDIR /
RUN git clone https://github.com/alanxz/rabbitmq-c && git clone https://github.com/phadej/igbinary && git clone git://github.com/goatherd/php-snappy.git
WORKDIR /rabbitmq-c
RUN git checkout v0.5.2 \
    && autoreconf -i \
    && ./configure \
    && make \
    && make install \
    && pecl install amqp \
    && echo "extension=amqp.so" > /etc/php5/mods-available/amqp.ini
WORKDIR /igbinary
RUN phpize \
    && ./configure CFLAGS="-O2 -g" --enable-igbinary --with-php-config=/usr/bin/php-config \
    && make \
    && make install \
    && echo -e "\n\n[igbinary]\n;https://github.com/igbinary/igbinary/\nextension=igbinary.so\n\n;Use igbinary as session serializer\nsession.serialize_handler=igbinary\n\n;Enable or disable compacting of duplicate strings\nigbinary.compact_strings=On\n\n;Use igbinary as serializer in APC cache\n;apc.serializer=igbinary" > /etc/php5/mods-available/igbinary.ini
WORKDIR /php-snappy
RUN phpize \
    && ./configure \
    && make \
    && make install \
    && echo "extension=snappy.so" > /etc/php5/mods-available/snappy.ini
WORKDIR /etc/php5/mods-available
RUN php5enmod amqp && php5enmod igbinary && php5enmod snappy
WORKDIR /
RUN rm -rf rabbitmq-c/ && rm -rf igbinary/ && rm -rf php-snappy/

COPY run.sh /run.sh
RUN chmod a+x /run.sh

COPY teste.dev.conf /etc/nginx/sites-available/teste.dev.conf

# virtualhosts configuration
RUN ln -sf /etc/nginx/sites-available/teste.dev.conf /etc/nginx/sites-enabled/teste.dev.conf

# removing deb files
RUN apt-get clean

EXPOSE 80

USER docker

ENTRYPOINT ["/run.sh"]
