# Spark

Spark — веб-мессенджер с REST API и WebSocket-соединениями. Backend работает на FastAPI, PostgreSQL хранит данные, Redis передаёт события чатов и обслуживает очередь сообщений, Celery запускает фоновые задачи.

## Возможности

- Регистрация, подтверждение почты и вход по JWT.
- Личные и групповые чаты, профили, аватары и статусы набора текста.
- Сообщения с файлами, изображения в чате и реакции.
- Поиск по пользователям и создание групповых чатов.
- Опросы и викторины с настройкой анонимности, отмены голоса и показом правильного ответа.
- Форматирование сообщений, краткое содержание переписки и переписывание текста через Ollama с OpenAI-совместимым клиентом.
- Распознавание голосовых вложений через Whisper по запросу пользователя.
- Загрузка файлов в S3-совместимое хранилище Yandex Cloud.
- Аудио- и видеозвонки через WebRTC и LiveKit; Go-сервис сигналинга поддерживает гостевые ссылки.

## Требования

- Docker Desktop с Docker Compose.
- Ollama запускается в Docker и не требует внешнего API-ключа. Для реальной отправки почты и загрузки файлов укажите соответствующие значения в `.env`.
- Backend image использует Python 3.13 и FFmpeg для подготовки аудио к Whisper.

## Запуск через Docker Compose

Все команды выполняются из корня проекта.

1. Создайте конфигурацию из шаблона:

   ```powershell
   Copy-Item .env.example .env
   ```

   На macOS/Linux:

   ```bash
   cp .env.example .env
   ```

2. Для локальной разработки оставьте `API_BASE_URL=http://localhost:8001` и `FRONTEND_BASE_URL=http://localhost:5174` в `.env`. Первая настройка задаёт адрес API для браузера, вторая — адрес фронтенда в ссылках подтверждения почты. При необходимости измените `OLLAMA_MODEL` и `OLLAMA_BASE_URL`; по умолчанию используется `qwen3:0.6b` и адрес `http://ollama:11434`. Укажите реальные ключи Yandex Cloud и SMTP, если нужны эти интеграции. Тестовые заглушки из `.env.example` не подходят для отправки писем и работы с объектным хранилищем.

3. Соберите и запустите приложение:

   ```bash
   docker compose --profile dev up --build
   ```

   Compose запускает Ollama и загружает выбранную модель, затем запускает PostgreSQL, Redis, FastAPI, Celery worker и beat, а также frontend. Backend обращается к локальной модели через OpenAI-совместимый API и применяет миграции Alembic.

Откройте:

- Frontend: <http://localhost:5174>
- API: <http://localhost:8001>
- Swagger UI: <http://localhost:8001/docs>
- Health check: <http://localhost:8001/health>
- PostgreSQL: `localhost:5433`
- Redis: `localhost:6380`
- Ollama API: <http://localhost:11434>

Чтобы остановить сервисы и сохранить данные, выполните `docker compose --profile dev down`. Для полной очистки также удалите Docker volumes `spark_postgres_data` и `spark_ollama_data`, а также локальную папку `redisdata`.

## Whisper cache

Папка `./whisper-cache` подключена к backend и Celery worker как `/home/appuser/.cache/whisper`. Whisper скачивает модель при первом запросе на распознавание; последующие запуски используют тот же файл модели. Папка исключена из Git.

## API и WebSocket

Основные маршруты начинаются с `/api/v1`:

| Раздел | Маршрут |
| --- | --- |
| Аутентификация | `/auth` |
| Пользователи | `/users` |
| Чаты | `/chats` |
| Сообщения | `/chats/{chat_id}/messages`, `/messages/{message_id}` |
| Реакции | `/messages/{message_id}/reactions` |
| Вложения | `/attachments/` |
| ИИ | `/ai/chats/{chat_id}/summarize`, `/ai/messages/rewrite` |

Распознавание вложения: `POST /chats/{chat_id}/messages/{message_id}/attachments/{attachment_id}/transcribe`. Оно запускается только после нажатия кнопки распознавания в чате. WebSocket подключается по адресу `ws://localhost:8001/{chat_id}/ws?token=<JWT>`.

## Разработка и проверки

Backend-код находится в `backend/`, frontend — в `frontend/`. Frontend-команды запускаются из `frontend/`:

```bash
npm run dev
npm run build
npm run lint
npm test
```

Backend тесты запускаются в отдельном Docker Compose окружении:

```bash
docker compose --env-file .env.test -f docker-compose.test.yml up -d --build
docker compose --env-file .env.test -f docker-compose.test.yml exec backend-test pytest
docker compose --env-file .env.test -f docker-compose.test.yml down
```

Проверка Python-кода через Ruff выполняется из корня репозитория:

```bash
pip install -r backend/requirements/dev.txt
ruff check .
ruff format --check .
```

Чтобы автоматически отформатировать Python-файлы, выполните `ruff format .`.

Тестовое окружение поднимает отдельные PostgreSQL и Redis; для интеграционных тестов также требуется Docker для Testcontainers.
