![example workflow](https://github.com/denshvetsov/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)
### Cтек технологий:
[![Python](https://img.shields.io/badge/-Python-464646?style=flat&logo=Python&logoColor=56C0C0&color=008080)](https://www.python.org/)
[![Django](https://img.shields.io/badge/-Django-464646?style=flat&logo=Django&logoColor=56C0C0&color=008080)](https://www.djangoproject.com/)
[![Django REST Framework](https://img.shields.io/badge/-Django%20REST%20Framework-464646?style=flat&logo=Django%20REST%20Framework&logoColor=56C0C0&color=008080)](https://www.django-rest-framework.org/)
[![PostgreSQL](https://img.shields.io/badge/-PostgreSQL-464646?style=flat&logo=PostgreSQL&logoColor=56C0C0&color=008080)](https://www.postgresql.org/)
[![JWT](https://img.shields.io/badge/-JWT-464646?style=flat&color=008080)](https://jwt.io/)
[![Nginx](https://img.shields.io/badge/-NGINX-464646?style=flat&logo=NGINX&logoColor=56C0C0&color=008080)](https://nginx.org/ru/)
[![gunicorn](https://img.shields.io/badge/-gunicorn-464646?style=flat&logo=gunicorn&logoColor=56C0C0&color=008080)](https://gunicorn.org/)
[![Docker](https://img.shields.io/badge/-Docker-464646?style=flat&logo=Docker&logoColor=56C0C0&color=008080)](https://www.docker.com/)
[![Docker-compose](https://img.shields.io/badge/-Docker%20compose-464646?style=flat&logo=Docker&logoColor=56C0C0&color=008080)](https://www.docker.com/)
[![Docker Hub](https://img.shields.io/badge/-Docker%20Hub-464646?style=flat&logo=Docker&logoColor=56C0C0&color=008080)](https://www.docker.com/products/docker-hub)
[![GitHub%20Actions](https://img.shields.io/badge/-GitHub%20Actions-464646?style=flat&logo=GitHub%20actions&logoColor=56C0C0&color=008080)](https://github.com/features/actions)
[![Yandex.Cloud](https://img.shields.io/badge/-Yandex.Cloud-464646?style=flat&logo=Yandex.Cloud&logoColor=56C0C0&color=008080)](https://cloud.yandex.ru/)

# Проект YaMDb

Проект YaMDb собирает отзывы (Review) пользователей на произведения (Title). Произведения делятся на категории: «Книги», «Фильмы», «Музыка». Сами произведения в YaMDb не хранятся, здесь нельзя посмотреть фильм или послушать музыку.

## Workflow
* tests - Проверка кода на соответствие стандарту PEP8 (с помощью пакета flake8) и запуск pytest. Дальнейшие шаги выполнятся только если push был в ветку master или main.
* build_and_push_to_docker_hub - Сборка и доставка докер-образов на Docker Hub
* deploy - Автоматический деплой проекта на боевой сервер. Выполняется копирование файлов из репозитория на сервер:
* send_message - Отправка уведомления в Telegram

### Подготовка для запуска workflow
Отредактируйте файл `nginx/default.conf` и в строке `server_name` впишите IP виртуальной машины (сервера).  
Скопируйте подготовленные файлы `docker-compose.yaml` и `nginx/default.conf` из вашего проекта на сервер:

Зайдите в репозиторий на локальной машине и отправьте файлы на сервер.
```
scp docker-compose.yaml <username>@<host>/home/<username>/docker-compose.yaml
sudo mkdir nginx
scp default.conf <username>@<host>/home/<username>/nginx/default.conf
```
В репозитории на Гитхабе добавьте данные в `Settings - Secrets - Actions secrets`:
```
DOCKERHUB_LOGIN - имя пользователя в DockerHub
DOCKERHUB_PASS - пароль пользователя в DockerHub
HOST - ip-адрес сервера
USER - пользователь
SSH_KEY - приватный ssh-ключ (публичный должен быть на сервере)
PASSPHRASE - кодовая фраза для ssh-ключа
DB_ENGINE - django.db.backends.postgresql
DB_HOST - db
DB_PORT - 5432
SECRET_KEY - секретный ключ приложения django (необходимо чтобы были экранированы или отсутствовали скобки)
ALLOWED_HOSTS - список разрешённых адресов
TELEGRAM_TO - id своего телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
TELEGRAM_TOKEN - токен бота (получить токен можно у @BotFather, /token, имя бота)
DB_NAME - postgres (по умолчанию)
POSTGRES_USER - postgres (по умолчанию)
POSTGRES_PASSWORD - postgres (по умолчанию)
```

## Как запустить проект на сервере:
Установите Docker и Docker-compose:
```
sudo apt install docker.io
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
Проверьте корректность установки Docker-compose:
```
sudo  docker-compose --version
```
### После успешного деплоя:
Соберите статические файлы (статику):
```
sudo docker-compose exec web python manage.py collectstatic --no-input
```
Примените миграции:
```
sudo docker-compose exec web python manage.py makemigrations
sudo docker-compose exec web python manage.py migrate --noinput
```
Создайте суперпользователя:
```
sudo docker-compose exec web python manage.py createsuperuser
```
### Пользовательские роли

* **Аноним** — может просматривать описания произведений, читать отзывы и комментарии.
* **Аутентифицированный пользователь (user)**— может читать всё, как и Аноним, дополнительно может публиковать отзывы и ставить рейтинг произведениям (фильмам/книгам/песенкам), может комментировать чужие отзывы и ставить им оценки; может редактировать и удалять свои отзывы и комментарии.
* **Модератор (moderator)** — те же права, что и у Аутентифицированного пользователя плюс право удалять любые отзывы и комментарии.
* **Администратор (admin)** — полные права на управление проектом и всем его содержимым. Может создавать и удалять произведения, категории и жанры. Может назначать роли пользователям.
* **Администратор Django** — те же права, что и у роли Администратор.
    
### Алгоритм регистрации пользователей

1. Пользователь отправляет POST-запрос с параметром email на /api/v1/auth/email/.
2. YaMDB отправляет письмо с кодом подтверждения (confirmation_code) на адрес email .
3. Пользователь отправляет POST-запрос с параметрами email и confirmation_code на /api/v1/auth/token/, в ответе на запрос ему приходит token (JWT-токен).

Эти операции выполняются один раз, при регистрации пользователя. В результате пользователь получает токен и может работать с API, отправляя этот токен с каждым запросом.

После регистрации и получения токена пользователь может отправить PATCH-запрос на /api/v1/users/me/ и заполнить поля в своём профайле (описание полей — в документации).
Если пользователя создаёт администратор (например, через POST-запрос api/v1/users/...) — письмо с кодом отправлять не нужно. 

### Ресурсы API YaMDb

* Ресурс AUTH: аутентификация.
* Ресурс USERS: пользователи.
* Ресурс TITLES: произведения, к которым пишут отзывы (определённый фильм, книга или песенка).
* Ресурс CATEGORIES: категории (типы) произведений («Фильмы», «Книги», «Музыка»).
* Ресурс GENRES: жанры произведений. Одно произведение может быть привязано к нескольким жанрам.
* Ресурс REVIEWS: отзывы на произведения. Отзыв привязан к определённому произведению.
* Ресурс COMMENTS: комментарии к отзывам. Комментарий привязан к определённому отзыву.

Каждый ресурс описан в документации: указаны эндпойнты (адреса, по которым можно сделать запрос), разрешённые типы запросов, права доступа и дополнительные параметры, если это необходимо.

### Связанные данные и каскадное удаление

При удалении объекта пользователя User удаляться все отзывы и комментарии этого пользователя (вместе с оценками-рейтингами).  
При удалении объекта произведения Title удаляться все отзывы к этому произведению и комментарии к ним.  
При удалении объекта категории Category не удаляться связанные с этой категорией произведения (Title).  
При удалении объекта жанра Genre не удаляться связанные с этим жанром произведения (Title).  
При удалении объекта отзыва Review будут удалены все комментарии к этому отзыву.

## Ссылки на проект:
* http://51.250.25.105/api/v1/titles/
* http://51.250.25.105/api/v1/categories/
* http://51.250.25.105/api/v1/genres/
* http://51.250.25.105/redoc/
* http://51.250.25.105/admin/

***Автор проекта:***
* Александр Пименов | Github:https://github.com/Lets-dancing |
