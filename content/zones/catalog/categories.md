# Catalog · Categories

Категории товаров: иерархия, активность, порядок отображения, цвет, доступность по каналам.

## Endpoints

```
GET    /catalog/categories            список (с детализацией всех полей)
GET    /catalog/categories/{id}       404 — не реализован (F10)
POST   /catalog/categories            create
PATCH  /catalog/categories/{id}       update
DELETE /catalog/categories/{id}       204
```

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F10 | 🟡 | GET /catalog/categories/{id} → 404 (асимметрия CRUD) |
| BUG-039 | 🟡 | На странице категорий нельзя редактировать большинство полей через UI |
| BUG-040 | 🟡 | Активация parent НЕ каскадирует на children (только деактивация) |
| BUG-041 | 🟡 | 255 символов без пробелов ломают вёрстку, при редактировании — 409 |
| BUG-042 | 🟢 | Пустое название сохраняется без ошибки |

## Тест-кейсы

### TC-CATALOG-CAT-011 — Деактивировать категорию (каскад на дочерних)
**Status:** ⚠ inconclusive (F10) · **TC-CHAIN-011**
Создать parent → child → деактивировать parent → проверить child. Блокировано: GET /categories/{id} 404 — не подтвердить через API.
Workaround: проверять через `GET /catalog/categories` (списком) и фильтровать.

### TC-CATALOG-CAT-012 — Изменить порядок категорий
**Status:** ◯ todo · **TC-CHAIN-012**
PATCH category с `display_order` → проверить что list возвращает в новом порядке.

### TC-CATALOG-CAT-040 — Каскадная активация (BUG-040)
**Status:** ◯ todo
Подтверждение/опровержение того, что активация parent должна каскадировать на children.

### TC-CATALOG-CAT-042 — Валидация пустого имени (BUG-042)
**Status:** ◯ todo
POST с `{name: ""}` или без name — должно быть 400. Сейчас сохраняется.
