ohMyDocker
============

<img src="http://data.aymen-fnayou.com/oh-my-docker_logo.png" width="120px" align="left"/>

[![Build Status](https://travis-ci.org/fnayou/oh-my-docker.svg?branch=master)](https://travis-ci.org/fnayou/oh-my-docker)

**ohMyDocker** gives you full development environment for *PHP & Symfony*. With Nginx & PHP-FPM & MariaDB & Postgres etc.

## Installation

1. [download the latest release][link-release] or clone the repo
2. rename `docker-compose.yml.dist`
3. customize your `docker-compose.yml`
4. Build/run containers with (with and without detached mode)

```bash
$ docker-compose build
$ docker-compose up # or with "-d" option for "detached mode"
```

## About Containers

Here are the `docker-compose` built images:

* `nginx`: the Nginx web server container, volumes are gathered from the PHP container,
* `php`: the PHP-FPM container in which the application volume is mounted (OPcache & optimized for Docker usage),
* `mariadb`: the MariaDB MySQL container,
* `postgres`: the Postgres container,
* `mongo`: the MongoDB container,
* `couchdb`: the CouchDB container,

This results in the following running containers:

```bash
$ docker-compose ps
           Name                   Command               State       Ports
----------------------------------------------------------------------------------------
ohmydocker_nginx_1       nginx -c /nginx.conf             Up    0.0.0.0:8080->80/tcp
ohmydocker_php_1         docker-php-entrypoint php-fpm    Up    0.0.0.0:9000->9000/tcp
ohmydocker_mariadb_1     docker-entrypoint.sh mysqld      Up    0.0.0.0:3306->3306/tcp
ohmydocker_postgres_1    docker-entrypoint.sh postgres    Up    0.0.0.0:5432->5432/tcp
ohmydocker_mongo_1       docker-entrypoint.sh mongod      Up    0.0.0.0:27017->27017/tcp
ohmydocker_couchdb_1     tini -- /docker-entrypoint ...   Up    0.0.0.0:5984->5984/tcp
```

With this configuration, you can set up `docker` for one or many projects. all you have to do is to :

* update PHP `volumes` with paths to your projects
* rename `docker/nginx/nginx.conf.dist` to `docker/nginx/nginx.conf` and update project and mailcatcher configuration
* rename `docker/mariadb/env.dist` to `docker/mariadb/.env` and update configuration
* rename `docker/postgres/env.dist` to `docker/postgres/.env` and update configuration

### Nginx Container

Basic and full Nginx configuration within `nginx.conf` file. you can customize/duplicate the **project section** to fit your need.

```nginx
    # project configuration
    server {
        listen 80;
        server_name project.loc www.project.loc;
        root /var/www/html/web;

        location / {
            # try to serve file directly, fallback to app.php
            try_files $uri /app.php$is_args$args;
        }
        # PROD
        location ~ ^/app\.php(/|$) {
            fastcgi_pass php-upstream;
            fastcgi_split_path_info ^(.+\.php)(/.*)$;
            include fastcgi_params;
            # When you are using symlinks to link the document root to the
            # current version of your application, you should pass the real
            # application path instead of the path to the symlink to PHP
            # FPM.
            # Otherwise, PHP's OPcache may not properly detect changes to
            # your PHP files (see https://github.com/zendtech/ZendOptimizerPlus/issues/126
            # for more information).
            fastcgi_param  SCRIPT_FILENAME  $realpath_root$fastcgi_script_name;
            fastcgi_param DOCUMENT_ROOT $realpath_root;
            # Prevents URIs that include the front controller. This will 404:
            # http://domain.tld/app.php/some-path
            # Remove the internal directive to allow URIs like this
            internal;
        }

        error_log /var/log/nginx/project_error.log;
        access_log /var/log/nginx/project_access.log;
    }

    # mail catcher
    server {
        listen 80;
        server_name mailcatcher.project.loc;

        location / {
            proxy_pass http://mailcatcher-upstream;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Host $server_name;
            proxy_intercept_errors on;
        }

        error_log /var/log/nginx/mailcatcher_error.log;
        access_log /var/log/nginx/mailcatcher_access.log;
    }
```

> logs are shared within `docker/logs` directory.

### PHP Container

Built on the top of the wonderful [docker-library/php](https://github.com/docker-library/php/tree/4677ca134fe48d20c820a19becb99198824d78e3) so you can switch from many version of PHP just by customizing the `docker/php/Dockerfile`.

The PHP container includes :

* opcache, iconv, mcrypt, mbstring, intl, pdo, pdo_mysql, pdo_pgsql, gd and xdebug.
* **opcache** pre configured
* ready to use **composer**

> **timezone** set to `Europe/Paris`

### MariaDB & Postgres Containers

You can customize configuration (like password, user and database) within `docker/mariadb/env` or `docker/postgres/env`.

> all data are persisted within `docker/data` directory.

## Useful commands

I know that everyone has his own alias and functions for Docker. but here are some useful tricks.

```bash
#docker alias
 alias dc='docker-compose'
 alias dcup='docker-compose up -d'
 alias dcps='docker-compose ps'
 alias dcb='docker-compose build'
 alias dcd='docker-compose down'
 alias dcs='docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)'

 #docker functions
 dci(){ docker inspect "$@"; }
 dcip(){ docker inspect $(docker ps -f name="$@" -q) | grep IPAddress; }
 dcbash(){ docker-compose run "$@" bash; }
```

## Credits

- Aymen FNAYOU : [website][link-author] / [github][link-github]

## License

![license](https://img.shields.io/badge/license-MIT-lightgrey.svg) Please see [License File](LICENSE) for more information.

[link-author]: https://aymen-fnayou.com
[link-github]: https://github.com/fnayou
[link-release]: https://github.com/fnayou/oh-my-docker/releases
