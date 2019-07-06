# php-swoole original source: https://www.swoole.co.uk/article/docker.html
PHP + Swoole (async socket)

This article introduce how to get started with Swoole without manually compile the Swoole PHP extension.

Docker make it easy for us to try Swoole with only several commands.

Make sure you have installed Docker on your machine and updated the swoole version to be the latest version.

Folder structure:
/Dockerfile
/app/index.php

Content of index.php
<?php
  $http = new swoole_http_server("0.0.0.0", 8101);

  $http->on("start", function ($server) {
      echo "Swoole http server is started at http://127.0.0.1:8101\n";
  });

  $http->on("request", function ($request, $response) {
      $response->header("Content-Type", "text/plain");
      $response->end("Hello World\n");
  });

  $http->start();
?>


Content of Swoole Dockerfile:
FROM php:7.2-fpm

RUN apt-get update

RUN apt-get install vim -y && \
    apt-get install openssl -y && \
    apt-get install libssl-dev -y && \
    apt-get install wget -y 

RUN cd /tmp && wget https://pecl.php.net/get/swoole-4.2.9.tgz && \
    tar zxvf swoole-4.2.9.tgz && \
    cd swoole-4.2.9  && \
    phpize  && \
    ./configure  --enable-openssl && \
    make && make install

RUN touch /usr/local/etc/php/conf.d/swoole.ini && \
    echo 'extension=swoole.so' > /usr/local/etc/php/conf.d/swoole.ini

RUN mkdir -p /app/data

WORKDIR /app

COPY ./app /app

EXPOSE 8101
CMD ["/usr/local/bin/php", "/app/index.php"]

Compile the Dockerfile:
It may take several minutes to compile the Dockerfile including PHP Swoole.

docker build -f ./Dockerfile -t hello-swoole .

Execute your application with Docker:
docker run -p 8101:8101 hello-swoole
Open your browser and open http://127.0.0.1:8101, you will be able to see the hello world response from Swoole server.

Remeber to kill the docker container with docker ps and docker kill once you have done the test.
