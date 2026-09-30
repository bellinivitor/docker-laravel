**Configurações para Ambiente de Desenvolvimento Local com Laravel 13. Não utilizar em Ambientes de Produção.**

## Estrutura

Cada variante é autocontida: tem a própria pasta `docker/`, o `docker-compose.yml` e o `.dockerignore`.

```bash
    .
    ├── default
    │   ├── .dockerignore
    │   ├── docker-compose.yml
    │   └── docker
    │       ├── Dockerfile
    │       ├── nginx
    │       │   └── nginx.conf
    │       └── php
    │           └── php.ini
    ├── loadbalancer
    │   ├── .dockerignore
    │   ├── docker-compose.yml
    │   └── docker
    │       ├── Dockerfile
    │       ├── nginx
    │       │   └── nginx.conf
    │       └── php
    │           └── php.ini
    └── supervisor
        ├── .dockerignore
        ├── docker-compose.yml
        └── docker
            ├── Dockerfile
            ├── nginx
            │   └── nginx.conf
            ├── php
            │   └── php.ini
            └── supervisor
                └── supervisord.conf
```

Copie **todo o conteúdo** da variante desejada (incluindo o `.dockerignore`, que é um arquivo oculto) para a raiz do projeto Laravel:

```bash
cp -R default/. /caminho/do/projeto-laravel/
```

## Imagens

### Default
Utilize apenas para a aplicação, sem queue worker.

### Supervisor

Gerencia o PHP-FPM, o queue worker e o scheduler (`schedule:work`) junto à aplicação. **Use esta imagem apenas se o processamento de filas for essencial para o seu serviço**. Em outros casos, utilize o Supervisor separadamente em outro container.

### Loadbalancer

Mesma imagem do Default, sem queue worker, com duas instâncias da aplicação (`app1` e `app2`) atrás do nginx, que distribui as requisições via `upstream` (`least_conn`). Se uma instância cair, o nginx tenta a próxima. Cada resposta traz o header `X-Upstream` indicando qual instância atendeu.

Para adicionar uma instância, crie um novo serviço com `<<: *app` no `docker-compose.yml` e inclua o `server` correspondente no `upstream` do `nginx.conf`.

Como cada requisição pode cair numa instância diferente, sessão e cache não podem depender do container. Use `SESSION_DRIVER=redis` (ou `database`) e `CACHE_STORE=redis` no `.env`.

## Docker Compose

Dividi a aplicação em 4 partes para organizar a escalabilidade e manutenção:

### app
Executa o build da imagem contendo o PHP 8.5 FPM rodando em um Linux Alpine (o Laravel 13 exige PHP >= 8.3). A versão pode ser alterada pelo `ARG PHP_VERSION` do Dockerfile.

Extensões instaladas: `bcmath`, `gd`, `intl`, `pcntl`, `pdo_pgsql`, `redis` e `zip` (o `opcache` já vem embutido no PHP 8.5).
> volumes:
>    - ./:/app
>    - ./docker/php/php.ini:/usr/local/etc/php/conf.d/zz-app.ini

Realizo o mapeamento do diretório raiz do projeto para dentro de `/app` no container, onde configuro como `workdir` e já crio uma cópia ao efetuar o build da imagem.

O usuário `www-data` do container é ajustado para o UID/GID do host (`HOST_UID`/`HOST_GID`, padrão `1000`), evitando problemas de permissão em `storage/` e `bootstrap/cache/`. No Linux, defina no `.env` os valores retornados por `id -u` e `id -g`.

### web
Utiliza a imagem do nginx:1.28-alpine
> ./public:/app/public
>
> ./storage/app/public:/app/storage/app/public

Realizo o mapeamento da pasta pública e dos arquivos do `php artisan storage:link` para permitir o acesso a arquivos e imagens.

### database
Utiliza a imagem do postgres:18-alpine, com os dados persistidos no volume `database-data`. Sinta-se livre para utilizar qualquer banco desejado.

### cache
Utiliza a imagem do redis:8-alpine. Sem segredos, é apenas cache. Se optar por utilizar cache em arquivo, pode remover.

## Configuração do `.env`

O Docker Compose lê o `.env` do Laravel para configurar o banco e as portas:

```dotenv
DB_CONNECTION=pgsql
DB_HOST=database
DB_PORT=5432
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=secret

REDIS_HOST=cache

# Opcionais
APP_PORT=80
FORWARD_DB_PORT=5432
FORWARD_REDIS_PORT=6379
HOST_UID=1000
HOST_GID=1000
```

## Uso

```bash
docker compose up -d --build
```

Execute os comandos como `www-data` para não criar arquivos com dono `root`:

```bash
docker compose exec -u www-data app php artisan migrate
```

---

Sugestão de arquitetura:

```bash
    .
    ├── Domain
    ├── app
    ├── artisan
    ├── bootstrap
    ├── config
    ├── database
>>> ├── docker
    │   ├── Dockerfile
    │   ├── nginx
    │   │   └── nginx.conf
    │   ├── php
    │   │   └── php.ini
    │   └── supervisor
    │       └── supervisord.conf
    ├── package.json
    ├── phpunit.xml
    ├── public
    ├── resources
    ├── routes
    ├── storage
    ├── tests
    ├── vendor
    ├── composer.json
    ├── composer.lock
>>> ├── .dockerignore
>>> ├── docker-compose.yml
    └── vite.config.js
```
