# Проект по развертыванию GLPI с помощью docker

![Docker Pulls](https://img.shields.io/docker/pulls/diouxx/glpi) !
[Docker Stars](https://img.shields.io/docker/stars/diouxx/glpi) [![](https://images.microbadger.com/badges/image/diouxx/glpi.svg)](http://microbadger.com/images/diouxx/glpi "Get your own image badge on microbadger.com") ![Docker Cloud Automated build](https://img.shields.io/docker/cloud/automated/diouxx/glpi)

# Оглавление
- [Проект по развертыванию GLPI с помощью docker](#project-to-deploy-glpi-with-docker)

- [Оглавление](#table-of-contents)
- [Введение](#Введение)
  - [Аккаунты по умолчанию](#Аккаунты_по_умолчанию)
- [Развертывание с помощью командной строки (CLI)](#Развертывание_с_помощью_командной_строки_(CLI))
  - [Развертывание GLPI](#Развертывание_GLPI)
  - [Развертывание GLPI с существующей базой данных](#Развертывание_GLPI_с_существующей_базой_данных)
  - [Развертывание GLPI с данными базы данных и переменными.](#Развертывание_GLPI_с_данными_базы_данных_и_переменными.)
  - [Развертывание конкретной версии GLPI](#Развертывание_конкретной_версии_GLPI)
- [Развертывание с помощью docker-compose](#Развертывание_с_помощью_docker-compose)
  - [Развертывание без сохраняемых данных (для быстрого тестирования)](#Развертывание_без_сохраняемых_данных_(для_быстрого_тестирования))
  - [Развертывание определенной версии](#Развертывание_определенной_версии)
  - [Развертывание с постоянными данными](#Развертывание_с_постоянными_данными)
    - [mariadb.env](#mariadbenv)
    - [docker-compose .yml](#docker-compose-yml)
- [Переменные среды](#Переменные_среды)
  - [TIMEZONE](#timezone)

# Введение

Установка и запуск экземпляр GLPI с помощью докера.

## Аккаунты_по_умолчанию

Больше информации в 📄[Docs](https://glpi-install.readthedocs.io/en/latest/install/wizard.html#end-of-installation)

| Login/Password     | Роль              	    |
|--------------------|------------------------|
| glpi/glpi          | администратор GLPI     |
| tech/tech          | технический специалист	|
| normal/normal      | простой пользователь  	|
| post-only/postonly | только публикаия      	|

# Развертывание_с_помощью_командной_строки_(CLI)

## Развертывание_GLPI
```sh
docker run --name mariadb -e MARIADB_ROOT_PASSWORD=diouxx -e MARIADB_DATABASE=glpidb -e MARIADB_USER=glpi_user -e MARIADB_PASSWORD=glpi -d mariadb:10.7
docker run --name glpi --link mariadb:mariadb -p 80:80 -d diouxx/glpi
```

## Развертывание_GLPI_с_существующей_базой_данных
```sh
docker run --name glpi --link yourdatabase:mariadb -p 80:80 -d diouxx/glpi
```

## Развертывание_GLPI_с_данными_базы_данных_и_переменными.

Для использования в производственной среде или ежедневного использования рекомендуется использовать контейнер с томами для постоянных данных.

* Сначала создайте контейнер MariaDB с томом

```sh
docker run --name mariadb -e MARIADB_ROOT_PASSWORD=diouxx -e MARIADB_DATABASE=glpidb -e MARIADB_USER=glpi_user -e MARIADB_PASSWORD=glpi --volume /var/lib/mysql:/var/lib/mysql -d mariadb:10.7
```

* Затем создайте контейнер GLPI с томом и свяжите контейнер с MariaDB.


```sh
docker run --name glpi --link mariadb:mariadb --volume /var/www/html/glpi:/var/www/html/glpi -p 80:80 -d diouxx/glpi
```

Наслаждайтесь :)

## Развертывание_конкретной_версии_GLPI

По умолчанию при запуске Docker будет использоваться последняя версия GLPI. Для использования в производственной среде рекомендуется установить конкретную версию. Вот пример для версии 9.1.6:

```sh
docker run --name glpi --hostname glpi --link mariadb:mariadb --volume /var/www/html/glpi:/var/www/html/glpi -p 80:80 --env "VERSION_GLPI=9.1.6" -d diouxx/glpi
```

# Развертывание_с_помощью_docker-compose

## Развертывание_без_сохраняемых_данных_(для_быстрого_тестирования)
```yaml
version: "3.8"

services:
#Контейнер MariaDB
  mariadb:
    image: mariadb:10.7
    container_name: mariadb
    hostname: mariadb
    environment:
      - MARIADB_ROOT_PASSWORD=password
      - MARIADB_DATABASE=glpidb
      - MARIADB_USER=glpi_user
      - MARIADB_PASSWORD=glpi

#Контейнер GLPI
  glpi:
    image: diouxx/glpi
    container_name : glpi
    hostname: glpi
    ports:
      - "80:80"
```

## Развертывание_определенной_версии

```yaml
version: "3.8"

services:
#Контейнер MariaDB
  mariadb:
    image: mariadb:10.7
    container_name: mariadb
    hostname: mariadb
    environment:
      - MARIADB_ROOT_PASSWORD=password
      - MARIADB_DATABASE=glpidb
      - MARIADB_USER=glpi_user
      - MARIADB_PASSWORD=glpi

#Контейнер GLPI
  glpi:
    image: diouxx/glpi
    container_name : glpi
    hostname: glpi
    environment:
      - VERSION_GLPI=9.5.6
    ports:
      - "80:80"
```

## Развертывание_с_постоянными_данными

Для развертывания с помощью docker compose используются файлы *docker-compose.yml* и *mariadb.env*. Вы можете изменить **_mariadb.env_**, чтобы персонализировать такие параметры, как:

* MariaDB root password - пароль root БД MariaDB
* GLPI database         - имя БД GLPI
* GLPI user database    - имя пользователя БД GLPI
* GLPI user password    - пароль пользователя БД GLPI


### mariadb.env
```
MARIADB_ROOT_PASSWORD=diouxx
MARIADB_DATABASE=glpidb
MARIADB_USER=glpi_user
MARIADB_PASSWORD=glpi
```

### docker-compose .yml
```yaml
version: "3.2"

services:
#MariaDB Container
  mariadb:
    image: mariadb:10.7
    container_name: mariadb
    hostname: mariadb
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    env_file:
      - ./mariadb.env
    restart: always

#GLPI Container
  glpi:
    image: diouxx/glpi
    container_name : glpi
    hostname: glpi
    ports:
      - "80:80"
    volumes:
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
      - /var/www/html/glpi/:/var/www/html/glpi
    environment:
      - TIMEZONE=Europe/Brussels
    restart: always
```

To deploy, just run the following command on the same directory as files

```sh
docker-compose up -d
```

# Переменные_среды

## TIMEZONE
Если вам нужно установить часовой пояс для Apache и PHP

Из командной строки
```sh
docker run --name glpi --hostname glpi --link mariadb:mariadb --volumes-from glpi-data -p 80:80 --env "TIMEZONE=Europe/Brussels" -d diouxx/glpi
```
С использованием docker-compose

Измените эти настройки
```yaml
environment:
     TIMEZONE=Europe/Moscow
```
