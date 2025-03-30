### Base image ###
FROM ubuntu:22.04

ARG APACHE_DOCROOT="/var/www/html"
ARG USER="webdev"
ARG USER_GROUP="webdev"

# Environment
ENV HOME=/home/${USER}
ENV USER=${USER}
ENV USER_GROUP=${USER_GROUP}

# Install additional packages
RUN apt-get update \
    && apt-get install -y \
        zip \
        unzip \
        bash-completion \
        build-essential \
        ninja-build \
        clang \
        htop \
        iputils-ping \
        jq \
        less \
        locales \
        man-db \
        nano \
        ripgrep \
        software-properties-common \
        sudo \
        stow \
        time \
        emacs-nox \
        vim \
        multitail \
        lsof \
        ssl-cert \
        fish \
        zsh \
        rlwrap \
        wget \
        curl \
        python3-launchpadlib \
        w3m \
    && locale-gen en_US.UTF-8

COPY install-packages upgrade-packages /usr/bin/
RUN chmod +x /usr/bin/install-packages /usr/bin/upgrade-packages

# Create the group + user
# '-l': see https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user
RUN useradd -l -G sudo -md /home/${USER} -s /bin/bash -p ${USER} ${USER} \
    # Remove `use_pty` option and enable passwordless sudo for users in the 'sudo' group
    && sed -i.bkp -e '/Defaults\tuse_pty/d' -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers

# use sudo so that user does not get sudo usage info on (the first) login
RUN sudo echo "Running 'sudo' for user: success" && \
    # create .bashrc.d folder and source it in the bashrc
    mkdir -p /home/${USER}/.bashrc.d && \
    (echo; echo "for i in \$(ls -A \$HOME/.bashrc.d/); do source \$HOME/.bashrc.d/\$i; done"; echo) >> /home/${USER}/.bashrc && \
    # create a completions dir for gitpod user
    mkdir -p /home/${USER}/.local/share/bash-completion/completions

# Install everything related to php and apache2
# Based on https://github.com/gitpod-io/workspace-images/tree/main/chunks/tool-nginx
# Check https://launchpad.net/~ondrej for more php extensions
RUN for _ppa in 'ppa:ondrej/php' 'ppa:ondrej/apache2'; do add-apt-repository -y "$_ppa"; done \
    && install-packages \
        apache2 \
        php \
        php-all-dev \
        php-bcmath \
        php-common \
        php-curl \
        php-date \
        php-fpm \
        php-gd \
        php-intl \
        php-json \
        php-mbstring \
        php-mysql \
        php-net-ftp \
        php-pear \
        php-redis \
        php-sodium \
        php-sqlite3 \
        php-tokenizer \
        php-xdebug \
        php-xml \
        php-zip \
    && ln -s /etc/apache2/mods-available/rewrite.load /etc/apache2/mods-enabled/rewrite.load \
    && chown -R ${USER}:${USER_GROUP} /etc/apache2 /var/run/apache2 /var/lock/apache2 /var/log/apache2 /var/www/html

COPY ./apache2/httpd.conf /etc/apache2/apache2.conf
COPY ./apache2/envvars /etc/apache2/envvars

# Enable apache modules
RUN a2enmod headers \
    && a2enmod rewrite \
    && a2enmod proxy \
    && a2enmod proxy_fcgi \
    && a2enmod proxy_http \
    && a2enmod ssl

# Expose port 8080
EXPOSE 8080

# Clean up
RUN apt-get clean \
    && apt-get autoremove -y \
    && rm -Rf /var/lib/apt/lists/* /usr/share/man/* /usr/share/doc/*

USER ${USER}