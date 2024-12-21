# Финальный проект: контейнеры и CI/CD для Kittygram
![Static Badge](https://img.shields.io/badge/NatalyaKozarezenko-kittygram_final-kittygram_final)
![GitHub top language](https://img.shields.io/github/languages/top/NatalyaKozarezenko/Oxygen)
![GitHub](https://img.shields.io/github/license/NatalyaKozarezenko/kittygram_final)
![GitHub Repo stars](https://img.shields.io/github/stars/NatalyaKozarezenko/kittygram_final)
![GitHub issues](https://img.shields.io/github/issues/NatalyaKozarezenko/kittygram_final)

## Описание проекта

Автоматическое тестирование и деплой проекта Kittygram на удалённый сервер.

Kittygram — социальная сеть для обмена фотографиями любимых питомцев.

<!-- ??????????????????- полное описание, что это за проект, для чего, какие функции  -->

## Стек
Django
djangorestframework
djoser
webcolors
psycopg2-binary
Pillow
python-dotenv
pytest
PyYAML
gunicorn

## 1. Установка проекта
1.1. Клонируйте репозиторий и перейдите в него:

```
git clone https://github.com/NatalyaKozarezenko/kittygram_final.git

cd kittygram_final
```

1.2. Cоздайте и активируйте виртуальное окружение:

```
python3 -m venv env

source venv/bin/activate
```

1.3. По необходимости установите/обновите пакетный менеджер pip:

```
python3 -m pip install --upgrade pip
```

1.4. Установите зависимости из файла requirements.txt:

```
pip install -r requirements.txt
```

1.5. Выполните миграции:

```
python3 manage.py migrate
```

## 2. Настройка .env файла
В корне проекта создайте файл .env и заполните следующими данными:

```
POSTGRES_DB=kittygram                               #  Укажите имя базы данных

POSTGRES_USER=kittygram_user                        #  Укажите имя пользователя в БД

POSTGRES_PASSWORD=kittygram_password                #  Укажите пароль для пользователя к БД

DB_HOST=db                                          #  Укажите имя Хоста

DB_PORT=5432                                        #  Укажите порт соединения к БД

SECRET_KEY=SECRET_KEY                               #  Укажите SECRET_KEY

ALLOWED_HOSTS='127.0.0.1 localhost kittygram.ru'    #  Укажите перечень разрешённых хостов, например:

SQLITE = False                                      #  Укажите False для работы с postgresql и True для sqlite.
```

## 3. Создание Docker-образов
3.1. Замените username на ваш логин на DockerHub:

```
cd frontend
docker build -t username/kittygram_frontend .
cd ../backend
docker build -t username/kittygram_backend .
cd ../nginx
docker build -t username/kittygram_gateway .
```
 
3.2. Загрузите образы на DockerHub:

```
docker push username/kittygram_frontend
docker push username/kittygram_backend
docker push username/kittygram_gateway
```

## 4. Деплой проекта на сервер
4.1. Подключитесь к серверу:

```
ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера
```

4.2. Установите Docker Compose

```
sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt install docker-compose-plugin
```

4.3. Создайте на сервере директорию kittygram и скопируйте в неё файлы docker-compose.production.yml и .env

```
scp -i path_to_SSH/SSH_name docker-compose.production.yml \
    username@server_ip:/home/username/taski/docker-compose.production.yml
```

4.4. Запустите Docker Compose

```
sudo docker compose -f docker-compose.production.yml ps
```

4.5. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/

```
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```

4.6. Измените настройки location в секции server конфиг Nginx на нижеследующий

```
    location / {
        proxy_set_header Host $http_host;
        client_max_body_size 20M;
        proxy_pass http://127.0.0.1:8000;
    }
```
4.7. Перезапустите Nginx

```
sudo service nginx reload
```

4.8. Настройте секреты в GitHub Actions для работы workflow

```
DOCKER_USERNAME - логин Docker Hub
DOCKER_PASSWORD - пароль Docker Hub
HOST - IP-адрес вашего сервера
SSH_KEY - содержимое текстового файла с закрытым SSH-ключом
SSH_PASSPHRASE - пароль к серверу
TELEGRAM_TO - ID вашего телеграм-аккаунта. Узнать ID можно у телеграм-бота @userinfobot. 
TELEGRAM_TOKEN - токен вашего бота. Получить этот токен можно у телеграм-бота @BotFather.
USER - имя пользователя
```

## Автор
[Наталья Козарезенко](https://github.com/NatalyaKozarezenko/) 
