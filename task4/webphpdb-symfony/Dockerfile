FROM webphpdb:0.2
#
RUN apt-get update \
&& apt-get install -y libicu-dev \
&& docker-php-ext-configure intl \
&& docker-php-ext-install intl 


