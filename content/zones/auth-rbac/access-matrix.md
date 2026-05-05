# Auth-RBAC · Access Matrix

Матрица доступа по эндпоинтам admin-bff для каждой роли. Дополнение к `permissions-matrix.md` (исходный backlog).

## Роли стенда

| Роль | Permissions count | Scope |
|------|--------------------|-------|
| Администратор (admin@erp.local) | 50+ | `all_franchise` |
| Менеджер ТТ (ivan@test.local) | 41 | 3 ТТ |
| Курьер (petr@test.local) | 4 | 1 ТТ (Арбат) |
| Кассир (anna@test.local) | (admin-login отказан) | через PIN на POS |

Курьер permissions: `customers.create_quick`, `customers.read`, `orders.read`, `pos.access`.

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F11 | ⚪ | Manager scope содержит ТТ другого ЮЛ — by-design |
| F12 | ⚪ | Cashier admin-login отказан — by-design |
| F13 | 🔴 | RBAC read-leak: 7 эндпоинтов доступны без permissions у Курьера |
| F14 | 🔴 | Refunds доступны Курьеру без `pos.refund` (в составе F13) |
| F15 | 🔴 | HR-эндпоинты → 500 для всех ролей (4 шт) |
| BUG-013 | 🔴 | Manager → Юр.лица 403 (по спеке: read по permission) |
| BUG-014 | 🔴 | Manager → ТТ 403 (по спеке: своя ТТ) |
| BUG-015 | 🔴 | Создание ТТ → 403 для Franchisee |
| BUG-016 | 🔴 | Manager → Склад 403 (на самом деле 404 — F16) |
| BUG-017 | 🔴 | Manager → Заказы 403 |
| BUG-049 | 🔴 | Стоп-листы → 403 у Manager |

## Матрица read-доступа Курьер vs Manager (session 2026-05-05)

✅ корректно 403 (Курьер) / 200 (Manager с permission):

| Endpoint | Курьер ожидание | Курьер факт | Manager ожидание | Manager факт |
|----------|-----------------|-------------|-------------------|---------------|
| `GET /stores` | 403 | 403 ✅ | 200 | 200 ✅ |
| `GET /legal-entities` | 403 | 403 ✅ | 403 | 403 ✅ |
| `GET /employees` | 403 | 403 ✅ | 200 | 200 ✅ |
| `GET /catalog/products` | 403 | 403 ✅ | 200 | 200 ✅ |
| `GET /catalog/categories` | 403 | 403 ✅ | 200 | 200 ✅ |
| `GET /catalog/price-lists` | 403 | 403 ✅ | 200 | 200 ✅ |
| `GET /catalog/menu-availabilities` | 403 | 403 ✅ | 200 | 200 ✅ |
| `GET /modifier-groups` | 403 | 403 ✅ | 200 | 200 ✅ |
| `GET /orders?store_id=Арбат` | 200 (своя ТТ) | 200 ✅ | 200 | 200 ✅ |
| `GET /orders?store_id=Бауманская` | 403 (не в scope) | 403 ✅ | 200 (в scope) | 200 ✅ |

❌ **F13 leak — Курьер видит без permissions:**

| Endpoint | Permission которого нет | Курьер факт |
|----------|--------------------------|---------------|
| `GET /catalog/kitchen-stations` | `kds.access` | **200 (4 шт)** |
| `GET /kds/devices` | `kds.access` | **200 (8 шт)** |
| `GET /kds/settings` | `kds.access` / `kds.settings.edit` | **200** |
| `GET /pos/devices` | `pos.settings.edit` | **200 (13 шт)** |
| `GET /kitchen-queue?store_ids=X` | `kds.access` | **200 — финансовая чувствительность** |
| `GET /refunds` | `pos.refund` | **200 (4 шт)** |
| `GET /external-menus` | `menu.read` | **200** |

❌ **F15 — 500 вместо 403/200** (для всех ролей):

| Endpoint | Что должно | Что происходит |
|----------|------------|------------------|
| `GET /payroll` | 200 (с perm) / 403 (без) | **500 INTERNAL_ERROR** |
| `GET /shift-templates` | 200 / 403 | **500** |
| `GET /shift-records` | 200 / 403 | **500** |
| `GET /schedules` | 200 / 403 | **500** |

✅ **Mutation RBAC работает корректно:**

| Запрос | Permission нужен | Результат под Manager |
|--------|------------------|------------------------|
| PATCH /catalog/products | menu.edit (Manager НЕТ) | 403 «No permission to edit catalog» ✅ |
| POST /catalog/categories | menu.edit | 403 ✅ |
| DELETE /catalog/products | menu.edit | 403 ✅ |
| PATCH /kds/settings | kds.settings.edit (Manager ЕСТЬ) | 200 ✅ |

## Тест-кейсы

### TC-RBAC-080 — Cashier ТТ-A POST orders ТТ-B → 403
**Status:** ⏳ partial (нужна Cashier-PIN-сессия) · **TC-CHAIN-080**

### TC-RBAC-081 — Cashier scope-фильтр на orders
**Status:** ✅ pass · **TC-CHAIN-081**
Курьер petr@ → GET /orders без фильтра → видит только свою ТТ (Арбат).

### TC-RBAC-082 — KDS-оператор видит только свою ТТ
**Status:** ◯ todo (требует разделения KDS-PIN на ТТ) · **TC-CHAIN-082**

### TC-RBAC-083 — Manager POST /orders/{id}/complete
**Status:** ◯ todo · **TC-CHAIN-083**

### TC-RBAC-084 — Manager mutation без permission → 403
**Status:** ✅ pass · **TC-CHAIN-084**

### TC-RBAC-085 — Cashier cancel чужого заказа → 403
**Status:** ◯ todo · **TC-CHAIN-085**

### TC-RBAC-086 — Franchisee видит ТТ другого ЮЛ → 403
**Status:** ⛔ blocked (нет creds Franchisee) · **TC-CHAIN-086**

### TC-RBAC-087 — PIN без `pos.access` → отказ
**Status:** ◯ todo · **TC-CHAIN-087**

### TC-RBAC-088 — Cashier admin-login → отказ
**Status:** ✅ pass by-design (F12) · **TC-CHAIN-088**
Anna admin-login → INVALID_CREDENTIALS

### TC-RBAC-013 — RBAC read-leak регрессия (после фикса F13)
**Status:** verify-needed
Прогнать таблицу 7 эндпоинтов под Курьером → все 403.

## Snippets

```bash
# Логин Курьером (пароль — см. private/creds.md)
PETR=$(curl -sS -X POST $B/auth/login -H "Content-Type: application/json" \
  -d "{\"email\":\"petr@test.local\",\"password\":\"$PETR_PASS\"}" \
  | node -e "let s='';process.stdin.on('data',d=>s+=d);process.stdin.on('end',()=>console.log(JSON.parse(s).data.access_token))")

# Регресс F13
for ep in catalog/kitchen-stations kds/devices kds/settings pos/devices "kitchen-queue?store_ids=fe4b54a9-2cc1-458f-9d0e-338bbc51df76" refunds external-menus; do
  printf "%s => %s\n" "$ep" "$(curl -sS -o /dev/null -w '%{http_code}' "$B/$ep" -H "Authorization: Bearer $PETR")"
done
# После фикса F13 — все должны быть 403
```
