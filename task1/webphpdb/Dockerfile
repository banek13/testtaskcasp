FROM  php:7.2-apache

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r mysql && useradd -r -g mysql mysql

RUN apt-get update && apt-get install -y --no-install-recommends gnupg dirmngr dnsutils  && rm -rf /var/lib/apt/lists/*



# add gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& apt-get update && apt-get install -y --no-install-recommends ca-certificates wget && rm -rf /var/lib/apt/lists/* \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
        && mkdir ~/.gnupg && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
	&& export GNUPGHOME="$(mktemp -d)" \
#	&& for server in ha.pool.sks-keyservers.net \
#              hkp://p80.pool.sks-keyservers.net:80 \
#              p80.pool.sks-keyservers.net \
#              keyserver.ubuntu.com \
#              hkp://keyserver.ubuntu.com:80 \
#              keyserver.pgp.com \
#              pgp.mit.edu; do \
#           gpg --batch --keyserver "$server" --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 && break || echo "Trying new server..." ;\
#           done \
#	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
#	&& gpgconf --kill all \
#	&& rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
        && echo

RUN mkdir /docker-entrypoint-initdb.d

RUN apt-get update && apt-get install -y --no-install-recommends \
# for MYSQL_RANDOM_ROOT_PASSWORD
		pwgen \
# for mysql_ssl_rsa_setup
		openssl \
# FATAL ERROR: please install the following Perl modules before executing /usr/local/mysql/scripts/mysql_install_db:
# File::Basename
# File::Copy
# Sys::Hostname
# Data::Dumper
		perl \
                lsb-release \
	&& rm -rf /var/lib/apt/lists/*

# Disable due to gpg error
#RUN set -ex; \
## gpg: key 5072E1F5: public key "MySQL Release Engineering <mysql-build@oss.oracle.com>" imported
#	key='A4A9406876FCBD3C456770C88C718D3B5072E1F5'; \
#	export GNUPGHOME="$(mktemp -d)"; \
#	gpg --batch --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-keys "$key"; \
#	gpg --batch --export "$key" > /etc/apt/trusted.gpg.d/mysql.gpg; \
#	gpgconf --kill all; \
#	rm -rf "$GNUPGHOME"; \
#	apt-key list > /dev/null

ENV MYSQL_MAJOR 8.0
#ENV MYSQL_VERSION 8.0.16-1debian9
ENV MYSQL_VERSION 5.7

#RUN echo "deb http://repo.mysql.com/apt/debian/ stretch mysql-${MYSQL_MAJOR}" > /etc/apt/sources.list.d/mysql.list

RUN { \
    echo 'mysql-apt-config mysql-apt-config/repo-codename select stretch'; \
    echo 'mysql-apt-config mysql-apt-config/repo-distro select debian'; \
    echo 'mysql-apt-config mysql-apt-config/repo-url string http://repo.mysql.com/apt/'; \
    echo 'mysql-apt-config mysql-apt-config/select-preview select '; \
    echo 'mysql-apt-config mysql-apt-config/select-product select Ok'; \
    echo 'mysql-apt-config mysql-apt-config/select-server select mysql-5.7'; \
    echo 'mysql-apt-config mysql-apt-config/select-tools select '; \
    echo 'mysql-apt-config mysql-apt-config/unsupported-platform select abort'; \
    } | debconf-set-selections \
    && export DEBIAN_FRONTEND=noninteractive \
    && wget http://repo.mysql.com/mysql-apt-config_0.8.9-1_all.deb \
    && dpkg -i mysql-apt-config_0.8.9-1_all.deb \
    && rm -f  mysql-apt-config_0.8.9-1_all.deb \ 
    && apt-get purge -y --auto-remove ca-certificates wget 
    

# the "/var/lib/mysql" stuff here is because the mysql-server postinst doesn't have an explicit way to disable the mysql_install_db codepath besides having a database already "configured" (ie, stuff in /var/lib/mysql/mysql)
# also, we set debconf keys to make APT a little quieter
RUN { \
		echo mysql-community-server mysql-community-server/data-dir select ''; \
		echo mysql-community-server mysql-community-server/root-pass password ''; \
		echo mysql-community-server mysql-community-server/re-root-pass password ''; \
		echo mysql-community-server mysql-community-server/remove-test-db select false; \
	} | debconf-set-selections \
	#&& apt-get update && apt-get install -y mysql-community-client="${MYSQL_VERSION}" mysql-community-server-core="${MYSQL_VERSION}" && rm -rf /var/lib/apt/lists/* \
	&& apt-get update && apt-cache search mysql && apt-get install -y mysql-client mysql-server && rm -rf /var/lib/apt/lists/* \
	&& rm -rf /var/lib/mysql && mkdir -p /var/lib/mysql /var/run/mysqld \
	&& chown -R mysql:mysql /var/lib/mysql /var/run/mysqld \
# ensure that /var/run/mysqld (used for socket and lock files) is writable regardless of the UID our mysqld instance ends up having at runtime
	&& chmod 777 /var/run/mysqld

VOLUME /var/lib/mysql
# Config files
COPY config/ /etc/mysql/
COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
#ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306 33060
#CMD ["mysqld"]
COPY wrapper_script.sh /wrapper_script.sh
RUN chmod +x /wrapper_script.sh
#CMD /wrapper_script.sh
