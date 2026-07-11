# ZipDone — безопасность документации

## Запрещено хранить в Git

- Supabase `service_role` и secret keys;
- Stripe secret/webhook keys;
- Firebase service-account JSON и APNs keys;
- access/refresh tokens;
- реальные пароли и фиксированные OTP production-пользователей;
- приватные Management API tokens;
- персональные данные клиентов и работников.

Publishable/anon keys технически публичны, но их не следует дублировать в документации: приложения получают их из env/config, а документация описывает только имя переменной.

## Где хранить

| Данные | Хранилище |
|---|---|
| Edge Function secrets | Supabase Secrets |
| CI secrets | GitHub Actions Secrets |
| Локальные ключи | ignored `.env.local` / `--dart-define-from-file` |
| Командные credentials | password manager |
| Firebase mobile config | app repository по принятой Firebase-модели; service account отдельно |

## Тестовые пользователи

Публичная документация может перечислять назначение ролей и процедуру создания seed-аккаунтов, но не общий пароль. Реальные тестовые credentials выдаются через password manager.

Существующий `Creads.md` в зеркалах содержит общий тестовый пароль и legacy anon JWT. Перед публикацией его необходимо удалить из общедоступной истории либо заменить на безопасный runbook, а скомпрометированные credentials — ротировать.

## Реакция на утечку

1. Считать secret скомпрометированным независимо от последующего удаления файла.
2. Немедленно ротировать credential у провайдера.
3. Удалить secret из текущей версии и, при необходимости, очистить Git history.
4. Проверить audit logs и активные сессии.
5. Зафиксировать incident без публикации самого секрета.
