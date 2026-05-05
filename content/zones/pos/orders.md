# POS · Orders

POS-side создание заказов, отмена, выдача, history. Большая часть — desktop UI.

## Endpoints (POS BFF — недоступен снаружи)

```
POST /api/v1/pos/orders/
POST /api/v1/pos/orders/submit
GET  /api/v1/pos/orders/by-table/
GET  /api/v1/pos/orders/open
GET  /api/v1/pos/orders/recent-paid
POST /api/v1/pos/refunds/
GET  /api/v1/pos/refund-requests/
POST /api/v1/pos/shifts/open
POST /api/v1/pos/shifts/close
GET  /api/v1/pos/reports/shift-report
GET  /api/v1/pos/staff
```

Тестируется через POS desktop координацию + наблюдение через `admin/orders` и `admin/kitchen-queue`.

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F25 | 🔴 | Заказ исчезает из POS UI после оплаты |
| F27 | 🔴 | На POS нет UI кнопки cancel (API работает) |
| F37 | 🔴 | На POS нет журнала закрытых заказов смены |
| F42 | 🟢 | UX: при достижении max опций — нет подсказки |

## Тест-кейсы

### TC-POS-ORDERS-038 — Race: 2 POS одновременно создают заказы
**Status:** ⛔ blocked (нужен 2-й POS) · **TC-CHAIN-038**

### TC-POS-ORDERS-100 — Пустой заказ → попытка оплаты
**Status:** ◯ todo · **TC-CHAIN-100**

### TC-POS-ORDERS-101 — 50+ позиций
**Status:** ◯ todo · **TC-CHAIN-101**

### TC-POS-ORDERS-102 — Длинный комментарий
**Status:** ◯ todo · **TC-CHAIN-102**

### TC-POS-ORDERS-103 — Длинная строка без пробелов
**Status:** ◯ todo · **TC-CHAIN-103**

### TC-POS-ORDERS-104 — POS оффлайн → создание заказа
**Status:** ◯ todo · **TC-CHAIN-104**

### TC-POS-ORDERS-105 — Catalog Service down → меню
**Status:** ◯ todo · **TC-CHAIN-105**

### TC-POS-ORDERS-106 — Order Service down → создание
**Status:** ◯ todo · **TC-CHAIN-106**

### TC-POS-ORDERS-110 — qty=0 / отрицательное на штучном
**Status:** ◯ todo · **TC-CHAIN-110**

### TC-POS-ORDERS-112 — Часы POS уехали → создание
**Status:** ◯ todo · **TC-CHAIN-112**

### TC-POS-ORDERS-027 — UI cancel на любом статусе (F27)
**Status:** ❌ fail · session 2026-05-05
Backend cancel API работает. Кнопки в UI нет ни на one statuse (`new` / `accepted` / `ready`).

### TC-POS-ORDERS-025 — Закрытый заказ остаётся видимым (F25)
**Status:** ❌ fail · session 2026-05-05
После оплаты заказ полностью пропадает из всех вкладок POS. Backend `closed` со всеми деталями.

### TC-POS-ORDERS-037 — Журнал закрытых заказов смены (F37)
**Status:** ❌ fail · session 2026-05-05
В UI нет вкладки/таблицы закрытых заказов смены. Только агрегаты.
