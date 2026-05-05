# Orders · Denormalization

Денормализация полей при создании позиции заказа: имя товара, цена, модификаторы, кухонная станция.

## Что денормализуется (подтверждённое)

| Поле | Где | Денормализация работает? |
|------|-----|--------------------------|
| `product_name` | items[].product_name | ✅ да (F70) |
| `unit_price` | items[].unit_price | ✅ да (F71) |
| `total_price` | items[].total_price | ✅ да |
| modifier `option_name`, `price` | items[].modifiers[] | ✅ да (после F44 fix flow) |
| `kitchen_station_id` | items[].kitchen_station_id | ✅ да (по умолчанию) |
| `assembly_time_seconds` | items[].assembly_time_seconds | ❌ **нет** — F21 |
| `kitchen_status` | items[].kitchen_status | живой статус, не денорм |

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F21 | 🟡 | `assembly_time_seconds` в позиции = null |

## Тест-кейсы

### TC-ORDERS-DENORM-070 — Имя товара
**Status:** ✅ pass · **TC-CHAIN-070** · session 2026-05-05 (Кола → ОРИГИНАЛ → revert, заказ #033)
1. Создать заказ с товаром
2. PATCH product name в каталоге
3. GET order → items[0].product_name = старое имя

### TC-ORDERS-DENORM-071 — Цена товара
**Status:** ✅ pass · **TC-CHAIN-071** · session 2026-05-05 (Кола 80 → 100 в price-list, заказ #034 vs #035)
1. Заказ с товаром, unit_price = X
2. PATCH /price-lists/{id}/items → новая цена Y
3. Существующий заказ → unit_price=X (frozen)
4. Новый заказ → unit_price=Y

### TC-ORDERS-DENORM-072 — Цена опции модификатора
**Status:** ✅ pass · **TC-CHAIN-072** · session 2026-05-05 (заказ #036 unit+0+0=50, заказ #037 unit+0+10=60)
PATCH /price-lists/{id}/modifier-items → новая цена опции применяется к новым заказам.

### TC-ORDERS-DENORM-073 — Soft-delete товара после создания заказа
**Status:** ◯ todo · **TC-CHAIN-073**

### TC-ORDERS-DENORM-074 — Удалить группу модификаторов после заказа
**Status:** ◯ todo · **TC-CHAIN-074**

### TC-ORDERS-DENORM-075 — Переименование категории — category_path
**Status:** ◯ todo · **TC-CHAIN-075**
Уточнить: денормализуется ли `category_path` в позиции или live?

### TC-ORDERS-DENORM-076 — Shift-report за день
**Status:** ⛔ blocked (F15 shift endpoints 500) · **TC-CHAIN-076**

### TC-ORDERS-DENORM-021 — assembly_time_seconds (F21)
**Status:** ❌ fail
1. Товар с `assembly_time: 240`
2. Создать заказ
3. GET order → items[0].assembly_time_seconds = null (ожидание 240)
