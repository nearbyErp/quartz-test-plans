# Orders · Lifecycle

Status-машина заказа, переходы, таймстампы, нумерация per-store per-day.

## Status-машина (подтверждённая F29)

```
new → accepted → ready → closed
                ↓
            cancelled (только из new/accepted, через API)
```

| Статус | accepted_at | kitchen_started_at | ready_at | paid_at | completed_at |
|--------|-------------|--------------------|---------|---------|--------------|
| `new` | null | null | null | null | null |
| `accepted` | ✓ | ✓ | null | null | null |
| `ready` | ✓ | ✓ | ✓ | null | null |
| `closed` | ✓ | ✓ | ✓ | ✓ | ✓ |
| Заказ только из не-кухонных | null | null | ✓ (мгновенно) | по оплате | по выдаче |

**Per-position:** items имеют свой `kitchen_status` + `kitchen_started_at` + `kitchen_ready_at`. Order переходит в `ready` когда **все** kitchen-items имеют `kitchen_status: ready` (F29 confirmed, F28 уточнение для не-кухонных).

## Endpoints

```
GET    /orders                          список (фильтры: store_id, status, date)
GET    /orders/{id}                     полные детали
POST   /orders/{id}/cancel              отмена (body обязателен: {reason})
GET    /kitchen-queue?store_ids=X       кухонная очередь
```

POS-side (через pos-bff, недоступен снаружи):
```
POST /api/v1/pos/orders/, /submit
GET  /api/v1/pos/orders/by-table/, /open, /recent-paid
```

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F19 | ⚪ | Быстрые заказы: `created_at == accepted_at == kitchen_started_at` (observation) |
| F20 | ⚪ | KDS одна кнопка «Готово», без двух фаз start/finish |
| F21 | 🟡 | `assembly_time_seconds` в позиции = null (денорм не сработала) |
| F22 | ⚪ | Семантика `kitchen_started_at` отличается на order/item уровне (doc) |
| F24 | 🟡 | Пропуски в нумерации (#22, #25) — гипотеза: симптом F25/F37 |
| F25 | 🔴 | Заказ исчезает из POS UI после оплаты |
| F27 | 🔴 | На POS нет UI кнопки cancel (API работает) |
| F28 | ⚪ | Заказ только из не-кухонных → сразу ready |
| F37 | 🔴 | На POS нет журнала закрытых заказов смены |
| BUG-064 | 🔴 | Назначение официанта на стол → 404 |
| BUG-065 | 🔴 | Перемещение стола → белая страница |

## Тест-кейсы

### TC-ORDERS-LIFE-050 — new → accepted (in_progress)
**Status:** ⚪ N/A by-design (F19) · **TC-CHAIN-050**
На быстрых заказах переход мгновенный. Если есть случай где accepted_at появляется отдельно — задокументировать.

### TC-ORDERS-LIFE-051 — accepted → ready (KDS «Готово»)
**Status:** ✅ pass (заказ #026) · **TC-CHAIN-051**
Latency 32 сек на тест-стенде.

### TC-ORDERS-LIFE-052 — ready → closed (оплата + выдача)
**Status:** ⚠ backend pass, UI fail F25/F26 · **TC-CHAIN-052**

### TC-ORDERS-LIFE-053 — Cancel в new
**Status:** ✅ backend (API) · **TC-CHAIN-053**
POST `/orders/{id}/cancel` с `{reason}` → 200 status=cancelled.
F27: UI кнопки нет ни на каком статусе.

### TC-ORDERS-LIFE-054 — Cancel в accepted
**Status:** ◯ todo · **TC-CHAIN-054**

### TC-ORDERS-LIFE-055 — Cancel closed → отказ
**Status:** ◯ todo · **TC-CHAIN-055**

### TC-ORDERS-LIFE-056 — Add позицию в accepted → отказ
**Status:** ◯ todo · **TC-CHAIN-056**

### TC-ORDERS-LIFE-057 — Откат «Готово»
**Status:** ⚪ N/A by-design (F20)

### TC-ORDERS-LIFE-058 — Per-position статусы
**Status:** ✅ pass (заказ #030 — Кола+Шаурма) · **TC-CHAIN-058**

### TC-ORDERS-LIFE-059 — Заказ ready только когда все позиции ready
**Status:** ✅ pass (F29) · **TC-CHAIN-059**

### TC-ORDERS-LIFE-060 — Race: 2 KDS жмут «Готово»
**Status:** ◯ todo · **TC-CHAIN-060**

### TC-ORDERS-LIFE-061 — Per-day numbering на полночь
**Status:** ◯ todo · **TC-CHAIN-061**

### TC-ORDERS-LIFE-024-INVESTIGATE — Пропуски нумерации (F24)
**Status:** verify-needed
SQL: `SELECT id, status, deleted_at, created_at FROM orders WHERE store_id=... AND order_number IN ('022','025')`
Если записи есть с заполненным `deleted_at` или непредвиденным `status` — это симптом F25/F37.

## Snippets

```bash
# Получить заказ полный
curl -sS "$B/orders/<order_id>" -H "$H"

# Отменить заказ
echo '{"reason":"тест"}' > /tmp/c.json
curl -sS -X POST "$B/orders/<order_id>/cancel" -H "$H" -H "Content-Type: application/json" --data-binary @/tmp/c.json

# Кухонная очередь
curl -sS "$B/kitchen-queue?store_ids=fe4b54a9-2cc1-458f-9d0e-338bbc51df76" -H "$H"
```
