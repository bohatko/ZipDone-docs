# ZipDone — Storage (6 buckets + RLS)

> Источник: live Supabase `storage` (проект `nlpswsajjexnaqpwyiph`, eu-west-1)  
> Обновлено: 2026-07-11

## Buckets

| Bucket | Public | Лимит | MIME | Назначение | Path |
|---|---|---|---|---|---|
| `avatars` | да | 5 MB | jpeg, png, webp | фото профилей | `{auth.uid}/...` |
| `company-logos` | да | 5 MB | jpeg, png, webp, svg | логотипы компаний | `{company_id}/...` |
| `verification-docs` | нет | 10 MB | jpeg, png, pdf | документы верификации | `{company_id}/...` |
| `order-photos` | нет | 10 MB | jpeg, png, webp | фото до/после заказа | `{order_id}/...` |
| `dispute-attachments` | нет | 10 MB | jpeg, png, webp, pdf | вложения диспутов | `{dispute_id}/...` |
| `site-assets` | да | 5 MB | webp | изображения витрины (через `site_media`) | `{section}/...` |

## RLS (`storage.objects`)

Первый сегмент пути (`storage.foldername(name)[1]`) = идентификатор владельца. Все политики — для роли `authenticated`. Публичные buckets раздают объекты по public-URL (`getPublicUrl`) без RLS — для них SELECT-политика не нужна.

| Bucket | Read (SELECT) | Write (INSERT/UPDATE/DELETE) |
|---|---|---|
| `avatars` | владелец (`folder[1] = auth.uid()`) или admin | владелец |
| `company-logos` | владелец компании (`companies.owner_id`) или admin | владелец компании; admin (ALL) |
| `verification-docs` | владелец компании или admin | INSERT — владелец компании; DELETE — admin |
| `order-photos` | client / company / назначенный worker заказа / admin | INSERT — назначенный worker |
| `dispute-attachments` | участник диспута (client/company) или admin | INSERT — участник диспута или admin |
| `site-assets` | публичный URL (listing-политики нет) | INSERT/UPDATE/DELETE — admin |

### Политики (как в БД)

| Policy | Cmd | Bucket | Условие |
|---|---|---|---|
| `avatars_authenticated_read` | SELECT | avatars | `folder[1] = auth.uid()` OR `is_admin()` |
| `avatars_owner_insert` | INSERT | avatars | `folder[1] = auth.uid()` |
| `avatars_owner_update` | UPDATE | avatars | `folder[1] = auth.uid()` |
| `avatars_owner_delete` | DELETE | avatars | `folder[1] = auth.uid()` |
| `company_logos_authenticated_read` | SELECT | company-logos | `is_admin()` OR `companies.owner_id = auth.uid()` |
| `company_logos_owner_insert` | INSERT | company-logos | `companies.owner_id = auth.uid()` |
| `company_logos_owner_update` | UPDATE | company-logos | `companies.owner_id = auth.uid()` |
| `company_logos_owner_delete` | DELETE | company-logos | `companies.owner_id = auth.uid()` |
| `company_logos_admin_all` | ALL | company-logos | `is_admin()` |
| `verification_docs_company_select` | SELECT | verification-docs | `is_admin()` OR `companies.owner_id = auth.uid()` |
| `verification_docs_company_insert` | INSERT | verification-docs | `companies.owner_id = auth.uid()` |
| `verification_docs_admin_delete` | DELETE | verification-docs | `is_admin()` |
| `order_photos_participant_select` | SELECT | order-photos | admin / client / company / назначенный worker заказа |
| `order_photos_worker_insert` | INSERT | order-photos | назначенный worker (`order_workers` + `workers.profile_id = auth.uid()`) |
| `dispute_attachments_participant_select` | SELECT | dispute-attachments | admin / client диспута / company диспута |
| `dispute_attachments_participant_insert` | INSERT | dispute-attachments | admin / client диспута / company диспута |
| `site_assets_admin_insert` | INSERT | site-assets | `is_admin()` |
| `site_assets_admin_update` | UPDATE | site-assets | `is_admin()` |
| `site_assets_admin_delete` | DELETE | site-assets | `is_admin()` |

> Изменения 2026-06-13 (миграция `db_cleanup_audit`): добавлен bucket `site-assets`; удалена избыточная listing-политика `site_assets_public_read` на `storage.objects` (публичный доступ к объектам идёт по public-URL без RLS).

## Потребители

| Приложение | Buckets |
|---|---|
| Web (landing) | `site-assets` |
| Web (company) | `company-logos`, `verification-docs` |
| Client | `avatars`, `dispute-attachments` |
| Worker | `avatars`, `order-photos` |
| Admin | все buckets через admin policies |

Private buckets читаются через signed URL. Клиенты не формируют привилегированные URL самостоятельно.
