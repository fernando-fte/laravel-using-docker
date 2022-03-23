# Criação de um projeto modelo de Laravel

Instale o composer `https://getcomposer.org/download/` Composer-Setup.exe.

Iniciando, com o Composer já instalado, inicie em um diretorio a instalação do Laravel `https://laravel.com/docs/9.x/installation#installation-via-composer`.

Para testar com o PHP instalado na maquina basta digitar `php -S localhost:9002 -t app\public` sendo *app\public* o diretório que irá rodar. Desta forma o PHP inicia uma rota host neste endereço.

Após configurar os arquivos `Dockerfile` e `nginx.conf`

```
// Dockerfile
FROM wyveo/nginx-php-fpm:latest

COPY . /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

WORKDIR /usr/share/nginx/html

RUN ln -s public html
RUN apt update; \
    apt install vim -y

EXPOSE 80
```

```
// nginx.conf
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /usr/share/nginx/html/public;
    index index.php index.html index.htm;

    server_name laravel_docker;

    location / { 
        try_files $uri $uri/ /index.php?query_string;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
Inicie o Docker com `> docker build -t laravel-docker .`, desta forma ele irá criar a imagem do projeto conforme os arquivos de configuração

Para rodar no Docker é necessário no terminal fazer a seguinte chamada `> docker run -d -p 9002:80 laravel-docker`, onde em `-p 9002:80` significa que ao acessar externamente você utilizará a porta `9002` e o docker internamente irá chamar a porta `80` para a aplicação.

Com `> docker ps` exibe os conteiner funcionando.

Com `> docker ps` lista os docker rodando e o ID deles.

Com `> docker stop $ID` para a imagem.

Com `> docker exec -it $ID /bin/sh` você abre o terminal


### Passo 2 - Criando um docker-composer

Para isso é necessário criar um arquivo `docker-compose.yml`

```
version: '3.3'
services:
  laravel-docker:
    build: ./
    ports:
      - 9002:80
    volumes:
      - ./:/usr/share/nginx/html
    restart: always
    networks:
      - docker

networks:
  docker:
    driver: bridge
```

Após isso inicie o script `> docker-compose up -d`
