# Проект "Kittygram"
![example workflow](https://github.com/NatalyaKozarezenko/kittygram_final/actions/workflows/main.yml/badge.svg)

## Описание проекта

Kittygram — социальная сеть для обмена фотографиями любимых питомцев.

Для неё настроен запуск в контейнерах и автоматическое тестирование, а так же и деплой проекта на удалённый сервер.
Автоматизация настроена с помощью сервиса GitHub Actions.

При пуше в ветку main:

1. проект тестируется,

2. при успешном прохождении тестов образы обновляются на Docker Hub,

3. на сервере запускаются контейнеры из обновлённых образов.

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
2.1. В корне проекта создайте файл .env и заполните следующими данными:

```
POSTGRES_DB=kittygram                               - имя базы данных
POSTGRES_USER=kittygram_user                        - имя пользователя в БД
POSTGRES_PASSWORD=kittygram_password                - пароль для пользователя к БД
DB_HOST=db                                          - имя Хоста
DB_PORT=5432                                        - порт соединения к БД
SECRET_KEY=SECRET_KEY                               - SECRET_KEY
ALLOWED_HOSTS='127.0.0.1 localhost kittygram.ru'    - перечень разрешённых хостов (пример)
SQLITE = False                                      - False для работы с postgresql и True для sqlite.
DEBUG = True                                        - статус режима отладки  
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

Файл workflow: kittygram/.github/workflows/main.yml

```
DOCKER_USERNAME - логин Docker Hub
DOCKER_PASSWORD - пароль Docker Hub
HOST            - IP-адрес вашего сервера
SSH_KEY         - содержимое текстового файла с закрытым SSH-ключом
SSH_PASSPHRASE  - пароль для ssh-ключа
TELEGRAM_TO     - ID вашего телеграм-аккаунта. Узнать ID можно у телеграм-бота @userinfobot. 
TELEGRAM_TOKEN  - токен вашего бота. Получить токен можно у телеграм-бота @BotFather.
USER            - имя пользователя
```

## Автор
[Наталья Козарезенко](https://github.com/NatalyaKozarezenko/) 
