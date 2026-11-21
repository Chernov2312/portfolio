# Chat with model — веб-приложение для общения с GigaChat

Веб-приложение с frontend на React/Vite и API на FastAPI. PostgreSQL хранит пользователей, чаты и историю сообщений; ответы модели передаются потоком через Server-Sent Events.

## Возможности

- Регистрация и вход с Bearer-токеном.
- Создание и удаление чатов, просмотр списка сессий пользователя.
- Потоковая генерация ответов и сохранение истории сообщений.
- Системный промпт для сессии, профиль пользователя и обновление email.
- Используется настроенная модель GigaChat `GigaChat-3-Ultra`; выбор модели из каталога пока не реализован.

## Запуск

Требуются Docker Desktop с Docker Compose и действующие GigaChat credentials.

1. Создайте `.env` на основе `.env.example`.
2. Укажите `GIGA_KEY`, `CLIENT_ID`, случайный `SECRET_KEY` и параметры PostgreSQL. Не публикуйте `.env` или рабочие credentials.
3. Из корня проекта запустите:

   ```powershell
   docker compose up --build
   ```

Compose запускает PostgreSQL, API и frontend; backend создаёт отсутствующие таблицы при старте.

| Сервис | Адрес |
| --- | --- |
| Frontend | <http://localhost:5173> |
| API и Swagger UI | <http://localhost:8000/docs> |
| Проверка API | <http://localhost:8000/health> |

Для локальной разработки frontend можно запустить отдельно из папки `frontend` командами `npm install` и `npm run dev`. Адрес API задаётся через `VITE_API_URL`.

## Технологии

React, Vite, FastAPI, PostgreSQL, GigaChat, Server-Sent Events, Docker Compose.
