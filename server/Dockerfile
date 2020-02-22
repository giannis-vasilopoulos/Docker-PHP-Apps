FROM ubuntu:18.04

# Get user id and group id from arguments
ARG UID
ARG GID
# Get root password.
ARG SERVER_ROOT_PASSWORD

RUN \
	apt-get update \
	&& apt-get install -y tzdata \
	&& apt-get install -y language-pack-en-base software-properties-common \
	&& add-apt-repository ppa:ondrej/php \
	&& apt-get update \
	&& apt-get install -y \
		apache2 \
		curl \
		git unzip ca-certificates wget supervisor openssh-server \
		less \
		libapache2-mod-php7.4 \
		libsodium23 \
		mysql-client \
		nano \
		php-apcu \
		php-xdebug \
		php7.4 \
		php7.4-bcmath \
		php7.4-cli \
		php7.4-curl \
		php7.4-gd \
		php7.4-imagick \
		php7.4-json \
		php7.4-ldap \
		php7.4-mbstring \
		php7.4-mysql \
		php7.4-opcache \
		php7.4-pgsql \
		php7.4-soap \
		php7.4-sqlite3 \
		php7.4-xml \
		php7.4-xsl \
		php7.4-fpm \
		php7.4-zip \
		php-xdebug\
		ssmtp \
		subversion \
		sudo \
	&& rm -rf /var/lib/apt/lists/*


# Configuration.
ADD xdebug /tmp/xdebug
RUN cat /tmp/xdebug >> /etc/php/7.4/apache2/conf.d/20-xdebug.ini && rm /tmp/xdebug

# Remove package lists.
RUN apt-get clean && rm -rf /var/cache/apt/* && rm -rf /var/lib/apt/lists/*

# Enable apache modules.
RUN a2enmod rewrite

# Install wp-cli
RUN curl -o /usr/local/bin/wp -SL https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
	&& chmod +x /usr/local/bin/wp

# Install PsySH to use in wp-cli shell
RUN curl https://psysh.org/psysh -L -o /usr/local/bin/psysh \
	&& chmod +x /usr/local/bin/psysh

# Set server name.
RUN echo 'ServerName localhost' >> /etc/apache2/apache2.conf

# Create user, vhosts, permissions.
# Change user apache runs.
RUN sed -i -e"s/^export\s*APACHE_RUN_USER=www-data/export APACHE_RUN_USER=serveruser/" /etc/apache2/envvars && \
  sed -i -e"s/^export\s*APACHE_RUN_GROUP=www-data/export APACHE_RUN_GROUP=serveruser/" /etc/apache2/envvars

# Create user.
RUN groupadd -r serveruser -g $GID && useradd -ms /bin/bash serveruser -u $UID -g $GID

# Install composer.
RUN cd ~ && curl -sS https://getcomposer.org/installer -o composer-setup.php && \
  php composer-setup.php --install-dir=/usr/local/bin --filename=composer && \
  rm -rf composer-setup.php && \
  echo "PATH=$PATH:~/.composer/vendor/bin" >> ~/.bashrc

# Install phpunit.
RUN wget https://phar.phpunit.de/phpunit.phar && \
  chmod +x phpunit.phar && \
  mv phpunit.phar /usr/local/bin/phpunit

# Add vhosts and enable sites.
COPY vhosts /etc/apache2/sites-available
RUN cd /etc/apache2/sites-available && a2ensite * && a2dissite default-ssl 000-default

# Setup ssh.
RUN mkdir /var/run/sshd && \
  echo 'root:${SERVER_ROOT_PASSWORD}' | chpasswd && \
  sed -i 's/prohibit-password/yes/' /etc/ssh/sshd_config

# Setup supervisor to run child processes.
ADD ./supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
ADD ./supervisor/apache.conf /etc/supervisor/conf.d/apache.conf
ADD ./supervisor/ssh.conf /etc/supervisor/conf.d/ssh.conf
RUN chown root:root /etc/supervisor/conf.d/* && chmod 644 /etc/supervisor/conf.d/*

EXPOSE 80 22

CMD ["/usr/bin/supervisord", "-n"]
