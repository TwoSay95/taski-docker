# Kittygram — социальная сеть для обмена фотографиями любимых питомцев. 

## Описание проекта

### Стек технологий

Проект написан на Python 3.10. с использованием библиотек:

    Django==3.2.3
    djangorestframework==3.12.4
    djoser==2.1.0
    webcolors==1.11.1
    Pillow==9.0.0
    pytest==6.2.4
    pytest-django==4.4.0
    pytest-pythonpath==0.7.3
    gunicorn==20.1
    python-dotenv==1.0.0

### Доступные методы
Для работы с пользователями использованы стандартные пути из библиотеки  
<a href="https://djoser.readthedocs.io/en/latest/sample_usage.html">Djoser</a>

Некоторые методы:

Получить всех котов:

        GET api/cats/
Добавить нового кота:

        POST api/cats/

С остальными методами можно ознакомиться в файле backend/kittygram_backend/urls.py и backend/cats/views.py

## Как развернуть проект

1. Склонировать репозиторий:

        git clone git@github.com:TwoSay95/infra_sprint1.git

2. Создать и активировать виртуальное окружение:

        python3 -m venv venv
        source venv/bin/activate

3. Установить зависимости для Python:

        pip install -r requirements.txt 

4. Перейти в папку backend и выполнить миграции:

        cd infra_sprint1/backend/
        python manage.py migrate

5. Создать суперюзера:

        python manage.py createsuperuser

6. В файле infra_sprint1/backend/kittygram_backend/settings.py в переменную ALLOWED_HOSTS добавить локальные адреса, 
   а также доменное имя или внешний IP (если есть):

        ALLOWED_HOSTS = ['127.0.0.1', '0.0.0.0', 'localhost']

7. В этом же файле поменять значение переменной DEBUG с True на False:

        DEBUG = False

8. Установить node.js:

        curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
        sudo apt-get install -y nodejs

9. Перейти в infra_sprint1/frontend и установить зависимости для frontend-приложения:

        npm i

10. Для backend-приложения установить WSGI-сервер gunicorn:

        pip install gunicorn

11. Создать файл конфигурации для автозапуска WSGi-сервера:

        sudo nano /etc/systemd/system/gunicorn.service

12. Внести в него следующие настройки:

        [Unit]
        Description=gunicorn daemon 
        After=network.target 

        [Service]
        User=<имя-пользователя-в-системе>  # (!) заменить на собственное
        WorkingDirectory=/home/<имя-пользователя>/infra_sprint1/backend/
        ExecStart=/home/<имя-пользователя>/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8000 backend.wsgi

        [Install]
        WantedBy=multi-user.target 

13. Запустить созданную службу и внести её в автозапуск:

        sudo systemctl start gunicorn
        sudo systemctl enable gunicorn

14. Установить веб-сервер и запустить его:

        sudo apt install nginx -y
        sudo systemctl start nginx 

15. Открыть порты Firewall и активировать его:

        sudo ufw allow 'Nginx Full'
        sudo ufw allow OpenSSH
        sudo ufw enable

16. Перейти в infra_sprint1/frontend/, собрать frontend-приложение и скопировать его в системную директорию веб-сервера 
    (необходимо для корректной работы статики):

        npm run build
        sudo cp -r <имя_пользователя>/infra_sprint1/frontend/build/. /var/www/infra_sprint1/

17. Перейти в infra_sprint1/backend/, собрать статику для админки Django и копировать его в системную директорию 
    веб-сервера (ВАЖНО! Необходимо изменить название папки со статикой во избежание конфликта имён):

        infra_sprint1/backend/kittygram_backend/settings.py поменять значение переменной STATIC_URL и добавить новую STATIC_ROOT

        STATIC_URL = 'static_backend'
        STATIC_ROOT = BASE_DIR / 'static_backend' 

        python manage.py collectstatic
        sudo cp -r infra_sprint1/backend/static_backend/ /var/www/infra_sprint1/

18. Создать папку для медиафайлов в директории веб-сервера, изменить права доступа:

        cd /var/www/infra_sprint1/
        mkdir media
        sudo chown -R <имя_пользователя> /var/www/infra_sprint1/media/

19. Перейти в файл конфигурации веб-сервера и изменить его настройки на следующие (для доступа нужны права root):

        sudo nano /etc/nginx/sites-enabled/default 

        server {

            server_name server_name <публичный-IP-адрес> <доменное-имя>;

            location /api/ {
                proxy_pass http://127.0.0.1:8000;
            }

            location /admin/ {
                proxy_pass http://127.0.0.1:8000;
            }

            location /media/ {
                alias /var/www/infra_sprint1/media/;
            }
        
            location / {
            root    /var/www/infra_sprint1;
            index   index.html index.htm;
            try_files  $uri /index.html;
            }
        
        }

20. Проверить файл конфигурации веб-сервера, перезагрузить его в случае успеха:

        sudo nginx -t
        sudo systemctl reload nginx

21. (Опционально) Получить SSL-сертификат для вашего доменного имени при помощи certbot:

        sudo apt install snapd
        sudo snap install core; sudo snap refresh core
        sudo snap install --classic certbot
        sudo ln -s /snap/bin/certbot /usr/bin/certbot
        sudo certbot --nginx

#### Для примера сайт доступен по ссылке https://kittyfree.linkpc.net/

## Автор
<a href="https://github.com/TwoSay95">Молотков Виктор</a>
