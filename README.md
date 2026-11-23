## Запуск

Скопируйте `.env.example` в `.env`. В `APP_PORT` задаётся внешний порт, в `GREETING` — приветствие. В примерах используется порт `8080`.

```bash
cp .env.example .env
docker compose up -d --build --wait --wait-timeout 60
curl -fsS http://127.0.0.1:8080/
```

Основной `compose.yaml` запускает приложение и Redis. Для запуска версии с SQLite остановите основной проект командой `docker compose down`, затем выполните `docker compose -f compose.sqlite.yaml up -d --build`.

## Результаты проверок

- **Один контейнер.** Собрал образ командой `docker build -t hw3-counter .` и запустил через `docker run -d --name hw3-app -p 127.0.0.1:8080:8000 -e GREETING="q qDocker!" hw3-counter`. Запросы `GET /`, `POST /increment`, `GET /` вернули `0 → 1 → 1`. Ответ содержал переданное приветствие. [Логи запуска](level4.log).
- **Compose и SQLite.** После `POST /increment` счётчик был равен `1`. После `docker compose -f compose.sqlite.yaml down` и повторного запуска `GET /` снова вернул `1`. Приложение сохранило базу в томе `counter-data`.
- **Compose и Redis.** После `docker compose stop redis` запросы `GET /healthz`, `GET /readyz` и `POST /increment` вернули соответственно `200`, `503` и `503`. После `docker compose start redis` приложение снова стало готово без ручного перезапуска. `GET /` вернул `2`, следующий `POST /increment` — `3`.
- **Сохранение данных Redis.** Выполнил `GET /`, затем `docker compose down` и `docker compose up -d --wait --wait-timeout 60`. Оба сервиса стали `healthy`. Повторный `GET /` показал `3`, как и до пересоздания контейнеров.