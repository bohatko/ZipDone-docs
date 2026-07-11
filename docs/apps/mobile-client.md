# ZipDone — мобильное приложение Client

> Репозиторий: `ZipDone-Flutter-client`  
> Package: `zipdone_client`, application ID: `com.zipdone.client`  
> Обновлено: 2026-07-11

## Стек

Flutter/Dart 3.6+, Riverpod 3, GoRouter 17, Supabase Flutter, Google Maps, Firebase Messaging, Flutter Stripe, EN/AR localization.

## Маршруты

| Route | Назначение |
|---|---|
| `/login`, `/auth/otp`, `/auth/invite/:token` | Phone OTP и invitation |
| `/auth/onboarding/profile` | Завершение профиля |
| `/home` | Адрес, промо, recent bookings |
| `/booking` | Upcoming/completed/cancelled |
| `/booking/new`, `/booking/new/recap` | Booking wizard и оплата |
| `/booking/:orderId` | Детали и lifecycle заказа |
| `/support`, `/support/:disputeId` | Диспуты |
| `/notifications` | Inbox |
| `/account/*` | Профиль, адреса, карты, настройки |

## Auth

Returning login проверяется через `check_auth_phone_exists` и `check_auth_client_phone`. Новый номер разрешён только по действующему client invite. После OTP trigger создаёт профиль/client; `accept_client_invitation` связывает клиента с пригласившей компанией.

## Booking

1. Выбор адреса, property type, числа комнат, клинеров, длительности и времени.
2. `calculate-price`.
3. INSERT `orders`.
4. `create-payment-intent` с выбранной сохранённой картой.
5. При необходимости Stripe PaymentSheet для customer action.
6. `sync-payment-status`.
7. `match-order`.
8. Realtime обновляет список и detail.

Статусы отображаются напрямую из общего order lifecycle. `reopen_order` возвращает completed-заказ в `in_progress` согласно backend rules.

## Платежи

- Карта сохраняется через SetupIntent/PaymentSheet.
- В `payment_methods` находятся только Stripe ID и безопасные metadata.
- Client имеет SELECT own methods; изменения выполняют Edge Functions.
- PaymentIntent авторизуется при booking и capture выполняется backend.
- PAN/CVC никогда не проходят через Supabase или код приложения.

## Диспуты

Client создаёт диспут через `create_order_dispute`, загружает вложения в `dispute-attachments/{dispute_id}` и регистрирует их через `add_dispute_attachments`. Private attachments читаются через signed URL.

## Уведомления

FCM реализован только на mobile. Device token регистрируется RPC. Payload с `target_app != client` игнорируется. Подробнее: [уведомления](../notifications.md).

## Реализовано не полностью

- Legal pages содержат placeholder-тексты.
- Некоторые категории и promo content остаются mock/MVP.
- Политика отмены `in_progress`, tips/rebook и отдельные действия detail требуют сверки с backend перед релизом.
- E2E push зависит от Firebase service account/APNs configuration.
