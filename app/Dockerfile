# Docker - Drupal+CiviCRM
#
# Use official Drupal7+FPM as base image for build
FROM drupal:9.4.2-fpm

# VERSION 0.0.1
MAINTAINER William Hale <salt@snowdrift.coop>

# Disable PHP daemon mode
RUN sed -i -e "s/;daemonize\s*=\s*yes/daemonize = no/g" /usr/local/etc/php-fpm.conf

# Install Packages
RUN DEBIAN_FRONTEND="noninteractive" apt-get update \
  && apt-get install -y apt-utils \
  && apt-get install -y \
    cron \
    libc-client-dev \
    libkrb5-dev \
    libicu-dev \
    libmcrypt-dev \
    libxml2-dev \
    mariadb-client \
    ssmtp
    
COPY docker-php-ext-get /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-php-ext-get

# Install additional PHP extentions
RUN docker-php-source extract \
  && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
  && docker-php-ext-install imap \
  && docker-php-ext-install intl \
  && docker-php-ext-install mysqli \
  && docker-php-ext-install soap \
  && docker-php-ext-get mcrypt 1.0.5 \
  && docker-php-ext-install mcrypt

# Install ssmtp
COPY ssmtp.conf /etc/ssmtp/ssmtp.conf
RUN chown root:mail /etc/ssmtp/ssmtp.conf \
  && chmod 640 /etc/ssmtp/ssmtp.conf \
  && usermod -a -G mail www-data \
  && echo 'sendmail_path = "/usr/sbin/ssmtp -t"' > /usr/local/etc/php/conf.d/mail.ini

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php \
  && mv composer.phar /usr/local/bin/composer

# Install Drush
RUN curl -fSL hhttps://github.com/drush-ops/drush/releases/download/8.4.11/drush.phar -o /usr/local/bin/drush \
  && chmod +x /usr/local/bin/drush \
  && curl -fSL https://raw.githubusercontent.com/civicrm/civicrm-drupal/7.x-master/drush/civicrm.drush.inc -o /usr/share/drush/commands/civicrm.drush.inc --create-dirs
# COPY civicrm.drush.inc /usr/share/drush/commands/civicrm.drush.inc

# Download CiviCRM
ENV CIVICRM_VERSION 5.51.0
RUN curl -fSL "https://download.civicrm.org/civicrm-${CIVICRM_VERSION}-drupal.tar.gz" -o /var/www/civicrm.tar.gz

# Install default error pages
COPY 404.html /var/www/html/404.html
COPY 50x.html /var/www/html/50x.html

# Install maintenance scripts
COPY civi_install.sh /usr/local/bin/civi_install.sh
COPY civi_run.sh /usr/local/bin/civi_run.sh
COPY civi_upgrade.sh /usr/local/bin/civi_upgrade.sh
RUN chmod +x /usr/local/bin/civi_install.sh \
  && chmod +x /usr/local/bin/civi_run.sh \
  && chmod +x /usr/local/bin/civi_upgrade.sh

# Enable Cron
COPY civi_cron.sh /usr/local/bin/civi_cron.sh
COPY crontab /etc/cron.d/crontab
RUN chmod +x /usr/local/bin/civi_cron.sh \
  && chmod +x /etc/cron.d/crontab \
  && crontab /etc/cron.d/crontab

# Cleanup
RUN apt-get clean \
  && rm -rf \
    /var/lib/apt/lists/* \
    /tmp/* \
    /var/tmp/*

LABEL Description="Docker for Snowdrift.coop CRM. Debian Jessie+nginx+MariaDB+PHP5.6+Drupal7/Drush+CiviCRM" \
  Version="0.0.1"

CMD ["/usr/local/bin/civi_run.sh"]
