# ZipDone — единая база знаний

> Обновлено: 2026-07-11 21:52 (UTC+3)  
> Production: Supabase **Zipdone** (`nlpswsajjexnaqpwyiph`, `eu-west-1`)  
> Сайт: https://bohatko.github.io/ZipDone-docs/

Этот репозиторий — канонический источник общих знаний для четырёх пользовательских зон:

| Зона | Платформа | Роль | Код |
|---|---|---|---|
| Super Admin | Web | `admin` | `ZipDone-web` |
| Company Panel | Web | `company` | `ZipDone-web` |
| Zipdone.Client | Flutter | `client` | `ZipDone-Flutter-client` |
| Zipdone.Worker | Flutter | `worker` | `ZipDone-Flutter-woker` |

## Быстрая навигация

- [Архитектура экосистемы](architecture.md)
- [API и Supabase-контракты](api-contracts.md)
- [Live-схема production БД](database-schema.md)
- [Storage и RLS](storage-and-rls.md)
- [Уведомления и push](notifications.md)
- [Web](apps/web.md)
- [Client](apps/mobile-client.md)
- [Worker](apps/mobile-worker.md)

## Правила источников

1. Общие контракты и бизнес-правила изменяются здесь одновременно с кодом.
2. `database-schema.md` генерируется из live DB после миграций.
3. Локальные `docs/` приложений — временные зеркала или специфичная для приложения документация.
4. Пароли, service role, Stripe/Firebase secrets и приватные ключи запрещены в Git.
5. Имена таблиц, RPC, Edge Functions, slug и маршруты сохраняются на английском.

## Текущее состояние

- 62 public-таблицы, RLS включён на всех.
- 89 применённых production-миграций.
- 102 функции (`public`: 77, `private`: 25).
- 33 public-триггера, 7 cron-задач, 6 Storage buckets.
- 15 активных Edge Functions.
- Общие языки интерфейса: EN/AR; валюта: AED; бизнес-таймзона: `Asia/Dubai`.
