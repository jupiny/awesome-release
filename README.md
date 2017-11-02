## How to start
`.pem`파일이 수정될 수 없도록 권한 변경

`$ chmod 400 [PEM NAME].pem`

서버에 SSH 접속 

`$ ssh ubuntu@[IP ADDRESS] -i [PEM NAME].pem`

패키지 업데이트, 업그레이드

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

## Install git
`$ sudo apt-get install git`

## Install pyenv
[https://github.com/yyuu/pyenv/wiki/Common-build-problems](https://github.com/yyuu/pyenv/wiki/Common-build-problems)

```
$ sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils
```

[https://github.com/yyuu/pyenv-installer](https://github.com/yyuu/pyenv-installer)

`$ curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash`

`~/.bash_profile`에 아래의 세 줄 추가

```
export PATH="/home/ubuntu/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

`~/.bash_profile` 재실행

`$ source ~/.bash_profile`

pyenv로 원하는 Python 버젼 설치

`$ pyenv install [PYTHON VERSION]`

Python 버젼에 해당하는 가상환경 생성

`$ pyenv virtualenv [PYTHON VERSION] [ENV NAME]`

`~/.bash_profile`에 다음 줄 추가 

`$ pyenv activate [ENV NAME]`

`~/.bash_profile` 재실행

`$ source ~/.bash_profile`

## Install autoenv
`$ git clone git://github.com/kennethreitz/autoenv.git ~/.autoenv`

`~/.bash_profile`에 다음 줄 추가

`source ~/.autoenv/activate.sh`

`~/.bash_profile` 재실행

`$ source ~/.bash_profile`

## Install Postgresql
[https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)

`$ sudo apt-get install postgresql postgresql-contrib`

최고 관리자 계정인 postgres으로 전환

`$ sudo -i -u postgres`

Postgresql의 CLI(Command Line Interface)인 `psql`로 postgres 데이터베이스에 접속

`$ psql -d postgres` 또는 `$ psql postgres` 또는 `$ psql`

새로운 계정의 이름과 비밀번호 생성

`CREATE USER [USERNAME] WITH PASSWORD '[PASSWORD]';`

새로운 데이터베이스 생성

`CREATE DATABASE [DATABASE NAME];`

데이터베이스의 권한을 유저에게 부여

`GRANT ALL PRIVILEGES ON DATABASE [DATABASE NAME] to [USERNAME];`

psql 종료
 
`\q` 또는 **[Ctrl + d]**

postgres 계정에서 나온 후,

`$ exit`

root 계정으로 전환

`$ sudo -s`

아까 만든 계정 추가

`$ adduser [USERNAME]`

비밀번호를 통해서 데이터베이스에 들어올 수 있게 하기 위해 설정 변경

`$ sudo vi /etc/postgresql/[POSTGRESQL VERSION]/main/pg_hba.conf`

90번째 줄에서 `peer`를 `md5`로 변경 후 데이터베이스 재시작

`$ sudo service postgresql restart`

root 계정에서 나온 후,

`$ exit`

아까 만든 계정으로 새로 생성한 데이터베이스 접속되는지 확인

`$ psql -U [USERNAME] -d [DATABASE NAME] -W`

## Install nginx

[https://developers.google.com/speed/pagespeed/module/build_ngx_pagespeed_from_source#build-instructions](https://developers.google.com/speed/pagespeed/module/build_ngx_pagespeed_from_source#build-instructions)

`$ sudo apt-get install build-essential zlib1g-dev libpcre3 libpcre3-dev unzip`

```
$ cd
$ NGINX_VERSION=1.8.1
$ wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
$ tar -xvzf nginx-${NGINX_VERSION}.tar.gz
$ cd nginx-${NGINX_VERSION}/
$ ./configure --with-http_ssl_module
$ make 
$ sudo make install
```

nginx를  편리하게 구동하기 위해

[https://github.com/JasonGiedymin/nginx-init-ubuntu](https://github.com/JasonGiedymin/nginx-init-ubuntu)

```
$ sudo wget https://raw.githubusercontent.com/JasonGiedymin/nginx-init-ubuntu/master/nginx -O /etc/init.d/nginx
$ sudo chmod +x /etc/init.d/nginx
```

nginx 시작하여 되는지 확인

`$ sudo service nginx start`

만약 nginx 설정 망해서 처음부터 해야 될 때, nginx 관련 파일 모두 삭제하는 명령어

`sudo rm -f -R /usr/local/nginx && rm -f /usr/local/sbin/nginx`


## Install Redis
[https://www.digitalocean.com/community/tutorials/how-to-install-and-use-redis](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-redis)

`$ sudo apt-get install tcl8.5`

```
$ wget http://download.redis.io/releases/redis-stable.tar.gz
$ tar xzf redis-stable.tar.gz
$ cd redis-stable
$ make
$ make test
$ sudo make install
$ cd utils
$ sudo ./install_server.sh
$ sudo service redis_6379 start
```

데이터베이스에 접속되는지 확인

`$ redis-cli`

## Get SSL authentication
[https://dobest.io/gettings-started-with-jupyterhub/](https://dobest.io/gettings-started-with-jupyterhub/)

```
$ sudo service nginx stop
$ git clone https://github.com/letsencrypt/letsencrypt
$ cd letsencrypt
$ ./letsencrypt-auto certonly --standalone -d [DOMAIN ADDRESS]
```

개인키와 인증키가 잘 가져와졌는지 확인

- 개인키 :

`$ sudo cat /etc/letsencrypt/live/[DOMAIN ADDRESS]/privkey.pem`

- 인증키 :

`$ sudo cat /etc/letsencrypt/live/[DOMAIN ADDRESS]/fullchain.pem`

## Setting nginx

`$ sudo vi /usr/local/nginx/conf/nginx.conf`

아래와 같이 저장

```
worker_processes  1;

events {
    worker_connections  1024;
    accept_mutex off;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile on;

    server {
        listen       80;
        server_name  [DOMAIN ADDRESS];

        client_max_body_size 4G;
        keepalive_timeout 5;

        return 301 https://$server_name$request_uri;
    }

    server {
        listen  443 default_server ssl;
        server_name  [DOMAIN ADDRESS];

        client_max_body_size 4G;
        keepalive_timeout 5;

        ssl_certificate                 /etc/letsencrypt/live/[DOMAIN ADDRESS]/fullchain.pem;
        ssl_certificate_key             /etc/letsencrypt/live/[DOMAIN ADDRESS]/privkey.pem;

        location / {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header HOST $http_host;
            proxy_set_header X-NginX-Proxy true;

            proxy_pass http://127.0.0.1:5736;
            proxy_redirect off;
        }
    }
}
```

5736번 포트로 서버를 하나 구동시켜 [DOMAIN ADDRESS]로 잘 들어가지는 지 확인

`$ python -m http.server 5736`

## Get django project
`$ git clone [PROJECT GIT ADDRESS]`

라이브러리를 설치하기 전에

`Pillow`라는 라이브러리가 프로젝트에 설치되어 있어야 한다면, 아래를  실행

`$ sudo apt-get install libtiff5-dev libjpeg8-dev zlib1g-dev \
    libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python-tk`

`psycopg2`라는 라이브러리가 프로젝트에 설치되어 있어야 한다면, 아래를 실행

`$ sudo apt-get install postgresql-server-dev-[POSTGRESQL VERSION]`

라이브러리 설치

`$ pip install -r requirements.txt`

`.env` 파일을 만들어

`$ vi .env`

기존의 `.env` 파일 내용 복사한다.

이 때  `.env`에`DJANGO_SETTINGS_MODULE`나 `DATABASE_URL`관련 설정도 해두었다면 production 버젼으로 수정

`$ make migrate`

`$ sudo apt-get install npm nodejs-legacy`

`$ sudo npm install -g yuglify`

`$ python [DJANGO_PROJECT_FOLDER]/manage.py collectstatic`

서버를 구동시켜 잘 나오는지 확인

`$ honcho start`

만약 `honcho`를 재시작해야 경우가 생긴다면,


먼저 해당 포트번호를 사용하는  서비스의 PID를 확인하여

`$ lsof -i :[PORT NUMBER]`

프로세스를 강제로 종료시킨 후 다시 `honcho`를 시작

`$ kill -9 [PID]`
 
`$ honcho start`
