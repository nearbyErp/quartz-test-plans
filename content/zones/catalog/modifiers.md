# Catalog · Modifiers

Группы модификаторов и опции. Привязка к товарам. Min/max валидация.

## Endpoints

```
GET    /modifier-groups                          список
GET    /modifier-groups/{id}                     детали (price у options НЕ возвращается)
POST   /modifier-groups                          create (silent no-op для options[].price → F44)
PATCH  /modifier-groups/{id}                     update
DELETE /modifier-groups/{id}                     204 (НЕ чистит modifier_items в price-list → F45)

POST   /catalog/products/{id}/modifiers          привязать группу к товару
                                                 body: {modifier_group_id, binding_type:"free"|"structural", override_min_amount?, override_max_amount?}
PATCH  /catalog/price-lists/{id}/modifier-items  обновить цены опций
                                                 body: {items:[{modifier_option_id, price}]}
```

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F30 | 🟡 | `max_amount < min_amount` принимается (BUG-044 confirmed) |
| F31 | 🟡 | Отрицательный `min_amount` принимается |
| F32 | 🟡 | Опция с отрицательной ценой принимается (потенциальный обход скидок) |
| F33 | 🟢 | Нет верхнего предела для `max_amount` (999999 принимается) |
| F34 | 🟢 | Группа без опций (`options: []`) принимается |
| F41 | 🟡 | PATCH product с `modifier_group_ids` не работает — нужен POST /products/{id}/modifiers |
| F42 | 🟢 | UX: при достижении max — нет подсказки на POS |
| F44 | 🟡 | POST `/modifier-groups` тихо игнорирует `options[].price` |
| F45 | 🟢 | DELETE modifier-group оставляет orphan записи в price-list/modifier-items |
| BUG-043 | 🟢 | UI: Описание в шрифте textarea |
| BUG-044 | 🟡 | (covered by F30/F31/F32) |
| BUG-045 | 🟢 | Имя опции: нет maxlength, принимает пробел |
| BUG-047 | 🟡 | Технология приготовления синхронизирована между опциями |
| BUG-048 | 🟡 | Не сохраняется удаление текста из «Технология приготовления» |

## Тест-кейсы

### TC-CATALOG-MOD-013 — Привязка группы к товару
**Status:** ✅ pass (через `POST /products/{id}/modifiers`) · **TC-CHAIN-013**
Связан с F41 — PATCH product с `modifier_group_ids` не работает.

### TC-CATALOG-MOD-014 — Min/max enforce на POS
**Status:** ✅ pass UX (max не даёт превысить) · **TC-CHAIN-014**
F42 — UX без подсказки. F30/F31/F32 — backend без валидации.

### TC-CATALOG-MOD-015 — Удалить группу — позиция в существующем заказе остаётся
**Status:** ◯ todo · **TC-CHAIN-015**
Создать заказ → DELETE modifier-group → открыть заказ — модификаторы должны остаться (denorm).

### TC-CATALOG-MOD-016 — Изменить цену опции
**Status:** ✅ pass · **TC-CHAIN-016**
PATCH `/price-lists/{id}/modifier-items` с `{items: [{modifier_option_id, price}]}` → 200, новая цена применяется к новым заказам.

### TC-CATALOG-MOD-030 — Валидация `max < min` (F30 / BUG-044)
**Status:** ❌ fail · **session 2026-05-05**
POST с `min_amount=3, max_amount=1` → 201 (ожидание 400)

### TC-CATALOG-MOD-031 — Отрицательный min (F31)
POST с `min_amount=-1` → 201

### TC-CATALOG-MOD-032 — Отрицательная цена опции (F32)
POST с `options:[{name:"x", price:-100}]` → 201

### TC-CATALOG-MOD-033 — Огромный max (F33)
POST с `max_amount=999999` → 201

### TC-CATALOG-MOD-034 — Группа без опций (F34)
POST с `options:[]` → 201

### TC-CATALOG-MOD-045 — DELETE не чистит price-list (F45)
**Status:** ❌ fail (session 2026-05-05)
1. POST modifier-group
2. DELETE
3. GET /price-lists/{id}/items → modifier_items этой группы остались

## Snippets

```bash
# Создать группу с опциями
echo '{"name":"test","type":"group","min_amount":1,"max_amount":2,"options":[{"name":"A"},{"name":"B"},{"name":"C"}]}' > /tmp/m.json
curl -sS -X POST "$B/modifier-groups" -H "$H" -H "Content-Type: application/json" --data-binary @/tmp/m.json

# Установить цену опции (правильный endpoint)
echo '{"items":[{"modifier_option_id":"<id>","price":10}]}' > /tmp/p.json
curl -sS -X PATCH "$B/catalog/price-lists/<pl_id>/modifier-items" -H "$H" -H "Content-Type: application/json" --data-binary @/tmp/p.json

# Привязать к товару
echo '{"modifier_group_id":"<group_id>","binding_type":"free"}' > /tmp/b.json
curl -sS -X POST "$B/catalog/products/<product_id>/modifiers" -H "$H" -H "Content-Type: application/json" --data-binary @/tmp/b.json
```
