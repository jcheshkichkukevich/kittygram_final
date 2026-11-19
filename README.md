# Kittygram

![Main Kittygram workflow](https://github.com/jcheshkichkukevich/kittygram_final/actions/workflows/main.yml/badge.svg)

## Описание

**Kittygram** — социальная сеть для владельцев котиков. Пользователи регистрируются, добавляют своих котов с фотографией, цветом, годом рождения и достижениями, а также смотрят котов других пользователей.

Проект запускается в Docker-контейнерах, а GitHub Actions при каждом пуше в ветку `main` автоматически проверяет код, запускает тесты, собирает Docker-образы, отправляет их на Docker Hub и присылает уведомление в Telegram.

### Технологии

- **Бэкенд:** Python 3.9, Django 3.2, Django REST Framework, Djoser, Gunicorn
- **Фронтенд:** React
- **База данных:** PostgreSQL 13
- **Инфраструктура:** Docker, Docker Compose, Nginx, GitHub Actions, Docker Hub

### Как устроен проект в контейнерах

| Контейнер | Образ | Что делает |
|---|---|---|
| `db` | `postgres:13` | База данных, хранит данные в volume `pg_data` |
| `backend` | `kittygram_backend` | Django + Gunicorn на порту 8000 |
| `frontend` | `kittygram_frontend` | Собирает React-приложение и копирует его в volume `static` |
| `gateway` | `kittygram_gateway` | Nginx: отдаёт фронтенд, статику и медиа, проксирует `/api/` и `/admin/` в бэкенд |

Volumes: `pg_data` — данные PostgreSQL, `static` — статика фронтенда и бэкенда, `media` — картинки, загруженные пользователями.

## Как запустить проект локально

1. Клонируйте репозиторий:

```bash
git clone https://github.com/jcheshkichkukevich/kittygram_final.git
cd kittygram_final
```

2. Создайте файл `.env` в корне проекта по образцу `.env.example`:

```env
POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=kittygram_password
DB_NAME=kittygram
DB_HOST=db
DB_PORT=5432
SECRET_KEY=ваш_секретный_ключ
DEBUG=False
ALLOWED_HOSTS=localhost,127.0.0.1
```

3. Соберите и запустите контейнеры:

```bash
docker compose up -d --build
```

4. Выполните миграции, соберите статику бэкенда и скопируйте её в volume:

```bash
docker compose exec backend python manage.py migrate
docker compose exec backend python manage.py collectstatic
docker compose exec backend cp -r /app/collected_static/. /backend_static/static/
```

5. При необходимости создайте суперпользователя:

```bash
docker compose exec backend python manage.py createsuperuser
```

Проект будет доступен по адресу http://localhost:9000/, админка — http://localhost:9000/admin/.

## Запуск из готовых образов Docker Hub

В `docker-compose.production.yml` вместо сборки (`build`) используются готовые образы с Docker Hub:

```bash
docker compose -f docker-compose.production.yml pull
docker compose -f docker-compose.production.yml up -d
```

Дальше — те же команды миграций и сбора статики, но с флагом `-f docker-compose.production.yml`.

## CI/CD

Workflow `.github/workflows/main.yml` при пуше в ветку `main`:

1. проверяет код бэкенда линтером **ruff** и запускает тесты бэкенда (с PostgreSQL);
2. запускает тесты фронтенда;
3. собирает образы `kittygram_backend`, `kittygram_frontend`, `kittygram_gateway` и отправляет их на Docker Hub;
4. отправляет сообщение в Telegram об успешном завершении.

Для работы workflow в настройках репозитория (Settings → Secrets and variables → Actions) нужно добавить секреты:

| Секрет | Значение |
|---|---|
| `DOCKER_USERNAME` | логин на Docker Hub |
| `DOCKER_PASSWORD` | пароль или access token Docker Hub |
| `TELEGRAM_TO` | ваш Telegram ID |
| `TELEGRAM_TOKEN` | токен Telegram-бота |

## Примеры запросов к API

Регистрация: `POST /api/users/`

```json
{
  "username": "kot",
  "password": "SuperCat_2026"
}
```

Получение токена: `POST /api/token/login/` → `{"auth_token": "..."}`

Список котов: `GET /api/cats/` с заголовком `Authorization: Token <токен>`.

## Автор

jcheshkichkukevich — [GitHub](https://github.com/jcheshkichkukevich)
