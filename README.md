# Лабораторная работа № 2. Проектирование и реализация клиент-серверной системы. HTTP, веб-серверы и RESTful веб-сервисы

## Цель работы:
Изучить методы отправки и анализа HTTP-запросов с
использованием инструментов telnet и curl, освоить базовую настройку и
анализ работы HTTP-сервера nginx в качестве веб-сервера и обратного
прокси, а также изучить и применить на практике концепции архитектурного
стиля REST для создания веб-сервисов (API) на языке Python.

## Оборудование и программное обеспечение:
- Операционная система: Ubuntu 20.04.6 LTS
(в рамках предоставленного образа).
- Сетевые утилиты: telnet, curl.
- Веб-сервер: nginx.
- Среда разработки:
   - Интерпретатор Python 3.8+.
   - Система управления пакетами python3-pip.
   - Инструмент для создания виртуальных окружений python3-
venv.
   - Микрофреймворк Flask для реализации REST API.
- Доступ к сети Интернет.

## Архитектуры решений:

**1) Запрос данных о
недвижимости с cian.ru и
анализ ответа.**

<img width="771" height="201" alt="image" src="https://github.com/user-attachments/assets/94beab71-6295-4220-acb8-f36c1a0a49c2" />



**2) API для "Спортивные
команды" (сущность: id,
team_name, sport_type).**

<img width="818" height="216" alt="image" src="https://github.com/user-attachments/assets/bf14f7f3-c5c6-4e75-ab3a-87ab2826871d" />



**3) Настроить кеширование
GET-запросов на 30 секунд.**

<img width="921" height="161" alt="image" src="https://github.com/user-attachments/assets/bbfba070-91a4-4bcf-b330-3f4d3e61a674" />

## Ход работы:

#### Этап 1: Подготовка рабочей среды

Была создана рабочая директория проекта для организации файлов и изоляции зависимостей:

```
mkdir ~/sports_teams_api
cd ~/sports_teams_api
```
<img width="629" height="82" alt="image" src="https://github.com/user-attachments/assets/8e9493f1-625f-40e2-8151-477fc9ae4d69" />


#### Этап 2: Установка программного обеспечения

Выполнена установка необходимых системных компонентов:

```
python3 -m venv venv
source venv/bin/activate
sudo apt install telnet curl nginx python3-venv -y
```
<img width="933" height="916" alt="image" src="https://github.com/user-attachments/assets/d3428c5a-29db-4668-b173-7a7dc08d83f6" />

#### Этап 3: Исследование веб-ресурса

Проведен анализ работы портала недвижимости cian.ru:

```
curl -v https://cian.ru
```
<img width="944" height="807" alt="image" src="https://github.com/user-attachments/assets/03c971f7-9126-4f86-a819-28ed308fd517" />

В результате анализа установлено использование современных протоколов безопасности и технологий кеширования.

#### Этап 4: Развертывание веб-сервера

Выполнена установка и настройка веб-сервера Nginx:

```
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

<img width="867" height="318" alt="image" src="https://github.com/user-attachments/assets/251949a5-ff57-43ad-bd03-b73d8661a8f3" />

#### Этап 5: Разработка REST API

Установлен веб-фреймворк Flask:

```
pip install Flask
```

<img width="935" height="476" alt="image" src="https://github.com/user-attachments/assets/61c7d08a-b1ec-47ef-acf7-84487a618966" />

Реализовано приложение **teams_api.py** со следующей структурой:

<img width="1515" height="621" alt="image" src="https://github.com/user-attachments/assets/533cce7c-f698-4ae4-9c35-1b21a47e7ec2" />

#### Этап 6: Тестирование функциональности API
Запущено приложение и проведено первичное тестирование:

```
python3 teams_api.py
curl -s http://localhost:5000/api/teams | jq
```

<img width="929" height="402" alt="image" src="https://github.com/user-attachments/assets/216513f9-afc9-4625-81b2-ba21591911ee" />

<img width="1423" height="882" alt="image" src="https://github.com/user-attachments/assets/3c2c5140-c452-47c4-818b-eb8a2ca582c8" />

#### Этап 7: Конфигурация Nginx с кешированием
Создана конфигурация прокси-сервера с поддержкой кеширования:
Настройка зоны кеширования в основном конфигурационном файле:

```
sudo nano /etc/nginx/nginx.conf
```

Добавлено в блок http { }:

```
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=sports_cache:10m inactive=60m;
```
<img width="495" height="776" alt="image" src="https://github.com/user-attachments/assets/50e90109-8dd9-4a00-9737-db8998882a9d" />


Создание конфигурации виртуального хоста:

```
sudo nano /etc/nginx/sites-available/sports_api
```

```
server {
    listen 80 default_server;
    server_name _;

    location / {
        root /var/www/html;
        index index.html index.htm;
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Конфигурация кеширования
        proxy_cache sports_cache;
        proxy_cache_valid 200 30s;
        add_header X-Cache-Status $upstream_cache_status;
        proxy_cache_methods GET;
    }
}
```
<img width="676" height="413" alt="image" src="https://github.com/user-attachments/assets/5fa0e392-1fff-4faa-84c2-80971b647990" />

#### Этап 8: Активация системы
Активирована финальная конфигурация:

```
sudo nano /etc/nginx/sites-available/sports_api
sudo ln -s /etc/nginx/sites-available/sports_api /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```
<img width="696" height="230" alt="image" src="https://github.com/user-attachments/assets/120527d0-8c8a-432c-803f-2d2cca3774e6" />

Результат проверки статуса Nginx:

<img width="973" height="870" alt="image" src="https://github.com/user-attachments/assets/e03e7266-a7c4-4d93-8b99-889bff396150" />

Веб-сервер Nginx успешно развернут, функционирует в штатном режиме и готов к обработке входящих запросов. Наличие процессов кеширования свидетельствует о корректной настройке механизма кеширования GET-запросов.

### Результаты тестирования:
#### Тест 1: Получение списка спортивных команд

```
curl -s http://localhost/api/teams | jq
```
<img width="601" height="264" alt="image" src="https://github.com/user-attachments/assets/a8add2d6-6c8c-4d6a-a9b7-07a28e315f0b" />

#### Тест 2: Получение команды по идентификатору

```
curl -s http://localhost/api/teams/1 | jq
```
<img width="618" height="102" alt="image" src="https://github.com/user-attachments/assets/46297041-0c51-4635-9750-e710f2071056" />

#### Тест 3: Создание новой спортивной команды
```
curl -X POST -H "Content-Type: application/json" \
-d '{"team_name": "Динамо", "sport_type": "Баскетбол"}' \
http://localhost/api/teams
```
<img width="692" height="140" alt="image" src="https://github.com/user-attachments/assets/54b25a1e-a748-422d-9f83-91a19fc74c37" />


#### Тест 4: Обновление спортивной команды по идентификатору
```
curl -X PUT -H "Content-Type: application/json" \
-d '{"team_name": "Спартак-М", "sport_type": "Футбол"}' \
http://localhost/api/teams/1
```
<img width="1087" height="213" alt="image" src="https://github.com/user-attachments/assets/c3f24808-10cd-4600-984d-a882162f9b00" />

#### Тест 5: Удаление спортивной команды по идентификатору
```
curl -X DELETE http://localhost/api/teams/2
```
<img width="1124" height="171" alt="image" src="https://github.com/user-attachments/assets/24a23c0a-e95a-46b3-bd7e-458ac0059d16" />

#### Тест 6: Верификация работы кеширования
```
curl -v http://localhost/api/teams
```

Первый запрос:

<img width="750" height="650" alt="image" src="https://github.com/user-attachments/assets/8cf50e4d-5438-4869-8a0e-1364c25d63c5" />

Второй запрос:

<img width="658" height="645" alt="image" src="https://github.com/user-attachments/assets/fd642551-35bc-41ec-9d74-0968ad4afb6c" />

Третий запрос:

<img width="607" height="537" alt="image" src="https://github.com/user-attachments/assets/14178596-e438-4c35-b4b9-0785119e1818" />


Статусы кеширования:

- MISS - данные не найдены в кеше, запрос обработан Flask-приложением
- HIT - данные получены из кеша Nginx, Flask-приложение не вызывалось
- EXPIRED - данные найдены в кеше, но их время жизни истекло (прошло более 30 секунд), запрос перенаправлен в Flask-приложение для обновления кеша

В результате тестирования кеширования подтверждена корректная работа механизма: первый запрос после очистки кеша возвращает статус MISS, последующие запросы в течение 30 секунд - статус HIT, что свидетельствует об эффективном использовании кеша Nginx. После истечения 30 секунд наблюдается статус EXPIRED, указывающий на необходимость обновления данных в кеше. При последующих запросах после EXPIRED статус снова меняется на HIT, подтверждая цикличность работы механизма кеширования.

## Вывод:
В ходе лабораторной работы успешно разработана клиент-серверная система, реализующая REST API для управления спортивными командами. Настроен веб-сервер Nginx в качестве обратного прокси с кешированием GET-запросов на 30 секунд, что подтверждено тестированием статусов MISS/HIT. Все компоненты системы функционируют корректно, обеспечивая эффективное взаимодействие между клиентом и сервером.
