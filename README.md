**Configurações para Ambiente de Desenvolvimento Local. Não utilizar em Ambientes de Produção.**

## Imagens

### Default
Utilize apenas para a aplicação, sem queue worker.

### Supervisor

Gerencia o queue worker junto à aplicação. **Use esta imagem apenas se o processamento de filas for essencial para o seu serviço**. Em outros casos, utilize o Supervisor separadamente em outro container.

## Docker Compose

Dividi a aplicação em 4 partes para organizar a escalabilidade e manutenção:

### app
Executa o build da imagem contendo o PHP 8.2 FPM rodando em um Linux Alpine.
> volumes:
>    - ./:/app

Realizo o mapeamento do diretório raiz do projeto para dentro de `/app` no container, onde configuro como `workdir` e já crio uma cópia ao efetuar o build da imagem.

### web
Utiliza a imagem do nginx:1.25
> ./public/:/app/public

Realizo o mapeamento da pasta pública para permitir o acesso a arquivos e imagens, se necessário.

### database
Utiliza a imagem do postgres:15.2. Sinta-se livre para utilizar qualquer banco desejado.

### cache
Utiliza a imagem do redis:7. Sem segredos, é apenas cache. Se optar por utilizar cache em arquivo, pode remover.

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
>>> ├── infra
    │   └── docker
    │       ├── Dockerfile
    │       ├── nginx
    │       │   └── nginx.conf
    │       ├── php
    │       │   └── php.ini
    │       └── supervisor
    │           └── supervisord.conf
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
>>> ├── docker-compose.yml 
    └── vite.config.js
```