FROM ubuntu:15.10

MAINTAINER Christian Wiedemann <christian.wiedemann@key-tec.de>

# Set timezone and locale.
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8



ENV TERM xterm

RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -y install software-properties-common

RUN DEBIAN_FRONTEND=noninteractive add-apt-repository ppa:ondrej/php && \
    DEBIAN_FRONTEND=noninteractive apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -y install \
    php7.0-common \
    php7.0-cli \
    php7.0-mysql \
    php7.0-mcrypt \
    php7.0-curl \
    php7.0-mbstring \
    php-xdebug \
    php7.0-gd \
    php7.0-sqlite \
    php7.0-json \
    php7.0-intl \
    php7.0-xml \
    php7.0-fpm
    # Cleanup
# Basic packages
RUN DEBIAN_FRONTEND=noninteractive apt-get -y --force-yes --no-install-recommends install \
    zip unzip \
    git \
    curl \
    mysql-client \
    pv \
    openssh-client \
    rsync \
    apt-transport-https \
    sudo \
    locales \
    supervisor \
    procps \
     mc \
    # Cleanup
    && DEBIAN_FRONTEND=noninteractive apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN \
    # Create a non-root user with access to sudo and the default group set to 'users' (gid = 100)
    useradd -m -s /bin/bash -g users -G sudo -p docker docker && \
    echo 'docker ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers



# PHP packages

## PHP settings
RUN mkdir -p /var/www/docroot && \
    # PHP-FPM settings
    ## /etc/php/7.0/fpm/php.ini
    sed -i '/memory_limit = /c memory_limit = 256M' /etc/php/7.0/fpm/php.ini && \
    sed -i '/max_execution_time = /c max_execution_time = 300' /etc/php/7.0/fpm/php.ini && \
    sed -i '/upload_max_filesize = /c upload_max_filesize = 500M' /etc/php/7.0/fpm/php.ini && \
    sed -i '/post_max_size = /c post_max_size = 500M' /etc/php/7.0/fpm/php.ini && \
    sed -i '/error_log = /c error_log = \/dev\/stdout' /etc/php/7.0/fpm/php.ini && \
    sed -i '/;always_populate_raw_post_data/c always_populate_raw_post_data = -1' /etc/php/7.0/fpm/php.ini && \
    sed -i '/;sendmail_path/c sendmail_path = /bin/true' /etc/php/7.0/fpm/php.ini && \
    ## /etc/php/7.0/fpm/pool.d/www.conf
    sed -i '/user = /c user = docker' /etc/php/7.0/fpm/pool.d/www.conf && \
    sed -i '/;catch_workers_output = /c catch_workers_output = yes' /etc/php/7.0/fpm/pool.d/www.conf && \
    sed -i '/listen = /c listen = 0.0.0.0:9000' /etc/php/7.0/fpm/pool.d/www.conf && \
    sed -i '/listen.allowed_clients/c ;listen.allowed_clients =' /etc/php/7.0/fpm/pool.d/www.conf && \
    sed -i '/;clear_env = /c clear_env = no' /etc/php/7.0/fpm/pool.d/www.conf && \
    ## /etc/php/7.0/fpm/php-fpm.conf
    sed -i '/;daemonize = /c daemonize = no' /etc/php/7.0/fpm/php-fpm.conf && \
    sed -i '/error_log = /c error_log = \/dev\/stdout' /etc/php/7.0/fpm/php-fpm.conf && \
    # PHP CLI settings
    sed -i '/memory_limit = /c memory_limit = 512M' /etc/php/7.0/cli/php.ini && \
    sed -i '/max_execution_time = /c max_execution_time = 600' /etc/php/7.0/cli/php.ini && \
    sed -i '/error_log = php_errors.log/c error_log = \/dev\/stdout' /etc/php/7.0/cli/php.ini && \
    sed -i '/;sendmail_path/c sendmail_path = /bin/true' /etc/php/7.0/cli/php.ini && \
    # PHP module settings
    echo 'opcache.memory_consumption=128' >> /etc/php/7.0/mods-available/opcache.ini

#COPY config/php5/xdebug.ini /etc/php/7.0/mods-available/xdebug.ini

# Adding NodeJS repo (for up-to-date versions)
# This is a stripped down version of the official nodejs install script (https://deb.nodesource.com/setup_4.x)
RUN curl -sSL https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - && \
    echo 'deb https://deb.nodesource.com/node_4.x jessie main' > /etc/apt/sources.list.d/nodesource.list && \
    echo 'deb-src https://deb.nodesource.com/node_4.x jessie main' >> /etc/apt/sources.list.d/nodesource.list

# Other language packages and dependencies
RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -y --force-yes --no-install-recommends install \
    ruby-full \
    rlwrap \
    build-essential \
    # Cleanup
    && DEBIAN_FRONTEND=noninteractive apt-get clean &&\
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# bundler
RUN gem install bundler
# Home directory for bundle installs
ENV BUNDLE_PATH .bundler

ENV DRUSH_VERSION 8.0.1
ENV DRUPAL_CONSOLE_VERSION 0.10.1
RUN \
    # Composer
    curl -sSL https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer && \
    # Drush 8 (default)
    curl -sSL https://github.com/drush-ops/drush/releases/download/$DRUSH_VERSION/drush.phar -o /usr/local/bin/drush && \
    chmod +x /usr/local/bin/drush && \
    # Drupal Console
    curl -sSL https://github.com/hechoendrupal/DrupalConsole/releases/download/$DRUPAL_CONSOLE_VERSION/drupal.phar -o /usr/local/bin/drupal && \
    chmod +x /usr/local/bin/drupal
ENV PATH /home/docker/.composer/vendor/bin:$PATH

# Install gosu and give access to the users group to use it. gosu will be used to run services as a different user.
RUN curl -sSL "https://github.com/tianon/gosu/releases/download/1.7/gosu-amd64" -o /usr/local/bin/gosu && \
    chown root:users /usr/local/bin/gosu && \
    chmod +sx /usr/local/bin/gosu

# Set TERM so text editors/etc. can be used
ENV TERM xterm


# All further RUN commands will run as the "docker" user
USER docker

# Install nvm and a default node version
ENV NVM_VERSION 0.30.1
ENV NODE_VERSION 0.12.2
ENV NVM_DIR /home/docker/.nvm
RUN \
    curl -sSL https://raw.githubusercontent.com/creationix/nvm/v${NVM_VERSION}/install.sh | bash && \
    . $NVM_DIR/nvm.sh && \
    nvm install $NODE_VERSION && \
    nvm alias default $NODE_VERSION && \
    # Install global node packages
    npm install -g npm && \
    npm install -g bower && \
    npm install -g grunt-cli

RUN \
    # Legacy Drush versions (6 and 7)
    echo "alias drush8='/usr/local/bin/drush'" >> /home/docker/.bashrc && \
    # Drush modules
    drush dl registry_rebuild-7.x-2.2 && \
    drush dl coder --destination=/home/docker/.drush && \
    drush cc drush && \
    # Drupal Coder w/ a matching version of PHP_CodeSniffer
    composer global require drupal/coder && \
    phpcs --config-set installed_paths /home/docker/.composer/vendor/drupal/coder/coder_sniffer





# Copy configs and scripts
COPY config/.ssh /home/docker/.ssh
COPY config/.drush /home/docker/.drush
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY startup.sh /opt/startup.sh

# Fix permissions after COPY
RUN sudo chown -R docker:users /home/docker

EXPOSE 9000

WORKDIR /var/www

# Default SSH key name
ENV SSH_KEY_NAME id_rsa

# Starter script
ENTRYPOINT ["/opt/startup.sh"]

# By default, launch supervisord to keep the container running.
CMD ["gosu", "root", "supervisord"]
