<section id="setting-php" style="padding: 10px;">
<h2>Setting <code>php.dockerfile</code></h2>
<p>Its time to we build our PHP container with xdebug, composer and configs inside a Dockerfile. Create the file inside follow path: <code>/environmentProject/php.dockerfile</code></p>
<p>Inside the file we'll set our PHP 8.0 FPM version. To use it we need a tag that we can find on dockerhub. With tag in hands, our first line should be:</p>
<pre>
FROM php:8.0-apache
</pre>
<blockquote>
Versions tag list <a href="https://hub.docker.com/_/php?tab=tags&page=1&ordering=last_updated">here</a>.
</blockquote>
<br>
<h3>Core Extensions:</h3>
<p>Following the version we need to install our core extensions: </p>
<pre>
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install mysqli pdo pdo_mysql && docker-php-ext-enable pdo_mysql
</pre>
<blockquote>
Some extensions come by default. It depends on the PHP version you are using. Run <code>docker run --rm php:8.0-fpm php -m</code> in the your terminar to see full extension list.
</blockquote>
<br>
<h3><a href="https://pecl.php.net/">PECL</a> extensions:</h3>
<p>Its time to bring some extensions for performance and utility, to it we'll use PECL, a PHP extension repository.<br>
Here we'll install <code>xdebug</code>, <code>libmencached</code> and <code>zlib1g</code>. Lets add some more lines in our dockerfile:</p>
<code>docker-compose up -d</code>
<pre>
RUN pecl install xdebug-3.0.4 \
    && docker-php-ext-enable xdebug
RUN apt-get install -y libmemcached-dev zlib1g-dev \
    && pecl install memcached-3.1.5 \
    && docker-php-ext-enable memcached
</pre>
<blockquote>
 PECL extensions should be installed in series to fail properly if something went wrong. Otherwise errors are just skipped by PECL.
</blockquote>
<br>
<h3>Packages:</h3>
<p>For packages we'll use composer with zip and unzip:</p>
<pre>
RUN apt-get install zip unzip \
    && curl -sS https://getcomposer.org/installer -o composer-setup.php \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && unlink composer-setup.php
</pre>
<h3>Configs (utils):</h3>
<p>Setting our timezone and activating opcache for performance of PHP:</p>
<pre>
RUN echo 'date.timezone="America/Sao_Paulo"' >> /usr/local/etc/php/conf.d/date.ini \
    && echo 'opcache.enable=1' >> /usr/local/etc/php/conf.d/opcache.conf \
    && echo 'opcache.validate_timestamps=1' >> /usr/local/etc/php/conf.d/opcache.conf \
    && echo 'opcache.fast_shutdown=1' >> /usr/local/etc/php/conf.d/opcache
</pre>
<blockquote>
You can check the list of timezone supported by PHP <a href="https://www.php.net/manual/en/timezones.america.php">here</a>.
</blockquote>
</section>
