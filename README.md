# OmniRoute: конфигурация развёртывания

Здесь только compose-файл. Данные, база и ключи в репозитории не хранятся.

## Что где
- Контейнер: `diegosouzapw/omniroute:latest`, порт `127.0.0.1:20128` (наружу через прокси/туннель)
- Данные: папка `omniroute-data` (в контейнере `/app/data`): `storage.sqlite`, `server.env`, `oauth/`, `call_logs/` и др.
- Бэкап хранится отдельно: `omniroute-data.tar.gz` (без базы) и `storage-live.sqlite` (снимок базы)

## Восстановление на новом сервере
1. Поставить Docker, скопировать на сервер этот репозиторий и бэкап.
2. Распаковать данные и вернуть базу:
   ```bash
   tar xzf omniroute-data.tar.gz            # создаст omniroute-data/
   cp storage-live.sqlite omniroute-data/storage.sqlite
   rm -f omniroute-data/storage.sqlite-wal omniroute-data/storage.sqlite-shm
   chown -R 1000:1000 omniroute-data        # контейнер работает от пользователя node (uid 1000)
   ```
3. `cp .env.example .env` и поправить `OMNIROUTE_DATA_DIR` при необходимости.
4. Запуск: `docker compose up -d`, проверка: `docker logs --tail 30 omniroute` (нужен статус healthy).

## Снимок базы без остановки контейнера
```bash
sqlite3 /root/omniroute-data/storage.sqlite ".backup '/root/storage-live.sqlite'"
```
Не копируй `storage.sqlite` простым `cp`/`tar` с работающего контейнера: часть данных лежит в WAL-файле.