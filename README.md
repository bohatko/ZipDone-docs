# ZipDone — единая база знаний

Каноническая документация экосистемы ZipDone: web-панели, мобильные приложения Client и Worker, Supabase и внешние интеграции.

## Работа с документацией

1. Клонируйте репозиторий.
2. Откройте корень репозитория как Obsidian Vault.
3. Редактируйте Markdown в `docs/`.
4. Перед merge проверьте сайт:

```bash
python -m pip install -r requirements.txt
mkdocs serve
mkdocs build --strict
```

После push в `main` GitHub Actions публикует сайт в GitHub Pages.

## Источники правды

- общая архитектура, бизнес-правила и API: этот репозиторий;
- структура production БД: `docs/database-schema.md`, только из live Supabase Zipdone;
- реализация приложений: соответствующие кодовые репозитории;
- секреты и реальные credentials: password manager / Supabase Secrets, не Git.

Supabase project: **Zipdone** (`nlpswsajjexnaqpwyiph`, `eu-west-1`).
