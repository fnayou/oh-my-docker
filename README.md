ohMyDocker
============

<img src="http://data.aymen-fnayou.com/oh-my-docker_logo.png" width="120px" align="left"/>

[![Build Status](https://travis-ci.org/fnayou/oh-my-docker.svg?branch=master)](https://travis-ci.org/fnayou/oh-my-docker)

**ohMyDocker** gives you full development environment for *PHP & Symfony*. With Nginx & PHP-FPM & MariaDB & Postgres etc.

## Installation

1. [download the latest release][link-release] or clone the repo
2. customize your `docker-compose.yml`
3. Build/run containers with (with and without detached mode)

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

- update PHP `volumes` with paths to your projects
- update (duplicate *project section*) within `nginx.conf` file

### Nginx Container

Basic and full Nginx configuration within `nginx.conf` file. you can customize/duplicate the **project section** to fit your need.

```nginx
    # project configuration
    server {
        server_name project.tld www.project.tld;
        root /var/www/html/web;

        # server configuration

        error_log /var/log/nginx/project_error.log;
        access_log /var/log/nginx/project_access.log;
    }
```

> logs are shared within `docker/logs` directory.

### PHP Container

Built on the top of the wonderful [docker-library/php](https://github.com/docker-library/php/tree/4677ca134fe48d20c820a19becb99198824d78e3) so you can switch from many version of PHP just by customizing the `docker/php/Dockerfile`.

The PHP container includes :

- opcache, iconv, mcrypt, mbstring, intl, pdo, pdo_mysql, pdo_pgsql, gd and xdebug.
- **opcache** pre configured
- ready to use **composer**

> **timezone** set to `Europe/Paris`

### MariaDB & Postgres Containers

You can customize configuration (like password, user and database) within `docker/mariadb/env` or `docker/postgres/env`.

> all data are persisted within `docker/data` directory.

## Useful commands

I know that everyone has his own alias, functions for Docker. but here are some useful tricks.

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

[Aymen FNAYOU][link-author]

## License

![license](https://img.shields.io/badge/license-MIT-lightgrey.svg) Please see [License File](LICENSE) for more information.

[link-author]: https://aymen-fnayou.com
[link-release]: https://github.com/fnayou/oh-my-docker/releases
