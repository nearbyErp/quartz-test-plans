# Catalog · Products

CRUD товаров, флаги, привязка к станциям, доступность по ТТ.

## Endpoints

```
GET    /catalog/products              список (per_page=100)
GET    /catalog/products/{id}         детали (НЕ возвращает цену — F6b)
POST   /catalog/products              create
PATCH  /catalog/products/{id}         update (silent no-op для base_price/modifier_group_ids)
DELETE /catalog/products/{id}         soft-delete (500 если store_ids=[] — F9)
```

См. также [reference/endpoints.md](../../reference/endpoints.md) — там полный список.

## Findings в этой зоне

| ID | Severity | Title |
|----|----------|-------|
| F6a | 🔴 | PATCH product тихо игнорирует `base_price` |
| F6b | 🟡 | GET product не возвращает текущую цену |
| F8 | 🔴 | PATCH с `available_in_all_stores: false + store_ids: [X]` теряет store_ids → товар недоступен везде |
| F9 | 🔴 | DELETE → 500 для товара в состоянии F8 |
| F10 | 🟡 | GET /catalog/categories/{id} → 404 (несимметричный CRUD) |
| F35 | 🟢 | mutex `is_open_price + is_by_weight` отсутствует |
| F41 | 🟡 | PATCH с `modifier_group_ids` тихо игнорируется → POST /products/{id}/modifiers |
| BUG-031 | 🟡 | UI «Доступно во всех точках» — после Save галочка возвращается (UI-симптом F8) |
| BUG-032 | 🟡 | Фильтр «Ингредиент» в Товарах — пусто |
| BUG-033 | 🟡 | Изменение товара не создаёт новую версию (по ADR-011) |
| BUG-035 | 🟡 | Смена категории на «Без категории» — Save возвращает исходную |
| BUG-036 | 🟡 | Дублирование товара не переносит фото, модификаторы, техкарту |

## Тест-кейсы

### TC-CATALOG-PROD-001 — Создание товара (CRUD basic)
**Status:** ✅ pass · **Last run:** [2026-05-05](../../sessions/2026-05-05-e2e.md) · **Linked:** TC-CHAIN-001
**Steps:** POST /catalog/products с минимальным валидным body → 201, возвращает id и все дефолты
**Expected:** Все 40+ полей в response заполнены дефолтами, status=active, deleted_at=null

### TC-CATALOG-PROD-002 — Rename
**Status:** ✅ pass · **TC-CHAIN-002**
PATCH /products/{id} с `{name: "X"}` → 200, name обновлён, updated_at сдвинут

### TC-CATALOG-PROD-003 — Изменение цены через product API
**Status:** ❌ fail (F6a) · **TC-CHAIN-003**
PATCH с `{base_price: 100}` → 200 OK, но цена не меняется. Правильный путь — `PATCH /catalog/price-lists/{id}/items`

### TC-CATALOG-PROD-004 — Soft-delete
**Status:** ⚠ conditional · **TC-CHAIN-004**
DELETE на нормальном товаре → 204. На товаре в состоянии F8 → 500 (F9).

### TC-CATALOG-PROD-005 — Restore soft-deleted
**Status:** ⛔ blocked F9 · **TC-CHAIN-005**

### TC-CATALOG-PROD-006 — Toggle `available_in_all_stores`
**Status:** ❌ fail (F8) · **TC-CHAIN-006**
Отправить `{available_in_all_stores: false, store_ids: [X]}` → 200, но `store_ids = []` (потеряны)

### TC-CATALOG-PROD-007 — Флаг `is_admin_only`
**Status:** ⏳ admin-side pass · **TC-CHAIN-007**
Создать с `is_admin_only: true` → response эхо true. POS-side: товар не виден Cashier (TBD).

### TC-CATALOG-PROD-008 — Флаг `is_open_price`
**Status:** ⏳ admin-side pass · **TC-CHAIN-008**

### TC-CATALOG-PROD-009 — Флаг `is_by_weight`
**Status:** ⏳ admin-side pass · **TC-CHAIN-009**

### TC-CATALOG-PROD-010 — Флаги `is_alcohol`, `is_tobacco`
**Status:** ⏳ admin-side pass · **TC-CHAIN-010**
F35: mutex `is_open_price + is_by_weight` не enforce

### TC-CATALOG-PROD-073 — Soft-delete товара после создания заказа
**Status:** ◯ todo · **TC-CHAIN-073**
Создать заказ → DELETE товар → проверить что заказ остаётся видимым и оплачиваемым

## Snippets

```bash
# Создать тестовый товар
echo '{"name":"test","category_id":"cd608646-2495-46b8-8067-7886bd2c9b11","type":"dish","unit_of_measure":"шт","vat_rate":"vat20","requires_kitchen":false}' > /tmp/p.json
curl -sS -X POST "$B/catalog/products" -H "$H" -H "Content-Type: application/json" --data-binary @/tmp/p.json
```
