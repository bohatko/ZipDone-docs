# ZipDone — ведение базы знаний

## Канонический источник

`https://github.com/bohatko/ZipDone-docs` — единственный канонический репозиторий общей документации.

Obsidian используется как редактор: откройте локальный checkout `ZipDone-docs` как Vault. OneDrive Vault и `docs/` приложений считаются зеркалами на период миграции, а не независимыми источниками.

## Что обновлять

| Изменение | Документы |
|---|---|
| Таблицы, RPC, RLS, triggers, cron | `database-schema.md`, `api-contracts.md` |
| Auth/API/Realtime/Storage | `api-contracts.md`, `storage-and-rls.md` и app doc |
| Статусы, matching, финансы | `architecture.md` и app docs |
| Экран/маршрут только одного приложения | соответствующий `apps/*.md` |
| Push payload/event | `notifications.md` |
| Новая тема | новый Markdown + `mkdocs.yml` nav |

## База данных

После каждой production-миграции:

1. Убедиться, что работа идёт только с project ref `nlpswsajjexnaqpwyiph`.
2. Получить verbose live schema.
3. Запустить `ZipDone-web/scripts/build-database-schema-doc.mjs` с `SCHEMA_DUMP_PATH`.
4. Проверить migration count, functions, triggers, cron, buckets.
5. Проверить diff всех зеркал.
6. Обновить narrative API/business sections, если контракт изменился.

Структуру таблиц в `database-schema.md` вручную не редактировать.

## Pull request checklist

- Документ описывает реализованное состояние, а планы явно помечены.
- Ссылки и MkDocs navigation валидны.
- Нет паролей, JWT, service role, private keys и webhook secrets.
- Примеры не содержат production user IDs или персональных данных.
- EN-имена API/slugs совпадают с кодом.
- `mkdocs build --strict` проходит.

## Версионирование

Существенные контракты связываются с migration version. Breaking change требует:

1. backward-compatible rollout либо согласованной версии;
2. обновления всех потребителей;
3. описания migration/rollback;
4. удаления старого контракта только после обновления приложений.
