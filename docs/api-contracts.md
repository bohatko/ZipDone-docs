# ZipDone — API и Supabase-контракты

> Обновлено: 2026-07-11  
> Контракт относится только к Supabase `nlpswsajjexnaqpwyiph`.

## Правила доступа

- Public/mobile/web используют publishable/anon key и JWT пользователя.
- Авторизация определяется `profiles.role_slug` и RLS, а не `user_metadata`.
- `service_role`, Stripe secret, Firebase service account и webhook secrets доступны только backend.
- Клиент не должен считать успешным UPDATE/DELETE без проверки результата: RLS может вернуть ноль строк.

## Auth и onboarding

| RPC | Вызывающая сторона | Результат |
|---|---|---|
| `get_invitation_by_token` | guest | Проверка invite, target type и срока |
| `check_auth_phone_exists` | guest | Существует ли phone identity |
| `check_auth_client_phone` | Client | Разрешён ли returning client |
| `check_auth_worker_phone` | Worker | Разрешён ли returning worker |
| `accept_client_invitation` | Client | Привязка клиента к компании |
| `accept_company_invitation` | Company/Worker | Завершение invite onboarding |
| `prepare_client_auth_email_change` | Client | Подготовка смены seed email |
| `prepare_worker_auth_email_change` | Worker | Подготовка смены seed email |

Новый номер Client/Worker без активного invite не допускается. OTP: WhatsApp primary, SMS fallback. Канонический формат телефона — E.164.

## Edge Functions

Production содержит 15 активных функций.

| Function | JWT | Назначение |
|---|---:|---|
| `calculate-price` | да | Расчёт subtotal, fee и total по platform settings |
| `match-order` | да | Запуск/продолжение мэтчинга |
| `stripe-connect` | да | Stripe Express onboarding компании |
| `stripe-webhook` | нет | Stripe webhook с проверкой подписи |
| `create-payment-intent` | да | Авторизация оплаты заказа |
| `capture-payment` | да | Capture PaymentIntent |
| `send-push` | нет | Внутренний webhook доставки FCM |
| `create-setup-intent` | да | SetupIntent для сохранения карты |
| `confirm-setup-intent` | да | Сохранение metadata PaymentMethod |
| `manage-payment-method` | да | Delete / set default |
| `sync-payment-status` | да | Синхронизация PaymentIntent → `payments` |
| `stripe-config` | да | Возврат publishable Stripe config |
| `create-setup-checkout-session` | да | Web Checkout для сохранения карты |
| `confirm-setup-checkout-session` | да | Подтверждение Checkout Session |
| `admin-force-cancel` | да | Admin cancel/refund с проверкой роли |

`stripe-webhook` и `send-push` имеют `verify_jwt=false`, поэтому обязаны проверять собственную подпись/secret. Полные таблицы и сигнатуры RPC находятся в [схеме БД](database-schema.md).

## Admin

| RPC / Edge | Назначение |
|---|---|
| `get_matching_exhausted_order_ids(order_ids[])` | Заказ в `matching` без активного pending-оффера и без новых кандидатов с `working_day` + `working_hours` |
| `admin_force_cancel_order` | Принудительная отмена с refund/cancel PaymentIntent |
| `admin_create_order_dispute` | Создание диспута от имени платформы |
| `resolve_dispute` | Решение диспута и финансовые эффекты |
| `admin-force-cancel` (Edge) | Stripe cancel/refund + RPC `admin_force_cancel_order` |

Миграции: `find_matching_companies_team_capacity`, `admin_matching_exhausted`.

## Заказы

### Client

| Операция | Контракт |
|---|---|
| Цена | `calculate-price` |
| Создание | INSERT `orders` для собственного `client_id` |
| Авторизация оплаты | `create-payment-intent` |
| Проверка оплаты | `sync-payment-status` |
| Мэтчинг | `match-order` |
| Список/детали | SELECT own `orders` и разрешённые joins |
| Отмена | UPDATE own order в допустимом статусе |
| Оценка | UPSERT `order_ratings` для завершённого own order |
| Повторное открытие | `reopen_order` |
| Диспут | `create_order_dispute`, `add_dispute_attachments` |

### Company

| RPC/канал | Назначение |
|---|---|
| `accept_order_company_offer` | Принять активный оффер |
| `reject_order_company_offer` | Отклонить оффер |
| `assign_order_team` | Назначить команду подходящей ёмкости |
| `complete_order_manually` | Завершить исполнение |
| Realtime `order_company_offers` | Новый оффер компании |
| Realtime `orders` | Обновление Kanban и деталей |

### Worker

Worker получает только назначенные заказы через `order_workers`/RLS. Фото загружаются в `order-photos/{order_id}/...` и регистрируются в `order_photos` с `photo_type=before|after`.

## Статусы

| Домен | Значения |
|---|---|
| Order | `matching`, `assigned`, `in_progress`, `completed`, `cancelled` |
| Profile | `invited`, `active`, `suspended`, `deactivated` |
| Employment | `active`, `on_leave`, `terminated` |
| Dispute | `open`, `resolved` |
| Offer | `pending`, `accepted`, `rejected`, `expired`, `skipped` |
| Payment | `pending`, `authorized`, `captured`, `refunded`, `partially_refunded`, `failed` |

## Realtime

| Таблица | Filter | Потребители |
|---|---|---|
| `notifications` | `profile_id=eq.<auth.uid>` | Все приложения |
| `orders` | RLS + app query | Client, Company, Admin |
| `order_company_offers` | Company RLS | Company |

После reconnect приложение выполняет refetch: Realtime является сигналом инвалидации, а не единственным источником состояния.

## Storage

| Bucket | Доступ | Path contract |
|---|---|---|
| `avatars` | Owner/admin | `{profile_id}/...` |
| `company-logos` | Company owner/admin; public delivery | `{company_id}/...` |
| `verification-docs` | Company owner/admin | `{company_id}/...` |
| `order-photos` | Order participants | `{order_id}/...` |
| `dispute-attachments` | Dispute participants/admin | `{dispute_id}/...` |
| `site-assets` | Public read, admin write | `{section}/...` |

Private buckets выдаются через signed URL. Клиенты не формируют привилегированные URL самостоятельно.

### Worker (planned — backend ready, app stub)

| Операция | Контракт |
|---|---|
| Список назначений | SELECT `orders` через `order_workers`/RLS |
| Start / complete | RPC `start_order_work`, `complete_order_manually` |
| Фото | Storage `order-photos` + INSERT `order_photos` |
| Realtime | `orders`, `order_status_events` (не подключено в app) |

## Client RPC (Flutter, факт)

| RPC | Назначение |
|---|---|
| `get_invitation_by_token`, `check_auth_phone_exists`, `check_auth_client_phone` | Auth gate |
| `accept_client_invitation`, `prepare_client_auth_email_change`, `check_auth_email_exists` | Onboarding / email |
| `create_order_dispute`, `add_dispute_attachments` | Disputes |
| `reopen_order` | Reopen completed order |
| `mark_notification_read`, `mark_all_notifications_read` | Inbox |
| `upsert_device_token`, `remove_device_token` | Push |

## Worker RPC (Flutter, факт)

| RPC | Назначение |
|---|---|
| `get_invitation_by_token`, `check_auth_phone_exists`, `check_auth_worker_phone` | Auth gate |
| `accept_company_invitation`, `prepare_worker_auth_email_change`, `check_auth_email_exists` | Onboarding / email |
| `mark_notification_read`, `mark_all_notifications_read` | Inbox |
| `upsert_device_token`, `remove_device_token` | Push |

## Edge Functions — source of truth

| Function | Канонический репозиторий |
|---|---|
| `calculate-price`, `match-order`, `stripe-connect`, `admin-force-cancel` | `ZipDone-web` |
| Payment setup (`stripe-config`, `create-setup-intent`, …) | `ZipDone-Flutter-client/supabase/functions/` (дубли; сверять с web) |
| `send-push` | Триггерится БД; исходник в client repo |

## Drift registry

| Риск | Детали |
|---|---|
| Миграции | Канон: `ZipDone-web/supabase/migrations/` (68 local / 89 remote). Client repo: 6 локальных SQL. |
| Client cancel | Прямой UPDATE `orders`, не RPC |
| Client reopen | RPC + fallback UPDATE |
| Worker push routes | `/booking/:id` → `/booking` (detail route отсутствует) |
| Local notification prefs | Client/Worker: SharedPreferences, не backend |

Последняя верификация схемы: см. шапку [database-schema.md](database-schema.md).

## Ошибки

Приложения должны различать:

- `401`: отсутствующая/просроченная сессия — refresh или login;
- `403`: роль/RLS/бизнес-запрет — не повторять автоматически;
- `404`: сущность не существует или скрыта RLS;
- `409`: конфликт состояния/идемпотентности;
- `422`: бизнес-валидация;
- `429`/`5xx`: ограниченный retry с backoff, только для идемпотентных операций.
