# Short URL

Сервис был задеплоен на VPS сервер, и доступен по адресу [https://45.88.76.128/](https://45.88.76.128/docs) (ругается на сертификаты безопасности, потому что они самописные через openssl).  

## Описание API и примеры запросов

Примечение: документацию API со всеми запросами можно найти по ссылке (лучше использовать его): [https://45.88.76.128/docs](https://45.88.76.128/docs).

Ниже представлено описание эндпоинтов с примерами запросов. Дополнительным называет эндпоинт, который реализован сверх основных требований к API.

1. **POST `/links/shorten`** - создает короткую ссылку с возможностью задания кастомного алиаса и времени жизни. Кэширование не используется. Рабоатает, как для авторизированных пользовалетей, так и нет.

Пример: 
```
curl -k -X 'POST' \
  'https://45.88.76.128/links/shorten' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "original_url": "yahoo.com",
  "custom_alias": "yahoo",
  "expires_at": "2025-04-19 20:07:06"
}'
```

2. **GET `/links/search`** - поиск ссылки по оригинальному URL. Кэширование на 10 секунд, сбрасывается при изменении/удалении ссылки.

Пример: 
```
curl -k -X 'GET' \
  'https://45.88.76.128/links/search?original_url=yahoo.com' \
  -H 'accept: application/json'
```

3. **GET `/links/all`** (дополнительный) - возвращает список всех активных ссылок. Требует авторизации. Кэширование на 60 секунд.

Пример: 
```
curl -k -X 'GET' \
  'https://45.88.76.128/links/all' \
  -H 'accept: application/json'
```

4. **GET `/links/my-statistics`** (дополнительный) - экспорт статистики ссылок пользователя, под которым авторизованы, в CSV. Кэширование не используется.

Пример: 
```
curl -k -X 'GET' \
  'https://45.88.76.128/links/my-statistics' \
  -H 'accept: */*'
```


5. **GET `/links/{short_code}`** - перенаправляет на оригинальный URL. Кэширование на 5 минут (счетчик переходов обновляется в фоне).

Пример: 
```
curl -k -X 'GET' \
  'https://45.88.76.128/links/yahoo' \
  -H 'accept: application/json'
```



6. **DELETE `/links/{short_code}`** - удаляет ссылку (позволяет это сделать авторизированным пользовалям и только для своих ссылок). Кэширование не используется, сбрасывает кэш для этой ссылки.

Пример: 
```
curl -X 'DELETE' \
  'https://45.88.76.128/links/string' \
  -H 'accept: */*'
```

7. **PUT `/links/{short_code}`** - обновляет длинный URL или время жизни ссылки (позволяет это сделать авторизированным пользовалям и только для своих ссылок). Кэширование не используется, сбрасывает кэш для этой ссылки.

Пример: 
```
curl -k -X 'PUT' \
  'https://45.88.76.128/links/google' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "original_url": "google.com",
  "expires_at": "2025-04-19 20:07:06"
}'
```

8. **GET `/links/{short_code}/stats`** - предоставляет статистику по ссылке. Кэширование на 5 секунд, сбрасывается при изменении/удалении ссылки.

Пример: 
```
curl -k -X 'GET' \
  'https://45.88.76.128/links/yahoo/stats' \
  -H 'accept: application/json'
```


9. **GET `/admin/cache-keys`** (дополнительный) - отображает все ключи в Redis. Доступно только администраторам. Кэширование не используется.

Пример: 
```
curl -k -X 'GET' \
  'https://45.88.76.128/admin/cache-keys' \
  -H 'accept: application/json'
```

10. **POST `/auth/jwt/login`** - аутентификация пользователя через JWT. Кэширование не используется. По умолчанию есть такой пользователь, для авторизации под ним зайти под `admin:admin`.

Пример: 
```
curl -k -X 'POST' \
  'https://45.88.76.128/auth/jwt/login' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password&username=admin&password=admin&scope=&client_id=string&client_secret=string'
```

11. **POST `/auth/jwt/logout`** - выход из системы. Кэширование не используется.

Пример: 
```
curl -k -X 'POST' \
  'https://45.88.76.128/auth/jwt/logout' \
  -H 'accept: application/json' \
  -d ''
```

12. **POST `/auth/register`** - регистрация нового пользователя. Кэширование не используется.

Пример: 
```
curl -k -X 'POST' \
  'https://45.88.76.128/auth/register' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "email": "user_name",
  "password": "pass_word",
  "is_active": true,
  "is_superuser": false,
  "is_verified": false
}'
```

13. **GET `/health`** (дополнительный) - проверка работоспособности сервиса. Кэширование не используется.

Пример: 
```
curl -k -X 'GET' \
  'https://45.88.76.128/health' \
  -H 'accept: application/json'
```

### Дополнительные функции

Также помимо этих запросов была реализована дополнительная функция (которая работает в фоне, через Celery,  по расписанию, каждые 10 секунд), что если ссылка просрочилась (прошел срок `expited_at`) или не использовалась `LINK_TTL_IN_DAYS` дней с момента создания / обновления , то такая ссылка удаляется.

## Инструкция по запуску

Для запуска выполните следующие шаги:

1. Клонировать репозиторий
2. В корневой папке проекта создать `.env` файл со следующим содержанием (секреты нужно изменить):

```
DB_HOST = 'postgres'
DB_PASS = 'changeme'
DB_PORT = '5432'
DB_NAME = 'ShortUrlDB'
DB_USER = 'changeme'
JWT_SECRET_KEY = 'changeme'
PASSWORD_SECRET_KEY = 'changeme'
MESSAGE_BROKER_URL = 'redis://redis:6379'
LINK_TTL_IN_DAYS = 5
CODE_GENERATION_ATTEMPTS = 5
CODE_GENERATION_SECRET = 'changeme'
SHORT_CODE_LENGTH = '9'
SITE_IP = 'localhost'
```

3. В корневой папке проекта вызывать `docker-compose build && docker-compose up -d`.
4. Подождать запуска. Сервис будет доступен по адресу: [http://localhost:8000](http://localhost:8000).


## Описание БД

#### 1. Таблица `alembic_version`
Служебная таблица для управления миграциями через Alembic.

| Колонка         | Тип               | Описание                     |
|-----------------|--------------------|------------------------------|
| `version_num`   | `VARCHAR`         | Номер версии миграции        |

---

#### 2. Таблица `links`
Хранит информацию о сокращенных ссылках.

| Колонка             | Тип                     | Описание                               |
|---------------------|--------------------------|----------------------------------------|
| `id`                | `INTEGER` (PK)          | Уникальный идентификатор ссылки        |
| `short_code`        | `VARCHAR`               | Короткий код ссылки (уникальный)       |
| `long_url`          | `VARCHAR`               | Оригинальный URL (уникальный)                       |
| `redirect_counter`  | `INTEGER`               | Количество переходов по ссылке         |
| `author_id`         | `INTEGER` (FK → `user`) | ID создателя (связь с таблицей `user`) |
| `created_at`        | `TIMESTAMP`             | Дата и время создания ссылки           |
| `updated_at`        | `TIMESTAMP`             | Дата и время последнего обновления     |
| `expires_at`        | `TIMESTAMP`             | Дата и время истечения срока действия  |
| `last_used_at`      | `TIMESTAMP`             | Дата и время последнего использования  |

---

#### 3. Таблица `user`
Хранит данные пользователей.

| Колонка             | Тип               | Описание                                   |
|---------------------|--------------------|--------------------------------------------|
| `id`                | `INTEGER` (PK)    | Уникальный идентификатор пользователя     |
| `email`             | `VARCHAR`         | Электронная почта (уникальная)            |
| `hashed_password`   | `VARCHAR`         | Хэшированный пароль                       |
| `registered_at`     | `TIMESTAMP`       | Дата и время регистрации                  |
| `is_active`         | `BOOLEAN`         | Активен ли аккаунт       |
| `is_superuser`      | `BOOLEAN`         | Является ли администратором               |
| `is_verified`       | `BOOLEAN`         | Подтвержден ли email                      |



### Хостинг

Как уже было сказано ранее, сервис был развернут не на хостинге, а не выделенном VPS сервере <sub>(тут можно и о доп. баллах подумать...)</sub>. Для этого во время сборки `docker compose` разворачивается nginx сервер (конфигурацию которого можно найти в файле `nginx.conf`).
Сервис работает через https, поэтому для работы были сгенерированы сертификаты (через openssl. Через certbot не получилось сгенерировать так как у меня нет свободного domain name для получения сертификатов).

На момент написания данного текста сервис находится в полностью рабочем состоянии. Если на момент проверки он, по каким-то причинам, будет недоступен, прошу написать [мне](https://t.me/nikitalotts), и я постараюсь оперативно его поднять.
