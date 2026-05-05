# Catalog · Stop-lists

Стоп-листы товаров и категорий per-store.

## Endpoints

```
GET    /catalog/stop-lists/stores/{store_id}                          стоп-лист ТТ ({products, categories})
POST   /catalog/stop-lists/stores/{store_id}/products                 добавить товар
                                                                       body: {product_id, reason?}
DELETE /catalog/stop-lists/stores/{store_id}/products/{product_id}    снять товар
POST   /catalog/stop-lists/stores/{store_id}/categories               добавить категорию
                                                                       body: {category_id, reason?}
DELETE /catalog/stop-lists/stores/{store_id}/categories/{category_id} снять категорию
```

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F39 | 🟢 | `reason: null` принимается без валидации |
| BUG-049 | 🔴 | Стоп-листы → 403 у Manager (по спеке: своя ТТ доступна) |

## Тест-кейсы

### TC-CATALOG-STOP-020 — Стоп товара
**Status:** ✅ pass · **TC-CHAIN-020**
1. POST product в стоп → 201
2. Refresh POS → товар не доступен для добавления

### TC-CATALOG-STOP-021 — Снятие со стопа
**Status:** ✅ pass · **TC-CHAIN-021**
DELETE → 204, товар снова виден на POS

### TC-CATALOG-STOP-022 — Стоп категории
**Status:** ⚠ backend pass, UI blocked (F40) · **TC-CHAIN-022**
POST category в стоп → 201. UI-проверка не получится для категорий с активным `menu-availability` (F40 их и так скрывает).

### TC-CATALOG-STOP-023 — Снятие категории со стопа
**Status:** ✅ backend pass · **TC-CHAIN-023**
DELETE → 204

### TC-CATALOG-STOP-039 — Валидация reason (F39)
**Status:** ❌ fail
POST без поля reason → 201 (ожидание 400 «reason обязателен»)

### TC-CATALOG-STOP-049 — Manager доступ к стопам своей ТТ (BUG-049)
**Status:** ◯ todo
Под Manager Ivan → GET stop-list своей ТТ → должно 200, по спеке.

## Snippets

```bash
# Стоп товара
SID="fe4b54a9-2cc1-458f-9d0e-338bbc51df76"
echo '{"product_id":"<id>","reason":"закончился"}' > /tmp/s.json
curl -sS -X POST "$B/catalog/stop-lists/stores/$SID/products" -H "$H" -H "Content-Type: application/json" --data-binary @/tmp/s.json

# Снять
curl -sS -X DELETE "$B/catalog/stop-lists/stores/$SID/products/<product_id>" -H "$H"
```
