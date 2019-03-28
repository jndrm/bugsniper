# Sentry Installation

## Preparation
```sh
$ export LC_ALL="en_US.UTF-8"
$ sudo apt-get install -y python-pip libpq-dev redis-server postgresql supervisor
# change pip source
$ mkdir ~/.pip && echo -e '[global]\nindex-url = https://pypi.tuna.tsinghua.edu.cn/simple' > ~/.pip/pip.conf
$ sudo pip install -U pip
$ sudo pip install -U virtualenv
$ virtualenv /approot/sentry/
$ source /approot/sentry/bin/activate
```

## Install Sentry
```sh
$ pip install -U sentry
```

## Initializing the Configuration
```sh
$ sudo /approot/sentry/bin/sentry init /etc/sentry
```

## Running Migrations
```sh
$ sudo -u postgres createdb -E utf-8 sentry
# edit /etc/postgresql/9.5/main/pg_hba.conf
# local   all             postgres                                trust
# sudo /etc/init.d/postgresql reload
$ SENTRY_CONF=/etc/sentry sentry upgrade
```

## Install Nginx
```sh
$ sudo apt-get install -y libpcre3 libpcre3-dev openssl libssl-dev
$ wget http://nginx.org/download/nginx-1.14.0.tar.gz
$ tar -xf nginx-1.14.0.tar.gz && cd nginx-1.14.0
$ ./configure --user=www-data --group=www-data --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-http_sub_module
$ make
$ sudo make install
```

### Proxying with Nginx
```
    server {
        listen       80;
        server_name  YOUR_DOMAIN;

        location / {
            proxy_pass         http://localhost:9000;
            proxy_redirect     off;

            proxy_set_header   Host              $host;
            proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }
    }
```

## Starting the Web Service
```sh
$ SENTRY_CONF=/etc/sentry sentry run web
```
## Starting Background Workers
```sh
$ SENTRY_CONF=/etc/sentry sentry run worker
```

## Running Sentry as a Service

### Configure supervisord
```
[program:sentry-web]
directory=/approot/sentry/
environment=SENTRY_CONF="/etc/sentry"
command=/approot/sentry/bin/sentry run web
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=syslog
stderr_logfile=syslog

[program:sentry-worker]
directory=/approot/sentry/
environment=SENTRY_CONF="/etc/sentry"
command=/approot/sentry/bin/sentry run worker
autostart=true
user=www-data
autorestart=true
redirect_stderr=true
stdout_logfile=syslog
stderr_logfile=syslog

[program:sentry-cron]
directory=/approot/sentry/
environment=SENTRY_CONF="/etc/sentry"
command=/approot/sentry/bin/sentry run cron
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=syslog
stderr_logfile=syslog
```