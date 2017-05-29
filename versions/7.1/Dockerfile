#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "./hack/run update"
#
# DO NOT EDIT IT DIRECTLY.
#
FROM php:7.1-fpm-alpine
# <pre>

MAINTAINER AusNimbus <support@ausnimbus.com.au>

LABEL summary="Platform for running PHP 7.1 applications" \
      maintainer="AusNimbus <support@ausnimbus.com.au>" \
      io.k8s.description="Platform for running PHP 7.1 applications" \
      io.k8s.display-name="PHP 7.1"

ENV HOME=/opt/app-root/src

USER 0

RUN apk --no-cache add bash rsync && \
    mkdir -p ${HOME}/.pki/nssdb && \
    chown -R 1001:0 ${HOME}/.pki && \
    adduser -S -u 1001 -G root -h ${HOME} -s /sbin/nologin \
      -g "Default Application User" default && \
    chown -R 1001:0 /opt/app-root

# Copy common executable utilities.
COPY ./common/bin/ /usr/bin/

# Each container image can have 'contrib' a directory with extra files needed to
# run and build the application.
COPY ./contrib/ /opt/app-root

# </pre>
# <s2i>

RUN apk --no-cache add git && rm -rf /var/cache/apk/*

ENV STI_SCRIPTS_PATH=/usr/libexec/s2i

LABEL io.openshift.expose-services="8080:http" \
    io.openshift.tags="builder,php" \
    io.openshift.s2i.scripts-url=image:///usr/libexec/s2i

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Directory with the sources is set as the working directory so all S2I scripts
# can execute relative to this path.
WORKDIR ${HOME}

EXPOSE 8080
CMD /usr/bin/base-usage

# </s2i>
# <template>

RUN set -xe; \
      apk add --no-cache --virtual .php-builddeps \
        zlib-dev \
        freetype-dev \
        jpeg-dev \
        libjpeg-turbo-dev \
        postgresql-dev \
        sqlite-dev \
        libltdl \
        libmcrypt \
        libmcrypt-dev \
        libpng \
        libpng-dev \
        icu \
        icu-dev \
        gettext-dev \
        libxml2-dev \
        freetype-dev \
        unzip \
        libc6-compat \
        gcc \
        autoconf && \
      docker-php-ext-configure gd --with-freetype-dir=/usr/include/  --with-jpeg-dir=/usr/include/ && \
      docker-php-ext-install \
        bcmath \
        ctype \
        dom \
        fileinfo \
        gd \
        gettext \
        intl \
        json \
        mbstring \
        mcrypt \
        mysqli \
        opcache \
        pcntl \
        pdo \
        pdo_sqlite \
        pdo_mysql \
        pdo_pgsql \
        phar \
        simplexml \
        soap \
        xml \
        zip && \
    	apk add --no-cache apache2-proxy sed gettext && \
      rm -rf /tmp/* && \
      rm -rf /var/cache/apk/*

RUN sed -i -f /opt/app-root/etc/httpdconf.sed /etc/apache2/httpd.conf && \
    echo "IncludeOptional /opt/app-root/etc/conf.d/*.conf" >> /etc/apache2/httpd.conf && \
    rm -f /usr/local/etc/php-fpm.d/zz-docker.conf && \
    mkdir /run/apache2 /tmp/sessions /run/php-fpm && \
    chown -R default:0 /opt/app-root /tmp/sessions /run/apache2 /run/php-fpm && \
    chmod -R ug+rwx /tmp/sessions /run/apache2 /run/php-fpm \
                   /usr/local/etc/php /usr/local/etc/php-fpm.d/

# </template>
# <post>

# Standardize on UID 100 (alpine:3.4 does not have the shadow package)
RUN apk add shadow --no-cache --repository http://rsync.alpinelinux.org/alpine/latest-stable/community/ && \
    usermod -o -u 100 default && \
    apk del shadow

# Drop the root user and make the content of /opt/app-root owned by user default
RUN chown -R 100:0 /opt/app-root && chmod -R ug+rwx /opt/app-root
USER 100

ENTRYPOINT ["container-entrypoint"]

# </post>