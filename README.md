# PASSOS para Funcionar XDebug com Laravel 9, no WSL, usando Docker com SAIL.

## Modelo 1 - Passos:
1- Altere o `.env` e adicione as seguintes linhas:

    SAIL_XDEBUG_MODE=debug,develop,coverage

    WWWGROUP=1000
    WWWUSER=1000

2- Instale o sail com `php artisan sail:install`;
3- Publique os arquivos do Sail com `php artisan sail:publish`
4- Altere o arquivo `docker/8.2/php.ini` e adicione as seguintes linhas:

    [XDebug]
    xdebug.mode=debug,develop,coverage
    xdebug.start_with_request=yes
    xdebug.discover_client_host=true
    xdebug.client_host=host.docker.internal
    xdebug.client_port=9003

5- Execute o comando `sail up`
6- Execute o Debug no VSCODE
7- Adicione um breakpoint
8- Abra uma página do projeto que execute um breakpoint

## Modelo 2 - Passos:

Siga as etapas 1 à 3 do Modelo 1;

No arquivo `.env`, adicione as linhas:

SAIL_XDEBUG_PORT=9003
SAIL_XDEBUG_INI_FILE="/etc/php/8.2/cli/conf.d/99-sail.ini"

No arquivo `docker-compose.yml` adicione as seguintes linhas, para o parâmetro `args` do serviço `laravel.tests`:

    XDEBUG: '${APP_DEBUG:-false}'
    XDEBUG_PORT: '${SAIL_XDEBUG_PORT:-9003}'
    XDEBUG_INI_FILE: '${SAIL_XDEBUG_INI_FILE:-false}'

Adicione no arquivo `docker/8.2/Dockerfile` na linha 55 (Antes de `EXPOSE 8000`) as seguintes linhas:

    ARG XDEBUG
    ARG XDEBUG_PORT
    ARG XDEBUG_INI_FILE

    RUN if [ "${XDEBUG_INI_FILE}" = 'false' ] ; then \
      XDEBUG_INI_FILE = $(find /etc/php/ -name 99-sail.ini) ;\
    fi;
    RUN if [ "${XDEBUG}" = 'true' ] ; then \
            echo "[XDebug]" >> ${XDEBUG_INI_FILE} \
            && echo "xdebug.mode=debug,develop,coverage" >> ${XDEBUG_INI_FILE} \
            && echo "xdebug.start_with_request=yes" >> ${XDEBUG_INI_FILE} \
            && echo "xdebug.discover_client_host=true" >> ${XDEBUG_INI_FILE} \
            && echo "xdebug.client_host=host.docker.internal" >> ${XDEBUG_INI_FILE} \
            && echo "xdebug.client_port=${XDEBUG_PORT}" >> ${XDEBUG_INI_FILE} ;\
    fi;

Execute o comando `sail up --build --force-recreate`
Execute os passos de 6 ao final, do Modelo 1.

### References:
https://blog.devgenius.io/xdebug-laravel-sail-project-in-vs-code-b7b73e3dedf7