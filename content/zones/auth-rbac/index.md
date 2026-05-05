# Auth-RBAC

Авторизация, роли, permissions, scope, проверка доступа.

## Структура зоны

| Файл | Что |
|------|-----|
| [login.md](login.md) | Email/password логин в admin-bff, JWT, refresh, logout |
| [pin-flow.md](pin-flow.md) | PIN-логин на POS/KDS, X-Device-Id headers, manager-approval flow |
| [permissions-matrix.md](permissions-matrix.md) | **Спека-матрица:** формальная 4×13 ролей × разделов по BR 1.4.4 (regression suite) |
| [access-matrix.md](access-matrix.md) | **Факт-матрица:** реальные результаты e2e сессий (RBAC leak F13, HR 500 F15, mutation-проверки) |

## Когда использовать какой файл

- **`permissions-matrix.md`** — для **регрессии после фикса RBAC**. Прогнать всю матрицу, заполнить колонки «факт», сравнить со «спекой». Покрытие — все 13 разделов админки × 4 роли.
- **`access-matrix.md`** — для понимания **что мы реально нашли** в e2e сессиях. Конкретные эндпоинты, конкретные роли, конкретные расхождения.

После полного прогона regression-suite (`permissions-matrix.md`) — обновить `access-matrix.md` свежими наблюдениями.

## Учётки на стенде

См. [reference/stand.md](../../reference/stand.md):
- `admin@erp.local` — Franchise owner (50+ perms)
- `ivan@test.local` — Manager ТТ (41 perm, scope = 3 ТТ)
- `petr@test.local` — Курьер (4 perms, scope = Арбат)
- `anna@test.local` — Cashier (admin-login отказан by-design)
- **Нет на стенде:** Franchisee owner (нужны для TC-CHAIN-086)

## Известные RBAC-баги (краткая сводка)

См. [findings.md](../../findings.md). Главные:
- 🔴 F13 — read-leak 7 эндпоинтов у Курьера
- 🔴 F14 — refunds доступны без `pos.refund` (часть F13)
- 🔴 F15 — HR 500-ки (4 endpoints) у всех ролей
- 🔴 BUG-013..017 — Manager 403 на разделы (по спеке должно быть доступно)
- 🔴 BUG-049 — Стоп-листы 403 у Manager
- 🔴 BUG-015 — Franchisee 403 на создание ТТ
