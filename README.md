# API для проекта YaMDB в контейнере Docker
[![yamdb_final](https://github.com/TiEnddd/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)](https://github.com/TiEnddd/yamdb_final/actions)

### Возможности проекта
Представляет собой расширение возможностей проекта YaMDB для совершения удаленных операций.   
Благодаря этому проекту зарегистрированные и аутентифицированные пользователи получают 
возможность оставлять рецензии на произведения различных категорий, 
комментировать рецензии других пользователей,просматривать сформированные на основе оценок рейтинги произведений. 
Сайт не предоставляет прямой доступ или ссылки для ознакомления непосредственно с произведениями.
### Расширение функциональности
Функционал проекта адаптирован для использования PostgreSQL и развертывания в контейнерах Docker. Используются инструменты CI и CD.

Может быть недоступно в связи с прекращением обслуживания.
## Технологии
 - Python 3.7
 - Django 2.2.16
 - REST Framework 3.12.4
 - PyJWT 2.1.0
 - Django filter 21.1
 - Gunicorn 20.0.4
 - PostgreSQL 12.2
 - Docker 20.10.2
 - подробнее см. прилагаемый файл зависимостей requrements.txt
## Установка
### Шаблон описания файла .env
 - DB_ENGINE=django.db.backends.postgresql
 - DB_NAME=postgres
 - POSTGRES_USER=postgres
 - POSTGRES_PASSWORD=postgres
 - DB_HOST=db
 - DB_PORT=5432
 - SECRET_KEY=<секретный ключ проекта django>
### Инструкции для развертывания и запуска приложения
для Linux-систем все команды необходимо выполнять от имени администратора
- Склонировать репозиторий
```bash
git clone https://github.com/tienddd/yamdb_final.git
```
- Выполнить вход на удаленный сервер
- Установить docker на сервер:
```bash
apt install docker.io 
```

- Создать .env файл по предлагаемому выше шаблону. Обязательно изменить значения POSTGRES_USER и POSTGRES_PASSWORD
- Для работы с Workflow добавить в Secrets GitHub переменные окружения для работы:
    ```
    DB_ENGINE=<django.db.backends.postgresql>
    DB_NAME=<имя базы данных postgres>
    DB_USER=<пользователь бд>
    DB_PASSWORD=<пароль>
    DB_HOST=<db>
    DB_PORT=<5432>
    
    ```
    Workflow состоит из четырёх шагов:
     - Проверка кода на соответствие PEP8
     - Сборка и публикация образа бекенда на DockerHub.
     - Автоматический деплой на удаленный сервер.
     - Отправка уведомления в телеграм-чат.
- собрать и запустить контейнеры на сервере:
```bash
docker-compose up -d --build
```
- После успешной сборки выполнить следующие действия (только при первом деплое):
    * провести миграции внутри контейнеров:
    ```bash
    docker-compose exec web python manage.py migrate
    ```
    * собрать статику проекта:
    ```bash
    docker-compose exec web python manage.py collectstatic --no-input
    ```  
    * Создать суперпользователя Django, после запроса от терминала ввести логин и пароль для суперпользователя:
    ```bash
    docker-compose exec web python manage.py createsuperuser
    ```

### Команды для заполнения базы данными
- Заполнить базу данными
- Создать резервную копию данных:
```bash
docker-compose exec web python manage.py dumpdata > fixtures.json
```
* Пример POST-запроса<br/>   
    Регистрация нового пользователя и получение `confirmation_code`. Доступно без токена.  
    `POST http://127.0.0.1:8000/api/v1/auth/signup/`
    ```json
    {
        "email": "user@example.com",
        "username": "string"
    }
    ```
* Пример ответа:
    ```json
    {
        "email": "string",
        "username": "string"
    }
    ```
  В проекте настроен filebased способ отправки почты, confirmation_code будет находится в папке send_email базовой директории.
* Получение JWT-токена в обмен на `username` и `confirmation_code`. Доступно без токена.  
    `POST http://127.0.0.1:8000/api/v1/auth/token/`
    ```json
    {
        "username": "string",
        "confirmation_code": "string"
    }
    ```
* Пример ответа:
    ```json
    {
        "token": "string"
    }
    ```
  В дальнейшем token передаётся в Header: Bearer
* Создание отзыва к произведению. Необходим токен.  
    `POST http://127.0.0.1:8000/api/v1/titles/{title_id}/reviews/`
    ```json
    {
        "text": "string",
        "score": 1
    }
    ```
* Пример ответа:
    ```json
    {
        "id": 0,
        "text": "string",
        "author": "string",
        "score": 1,
        "pub_date": "2019-08-24T14:15:22Z"
    }
    ```
