FROM php:8.1.1-fpm-buster
SHELL ["/bin/bash", "-oeux", "pipefail", "-c"]

ARG NEW_RELIC_AGENT_VERSION
ARG NEW_RELIC_LICENSE_KEY
ARG NEW_RELIC_APPNAME

ENV COMPOSER_ALLOW_SUPERUSER=1 \
  COMPOSER_HOME=/composer

COPY --from=composer:1.10 /usr/bin/composer /usr/bin/composer

RUN apt-get update && \
  apt-get -y install git unzip libzip-dev libicu-dev libonig-dev && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/* && \
  docker-php-ext-install intl pdo_mysql zip bcmath

RUN curl -L https://download.newrelic.com/php_agent/archive/${NEW_RELIC_AGENT_VERSION}/newrelic-php5-${NEW_RELIC_AGENT_VERSION}-linux.tar.gz | tar -C /tmp -zx \
    && export NR_INSTALL_USE_CP_NOT_LN=1 \
    && export NR_INSTALL_SILENT=1 \
    && /tmp/newrelic-php5-${NEW_RELIC_AGENT_VERSION}-linux/newrelic-install install \
    && rm -rf /tmp/newrelic-php5-* /tmp/nrinstall*

# ファイルを直接コピーして設置すると動かない問題が他でも報告されていた。
# 自分の環境でも再現したのでこの方法で設定ファイルを生成した。
RUN echo "newrelic.enabled = true" >> /usr/local/etc/php/conf.d/newrelic.ini \
    && echo "newrelic.license = ${NEW_RELIC_LICENSE_KEY}" >> /usr/local/etc/php/conf.d/newrelic.ini \
    && echo "newrelic.appname = ${NEW_RELIC_APPNAME}" >> /usr/local/etc/php/conf.d/newrelic.ini \
    && echo "newrelic.daemon.address = php-daemon:31339" >> /usr/local/etc/php/conf.d/newrelic.ini \
    && echo "newrelic.transaction_tracer.enabled = true" >> /usr/local/etc/php/conf.d/newrelic.ini \
    && echo "newrelic.distributed_tracing_enabled = true" >> /usr/local/etc/php/conf.d/newrelic.ini

COPY ./php.ini /usr/local/etc/php/php.ini

WORKDIR /work