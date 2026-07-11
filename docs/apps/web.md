# ZipDone — Web

> Репозиторий: `ZipDone-web`  
> Обновлено: 2026-07-11

## Стек

Vite 8, React 19, TypeScript 5.8, React Router 7, Tailwind CSS 4, Supabase JS, React Hook Form/Zod, Recharts, Leaflet.

## Зоны

| Prefix | Роль | Назначение |
|---|---|---|
| `/` | public | Landing EN/AR |
| `/privacy`, `/about` | public | Legal |
| `/auth/*` | guest | Общий auth/onboarding |
| `/admin/*` | `admin` | Super Admin |
| `/company/*` | `company` | Company Panel |

## Admin

Dashboard, Companies, Workers, Clients, Orders, Disputes, Notifications, Settings и Invitations. Admin управляет верификацией, жизненным циклом заказа, диспутами, platform settings и аудитом мэтчинга.

## Company

Dashboard, Orders, Zones, Teams, Clients, Finances, Disputes, Notifications и Account. Доступ требует approved verification. Stripe Connect gate направляет неподключённую компанию в Finances.

## Data access

- Browser использует только publishable/anon key.
- Запросы выполняются через PostgREST/RPC с RLS.
- Stripe и административная отмена выполняются Edge Functions.
- Realtime: offers, orders, notifications.

## Деплой

Vercel обслуживает Vite SPA. `vercel.json` направляет deep routes в `index.html`. Production env содержит только публичные Supabase client values; backend secrets находятся в Supabase.

## Технический долг

- `database.types.ts` остаётся неполной ручной заглушкой и должен генерироваться из production schema.
- Автотесты и CI для web отсутствуют.
- Старые неиспользуемые company pages следует удалить либо явно пометить как legacy.
