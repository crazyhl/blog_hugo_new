---
title: "Laravel in Kubernetes 系列文章个人总结（1）"
date: 2023-03-07T11:16:24+08:00
categories: [php, 运维]
slug: personal-summary-of-laravel-in-kubernetes-series-of-articles-1
tags: [laravel, kubernetes]
---

## 前言
前一段时间，自己实现了一个项目后，准备部署，自然还是要搞 k8s 啦。但是我觉得以前的用 golang 的封装就比较简单粗暴了，编译一层，然后再把编译后的输出一层就好了。然后前段也单独配置就很完美。但是 php 这里没搞前后端分离，且我觉得可能还需要再了解一下 docker 镜像编译的好套路，所以就搜索了一下，结果还真的让我发现了一个好东西 [https://chris-vermeulen.com/laravel-in-kubernetes/](https://chris-vermeulen.com/laravel-in-kubernetes/)。写的非常细致，且方便观看，每篇都很短，目的明确清晰。这个在写作上也给了我很多的指导。我以后在写博客的时候也要模仿这种操作。好了，下面我就会把我觉得我需要记录的东西，记录下来，方便后面再次使用到这部分时候回忆。

## 正文

### 修改 laravel 日子输出为 stdout
在 config/logging.php，添加新的日志输出通道，输出到 `stdout`。原因是可以通过 `kubectl logs` 查看到日志。
```php
return [
    'channels' => [
        'stdout' => [
            'driver' => 'monolog',
            'level' => env('LOG_LEVEL', 'debug'),
            'handler' => StreamHandler::class,
            'formatter' => env('LOG_STDOUT_FORMATTER'),
            'with' => [
                'stream' => 'php://stdout',
            ],
        ],
    ],
],
```

在 `.env` 中修改 日志 输出位置

```php
LOG_CHANNEL=stdout
```
这里我个人理解就是方便我们查看日志，不过话说回来，在真的大型商业服务中，谁会把日志放到 `stdout` 呢，肯定是投递到某个日志分析组件中了。这里就当学习设置日志输出通道把，也有可能日志组件在读取 `kubectl logs`？不是很了解。先按照我自己的理解来。

### Session 驱动调整
因为可能会开启多个 pods，所以 `Session` 自然不能用传统的文件方式存储，否则落到了，其他 pods 上会丢失登录信息，除非配置落到指定的 pods 上，但是实际上是不可能的。毕竟这玩意会‘频繁’ 的杀死开启，落到指定的服务器是可能呢，pods 万万不能。所以要把 `Session` 移动到一个合理的地方，那么 `Redis` 自然就是优选了。  
```bash
composer require predis/predis
```
安装 redis 扩展。使 php 能够使用 `redis`。

编辑 `.env` 文件调整 `Session` 驱动
```php
SESSION_DRIVER=redis
```

### 强制返回 HTTPS
编辑 `AppServiceProvider`
```php
<?php

namespace App\Providers;

# Add the Facade
use Illuminate\Support\Facades\URL;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /** All the rest */

    public function boot()
    {
        if($this->app->environment('production')) {
            URL::forceScheme('https');
        }
    }
}
```

这个其实就是为了解决证书卸载导致返回生成的 `url` 是 http。这个不仅是 k8s 会用到，在很多场景都会用到

### 镜像分层规划
![](https://www.cimple.ink/static/images/202303071438473.png)
不得不说，人家弄的这个就清晰。不过我自己的部分，没有弄的这么全面，所以我更需要把这个分层保存下来，方便后续，用到的时候来回顾。

### 添加 .dockerignore 文件
添加这个文件后，复制文件的时候就会略过描述的文件，同 `gitignore` 一样，方便管理
```txt
/vendor
/node_modules
```
这两个大文件，不会被需要的。

### 创建 docker 文件
```bash
# Create args for PHP extensions and PECL packages we need to install.
# This makes it easier if we want to install packages,
# as we have to install them in multiple places.
# This helps keep ou Dockerfiles DRY -> https://bit.ly/dry-code
# You can see a list of required extensions for Laravel here: https://laravel.com/docs/8.x/deployment#server-requirements
ARG PHP_EXTS="bcmath ctype fileinfo mbstring pdo pdo_mysql tokenizer dom pcntl"
ARG PHP_PECL_EXTS="redis"
```
这里示例里面增加了两个参数，实际上新版的 php 镜像中已经包含了需要的扩展了，就没必要定义这两个扩展参数了。但是还是记录下来，万一以后能用到呢。
#### composer 层
```bash
# We need to build the Composer base to reuse packages we've installed
FROM composer:2.1 as composer_base

# We need to declare that we want to use the args in this build step
ARG PHP_EXTS
ARG PHP_PECL_EXTS

# First, create the application directory, and some auxilary directories for scripts and such
RUN mkdir -p /opt/apps/laravel-in-kubernetes /opt/apps/laravel-in-kubernetes/bin

# Next, set our working directory
WORKDIR /opt/apps/laravel-in-kubernetes

# We need to create a composer group and user, and create a home directory for it, so we keep the rest of our image safe,
# And not accidentally run malicious scripts
RUN addgroup -S composer \
    && adduser -S composer -G composer \
    && chown -R composer /opt/apps/laravel-in-kubernetes \
    && apk add --virtual build-dependencies --no-cache ${PHPIZE_DEPS} openssl ca-certificates libxml2-dev oniguruma-dev \
    && docker-php-ext-install -j$(nproc) ${PHP_EXTS} \
    && pecl install ${PHP_PECL_EXTS} \
    && docker-php-ext-enable ${PHP_PECL_EXTS} \
    && apk del build-dependencies

# Next we want to switch over to the composer user before running installs.
# This is very important, so any extra scripts that composer wants to run,
# don't have access to the root filesystem.
# This especially important when installing packages from unverified sources.
USER composer

# Copy in our dependency files.
# We want to leave the rest of the code base out for now,
# so Docker can build a cache of this layer,
# and only rebuild when the dependencies of our application changes.
COPY --chown=composer composer.json composer.lock ./

# Install all the dependencies without running any installation scripts.
# We skip scripts as the code base hasn't been copied in yet and script will likely fail,
# as `php artisan` available yet.
# This also helps us to cache previous runs and layers.
# As long as comoser.json and composer.lock doesn't change the install will be cached.
RUN composer install --no-dev --no-scripts --no-autoloader --prefer-dist

# Copy in our actual source code so we can run the installation scripts we need
# At this point all the PHP packages have been installed, 
# and all that is left to do, is to run any installation scripts which depends on the code base
COPY --chown=composer . .

# Now that the code base and packages are all available,
# we can run the install again, and let it run any install scripts.
RUN composer install --no-dev --prefer-dist
```
注意版本号，及时使用最新版，当然配置也要跟着改进，比如 `--prefer-dist` 在新版本就没有了，而是采用 `--prefer-install=source/dist` 来弄，默认就是 dist，所以这个参数可以不用，写上的原因是更直观罢了。再比如在上面定义的 `PHPIZE_DEPS` 通篇文章都没有，那么这个参数是哪里来的呢，可以追溯 composer 镜像，再到 php 镜像，可以找到 `PHPIZE_DEPS` 的定义，这些都是我们在阅读源码的时候需要思考以及查询的。`PHPIZE_DEPS` 是使用 `ENV` 定义的，而我们在上面用了 `ARG` 这两个的区别看这里 [https://yeasy.gitbook.io/docker_practice/image/dockerfile/arg](https://yeasy.gitbook.io/docker_practice/image/dockerfile/arg)。
```php
# We need to declare that we want to use the args in this build step
ARG PHP_EXTS
ARG PHP_PECL_EXTS
```
注意这段不要直接用，要像最上面那样，写全才可以。

关于上面 composer 执行两次的问题，我思考了一下，主要是为了缓存，如果我们切换了顺序，可能就要每次都下载镜像了，浪费时间以及流量。不过这个在线上构建的时候是否有效不知道（比如 git action，或者 docker hub，我不知道他每次构建的时候是否会留有缓存，应该是没有的，每次都启动一个沙箱。），本地构建的时候确实是有缓存的。

测试构建 comopser 阶段 `docker build . --target composer_base` 不报错就好，报错了，就继续排查。

#### 前端资源编译阶段
```bash
# For the frontend, we want to get all the Laravel files,
# and run a production compile
FROM node:14 as frontend

# We need to copy in the Laravel files to make everything is available to our frontend compilation
COPY --from=composer_base /opt/apps/laravel-in-kubernetes /opt/apps/laravel-in-kubernetes

WORKDIR /opt/apps/laravel-in-kubernetes

# We want to install all the NPM packages,
# and compile the MIX bundle for production
RUN npm install && \
    npm run prod
```

这层就很简单了，从上层复制文件，然后进行编译保存，不过本次我自己的项目，没有这一步，就跳过了，等有需要的时候在来测试

#### cli 阶段
```bash
# For running things like migrations, and queue jobs,
# we need a CLI container.
# It contains all the Composer packages,
# and just the basic CLI "stuff" in order for us to run commands,
# be that queues, migrations, tinker etc.
FROM php:8.0-alpine as cli

# We need to declare that we want to use the args in this build step
ARG PHP_EXTS
ARG PHP_PECL_EXTS

WORKDIR /opt/apps/laravel-in-kubernetes

# We need to install some requirements into our image,
# used to compile our PHP extensions, as well as install all the extensions themselves.
# You can see a list of required extensions for Laravel here: https://laravel.com/docs/8.x/deployment#server-requirements
RUN apk add --virtual build-dependencies --no-cache ${PHPIZE_DEPS} openssl ca-certificates libxml2-dev oniguruma-dev && \
    docker-php-ext-install -j$(nproc) ${PHP_EXTS} && \
    pecl install ${PHP_PECL_EXTS} && \
    docker-php-ext-enable ${PHP_PECL_EXTS} && \
    apk del build-dependencies

# Next we have to copy in our code base from our initial build which we installed in the previous stage
COPY --from=composer_base /opt/apps/laravel-in-kubernetes /opt/apps/laravel-in-kubernetes
COPY --from=frontend /opt/apps/laravel-in-kubernetes/public /opt/apps/laravel-in-kubernetes/public
```
这个阶段主要是执行一些命令或者队列的，不过队列为什么不在 queue 阶段呢？
这个也是从不同阶段复制过来资源就好了。其实后面都是复制资源，我就不多写了。

#### fpm 阶段
```bash
# We need a stage which contains FPM to actually run and process requests to our PHP application.
FROM php:8.0-fpm-alpine as fpm_server

# We need to declare that we want to use the args in this build step
ARG PHP_EXTS
ARG PHP_PECL_EXTS

WORKDIR /opt/apps/laravel-in-kubernetes

RUN apk add --virtual build-dependencies --no-cache ${PHPIZE_DEPS} openssl ca-certificates libxml2-dev oniguruma-dev && \
    docker-php-ext-install -j$(nproc) ${PHP_EXTS} && \
    pecl install ${PHP_PECL_EXTS} && \
    docker-php-ext-enable ${PHP_PECL_EXTS} && \
    apk del build-dependencies
    
# As FPM uses the www-data user when running our application,
# we need to make sure that we also use that user when starting up,
# so our user "owns" the application when running
USER  www-data

# We have to copy in our code base from our initial build which we installed in the previous stage
COPY --from=composer_base --chown=www-data /opt/apps/laravel-in-kubernetes /opt/apps/laravel-in-kubernetes
COPY --from=frontend --chown=www-data /opt/apps/laravel-in-kubernetes/public /opt/apps/laravel-in-kubernetes/public

# We want to cache the event, routes, and views so we don't try to write them when we are in Kubernetes.
# Docker builds should be as immutable as possible, and this removes a lot of the writing of the live application.
RUN php artisan event:cache && \
    php artisan route:cache && \
    php artisan view:cache
```
这一个阶段也是复制资源以及执行一些 `artisan` 在生产阶段的优化命令。

#### web服务阶段
这个创建了一个 `docker` 文件夹，然后写入一个 `nginx.conf.template` 文件。这里面保存了 `nginx` 的配置，这里面就要根据实际需要进行编写了，下面这个就是个示例，都没有相关文件的缓存，在自己需要的时候得配置上
```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # We need to set the root for our sevrer,
    # so any static file requests gets loaded from the correct path
    root /opt/apps/laravel-in-kubernetes/public;

    index index.php index.html index.htm index.nginx-debian.html;

    # _ makes sure that nginx does not try to map requests to a specific hostname
    # This allows us to specify the urls to our application as infrastructure changes,
    # without needing to change the application
    server_name _;

    # At the root location,
    # we first check if there are any static files at the location, and serve those,
    # If not, we check whether there is an indexable folder which can be served,
    # Otherwise we forward the request to the PHP server
    location / {
        # Using try_files here is quite important as a security concideration
        # to prevent injecting PHP code as static assets,
        # and then executing them via a URL.
        # See https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/#passing-uncontrolled-requests-to-php
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Some static assets are loaded on every page load,
    # and logging these turns into a lot of useless logs.
    # If you would prefer to see these requests for catching 404's etc.
    # Feel free to remove them
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    # When a 404 is returned, we want to display our applications 404 page,
    # so we redirect it to index.php to load the correct page
    error_page 404 /index.php;

    # Whenever we receive a PHP url, or our root location block gets to serving through fpm,
    # we want to pass the request to FPM for processing
    location ~ \.php$ {
        #NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
        include fastcgi_params;
        fastcgi_intercept_errors on;
        fastcgi_pass ${FPM_HOST};
        fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
    }

    location ~ /\.ht {
        deny all;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

```bash
# We need an nginx container which can pass requests to our FPM container,
# as well as serve any static content.
FROM nginx:1.20-alpine as web_server

WORKDIR /opt/apps/laravel-in-kubernetes

# We need to add our NGINX template to the container for startup,
# and configuration.
COPY docker/nginx.conf.template /etc/nginx/templates/default.conf.template

# Copy in ONLY the public directory of our project.
# This is where all the static assets will live, which nginx will serve for us.
COPY --from=frontend /opt/apps/laravel-in-kubernetes/public /opt/apps/laravel-in-kubernetes/public
```
dockerfile 部分就是复制了 nginx 配置文件以及复制 public 文件，这里需要注意的是，由于我使用了 `Livewire` 有一部分文件不在 public 下，所以我复制了上层目录，这个可能不安全，所以我还得想想怎么弄，是不是可以把 livewire 的文件通过脚本复制出来更好一些？

注意，上面的nginx 配置文件中的 `fastcgi_pass` 使用了变量，这里是个特性，docker 的 nginx 1.19 版本后支持了这个特性的时候，所以我们可以通过 env 的方式传递需要使用的的 `fpm` 路径，很合理。

#### cron 阶段
```bash
# We need a CRON container to the Laravel Scheduler.
# We'll start with the CLI container as our base,
# as we only need to override the CMD which the container starts with to point at cron
FROM cli as cron

WORKDIR /opt/apps/laravel-in-kubernetes

# We want to create a laravel.cron file with Laravel cron settings, which we can import into crontab,
# and run crond as the primary command in the forground
RUN touch laravel.cron && \
    echo "* * * * * cd /opt/apps/laravel-in-kubernetes && php artisan schedule:run" >> laravel.cron && \
    crontab laravel.cron

CMD ["crond", "-l", "2", "-f"]
```

这里我没明白为什么不单独一个文件到 `docker` 目录下，然后复制进去呢？然后直接执行定时任务。

### docker-compose
这里就是替换本地 sail 默认的文件了
```bash
version: '3'
services:
    # We need to run the FPM container for our application
    laravel.fpm:
        build:
            context: .
            target: fpm_server
        image: laravel-in-kubernetes/fpm_server
        # We can override any env values here.
        # By default the .env in the project root will be loaded as the environment for all containers
        environment:
            APP_DEBUG: "true"
        # Mount the codebase, so any code changes we make will be propagated to the running application
        volumes:
            # Here we mount in our codebase so any changes are immediately reflected into the container
            - '.:/opt/apps/laravel-in-kubernetes'
        networks:
            - laravel-in-kubernetes

    # Run the web server container for static content, and proxying to our FPM container
    laravel.web:
        build:
            context: .
            target: web_server
        image: laravel-in-kubernetes/web_server
        # Expose our application port (80) through a port on our local machine (8080)
        ports:
            - '8080:80'
        environment:
            # We need to pass in the new FPM hst as the name of the fpm container on port 9000
            FPM_HOST: "laravel.fpm:9000"
        # Mount the public directory into the container so we can serve any static files directly when they change
        volumes:
            # Here we mount in our codebase so any changes are immediately reflected into the container
            - './public:/opt/apps/laravel-in-kubernetes/public'
        networks:
            - laravel-in-kubernetes
    # Run the Laravel Scheduler
    laravel.cron:
        build:
            context: .
            target: cron
        image: laravel-in-kubernetes/cron
        # Here we mount in our codebase so any changes are immediately reflected into the container
        volumes:
            # Here we mount in our codebase so any changes are immediately reflected into the container
            - '.:/opt/apps/laravel-in-kubernetes'
        networks:
            - laravel-in-kubernetes
    # Run the frontend, and file watcher in a container, so any changes are immediately compiled and servable
    laravel.frontend:
        build:
            context: .
            target: frontend
        # Override the default CMD, so we can watch changes to frontend files, and re-transpile them.
        command: ["npm", "run", "watch"]
        image: laravel-in-kubernetes/frontend
        volumes:
            # Here we mount in our codebase so any changes are immediately reflected into the container
            - '.:/opt/apps/laravel-in-kubernetes'
            # Add node_modeules as singular volume.
            # This prevents our local node_modules from being propagated into the container,
            # So the node_modules can be compiled for each of the different architectures (Local, Image)
            - '/opt/app/node_modules/'
        networks:
            - laravel-in-kubernetes

networks:
    laravel-in-kubernetes:
```

直接上示例就好了，没什么可以说的，我自己都理解了，如果你有不懂的地方，就需要查询一下了。注意上面 web 不分，就是用了，我刚才说的 nginx 变量
```bash
environment:
            # We need to pass in the new FPM hst as the name of the fpm container on port 9000
            FPM_HOST: "laravel.fpm:9000"
```


## 总结
今天到先到这里，后续就开始要在服务器进行部署了。