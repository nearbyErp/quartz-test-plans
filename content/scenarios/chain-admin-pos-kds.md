# Сценарий: Chain Admin → POS → KDS

Сквозная цепочка: каталог/настройки во **франш-админке** → меню и оформление заказа на **POS** → отображение и прогон статусов на **KDS**. Используется как regression-сценарий после любого изменения в Catalog/Order Service, POS или KDS.

**Полная версия со всеми steps:** `archive/chain-admin-pos-kds-original.md`
**Этот файл:** progress-tracker + ссылки на zones по этапам.

## Прогресс

**Прогнано:** 21 / 60. **Последний прогон:** [2026-05-05-e2e](../sessions/2026-05-05-e2e.md)

## Этап 1 — Admin → POS пропагация (zones/catalog/*)

| TC | Описание | Status | Findings | Zone |
|----|----------|--------|----------|------|
| TC-CHAIN-001 | Создание товара | ✅ pass | — | [products](../zones/catalog/products.md) |
| TC-CHAIN-002 | PATCH name (rename) | ✅ pass | — | [products](../zones/catalog/products.md) |
| TC-CHAIN-003 | PATCH base_price | ❌ fail | F6a | [products](../zones/catalog/products.md) |
| TC-CHAIN-004 | DELETE soft-delete | ❌ fail | F9 | [products](../zones/catalog/products.md) |
| TC-CHAIN-005 | Restore soft-deleted | ⛔ skipped | (зависит от F9) | [products](../zones/catalog/products.md) |
| TC-CHAIN-006 | Toggle available_in_all_stores | ❌ fail | F8 | [products](../zones/catalog/products.md) |
| TC-CHAIN-007 | is_admin_only | ⏳ admin part pass | — | [products](../zones/catalog/products.md) |
| TC-CHAIN-008 | is_open_price | ⏳ admin part pass | — | [products](../zones/catalog/products.md) |
| TC-CHAIN-009 | is_by_weight | ⏳ admin part pass | — | [products](../zones/catalog/products.md) |
| TC-CHAIN-010 | is_alcohol / is_tobacco | ⏳ admin part pass | F35 (mutex) | [products](../zones/catalog/products.md) |
| TC-CHAIN-011 | Деактивировать категорию | ⚠ inconclusive | F10 (GET cat 404) | [categories](../zones/catalog/categories.md) |
| TC-CHAIN-012 | Изменить порядок категорий | ◯ todo | — | [categories](../zones/catalog/categories.md) |
| TC-CHAIN-013 | Привязка модификатора | ✅ pass | F41 (PATCH product не работает) | [modifiers](../zones/catalog/modifiers.md) |
| TC-CHAIN-014 | min/max enforce на POS | ✅ pass (UI) | F30, F31, F32, F33, F34 (backend), F42 (UX) | [modifiers](../zones/catalog/modifiers.md) |
| TC-CHAIN-015 | Удалить группу — позиция остаётся | ◯ todo | F45 (orphan) | [modifiers](../zones/catalog/modifiers.md) |
| TC-CHAIN-016 | Изменить цену опции | ✅ pass | F44 (POST mod groups не пишет) | [modifiers](../zones/catalog/modifiers.md) |
| TC-CHAIN-017 | Прейскурант с override | ⛔ blocked | BUG-051, F1 | [price-lists](../zones/catalog/price-lists.md) |
| TC-CHAIN-018 | Изменение цены → новые заказы | ✅ pass | — | [price-lists](../zones/catalog/price-lists.md) |
| TC-CHAIN-019 | Цена 0 | ◯ todo | — | [price-lists](../zones/catalog/price-lists.md) |
| TC-CHAIN-020 | Стоп товара | ✅ pass | — | [stop-lists](../zones/catalog/stop-lists.md) |
| TC-CHAIN-021 | Снятие со стопа | ✅ pass | — | [stop-lists](../zones/catalog/stop-lists.md) |
| TC-CHAIN-022 | Стоп категории | ⚠ backend pass, UI blocked F40 | F39 | [stop-lists](../zones/catalog/stop-lists.md) |
| TC-CHAIN-023 | Снятие категории со стопа | ⚠ backend pass | — | [stop-lists](../zones/catalog/stop-lists.md) |
| TC-CHAIN-024 | KDS settings update | ✅ pass | F40 (menu-availability inverted) | [menu-availability](../zones/catalog/menu-availability.md) |

## Этап 2 — POS → KDS маршрутизация (zones/kds/*, zones/pos/*)

| TC | Описание | Status | Findings | Zone |
|----|----------|--------|----------|------|
| TC-CHAIN-030 | 1 кухонный товар → KDS-Кухня | ✅ pass | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-031 | Только не-кухонные → не на KDS | ✅ pass | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-032 | Микс кухня+бар | ⛔ blocked | F2, F3, F40 | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-033 | Микс кухня+не-кухонный | ✅ pass (UX «поз. на других станциях») | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-034 | Модификаторы на KDS | ✅ pass | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-035 | Комментарий к позиции на KDS | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-036 | Стол/номер заказа на KDS | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-037 | qty > 1 на KDS | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-038 | Race на 2 POS одновременно | ⛔ blocked (нужен 2-й POS) | — | [pos/orders](../zones/pos/orders.md) |
| TC-CHAIN-039 | Multi-tenancy ТТ-A vs ТТ-B | ⏳ partial (через RBAC orders) | — | [auth-rbac](../zones/auth-rbac/) |
| TC-CHAIN-040 | Изменить kitchen_station_id после in_progress | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-041 | Удалить позицию в new — KDS обновляется | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-042 | Добавить позицию в new — KDS обновляется | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-043 | KDS оффлайн → reconnect → catch-up | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-044 | Refresh KDS — состояние восстанавливается | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |

## Этап 3 — Status flow (zones/orders/lifecycle.md)

| TC | Описание | Status | Findings | Zone |
|----|----------|--------|----------|------|
| TC-CHAIN-050 | new → in_progress (на POS / KDS) | ⚪ N/A by-design | F19 (сразу accepted) | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-051 | in_progress → ready (KDS «Готово») | ✅ pass | — | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-052 | ready → closed (выдача + оплата) | ⚠ backend pass | F25, F26 | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-053 | Отмена в new | ✅ backend (API) | F27 (UI нет) | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-054 | Отмена в in_progress | ◯ todo | F27 | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-055 | Отмена closed → отказ | ◯ todo | — | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-056 | Add позицию в in_progress → отказ | ◯ todo | — | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-057 | KDS откат «Готово» | ⚪ N/A by-design | F20 | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-058 | Per-position статусы | ✅ pass | — | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-059 | Заказ остаётся accepted пока не все ready | ✅ pass | F29 | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-060 | Race: 2 KDS жмут «Готово» | ◯ todo | — | [lifecycle](../zones/orders/lifecycle.md) |
| TC-CHAIN-061 | Per-day numbering на полночь | ◯ todo | F24 (пропуски) | [lifecycle](../zones/orders/lifecycle.md) |

## Этап 4 — Денормализация (zones/orders/denormalization.md)

| TC | Описание | Status | Findings | Zone |
|----|----------|--------|----------|------|
| TC-CHAIN-070 | Денорм имени | ✅ pass | — | [denormalization](../zones/orders/denormalization.md) |
| TC-CHAIN-071 | Денорм цены | ✅ pass | — | [denormalization](../zones/orders/denormalization.md) |
| TC-CHAIN-072 | Денорм цены опции | ⚠ partial — нашли F44 | — | [denormalization](../zones/orders/denormalization.md) |
| TC-CHAIN-073 | Soft-delete товара после создания заказа | ◯ todo | — | [denormalization](../zones/orders/denormalization.md) |
| TC-CHAIN-074 | Удалить группу модификаторов после заказа | ◯ todo | F45 | [denormalization](../zones/orders/denormalization.md) |
| TC-CHAIN-075 | Переименование категории — category_path | ◯ todo | — | [denormalization](../zones/orders/denormalization.md) |
| TC-CHAIN-076 | Shift-report за день | ⛔ blocked | F15 | [denormalization](../zones/orders/denormalization.md) |

## Этап 5 — RBAC (zones/auth-rbac/*)

| TC | Описание | Status | Findings | Zone |
|----|----------|--------|----------|------|
| TC-CHAIN-080 | Cashier ТТ-A POST orders ТТ-B | ⏳ partial | — | [auth-rbac/access-matrix](../zones/auth-rbac/access-matrix.md) |
| TC-CHAIN-081 | Cashier ТТ-A GET orders ТТ-B | ✅ pass | — | [auth-rbac/access-matrix](../zones/auth-rbac/access-matrix.md) |
| TC-CHAIN-082 | KDS-оператор ТТ-A видит только ТТ-A | ◯ todo | — | [auth-rbac/access-matrix](../zones/auth-rbac/access-matrix.md) |
| TC-CHAIN-083 | Manager POST /complete | ✅ pass | — | [auth-rbac/access-matrix](../zones/auth-rbac/access-matrix.md) |
| TC-CHAIN-084 | Manager mutation без permission → 403 | ✅ pass | — | [auth-rbac/access-matrix](../zones/auth-rbac/access-matrix.md) |
| TC-CHAIN-085 | Cashier cancel чужого заказа | ◯ todo | — | [auth-rbac/access-matrix](../zones/auth-rbac/access-matrix.md) |
| TC-CHAIN-086 | Franchisee cross-LE → 403 | ⛔ blocked (нет creds Franchisee) | — | [auth-rbac/access-matrix](../zones/auth-rbac/access-matrix.md) |
| TC-CHAIN-087 | PIN без pos.access | ◯ todo | — | [auth-rbac/access-matrix](../zones/auth-rbac/access-matrix.md) |
| TC-CHAIN-088 | Cashier admin login отказан | ✅ pass by-design | F12 | [auth-rbac/access-matrix](../zones/auth-rbac/access-matrix.md) |
| (extra) | Курьер RBAC расширенный (read-leak) | ❌ fail | F13 (7 эндпоинтов) | [auth-rbac/access-matrix](../zones/auth-rbac/access-matrix.md) |

## Этап 6 — Edge / exploratory

| TC | Описание | Status | Findings | Zone |
|----|----------|--------|----------|------|
| TC-CHAIN-100 | Пустой заказ → попытка оплаты | ◯ todo | — | [pos/orders](../zones/pos/orders.md) |
| TC-CHAIN-101 | 50+ позиций | ◯ todo | — | [pos/orders](../zones/pos/orders.md) |
| TC-CHAIN-102 | Длинный комментарий | ◯ todo | — | [pos/orders](../zones/pos/orders.md) |
| TC-CHAIN-103 | Длинная строка без пробелов | ◯ todo | — | [pos/orders](../zones/pos/orders.md) |
| TC-CHAIN-104 | POS оффлайн → создание заказа | ◯ todo | — | [pos/orders](../zones/pos/orders.md) |
| TC-CHAIN-105 | Catalog Service down | ◯ todo | — | [pos/orders](../zones/pos/orders.md) |
| TC-CHAIN-106 | Order Service down | ◯ todo | — | [pos/orders](../zones/pos/orders.md) |
| TC-CHAIN-107 | Сменить kitchen_station_id во время in_progress | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-108 | Рестарт pos-bff (рестарт WS) | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-109 | KDS показывает заказ → soft-delete товара | ◯ todo | — | [kds/routing](../zones/kds/routing.md) |
| TC-CHAIN-110 | qty=0/отрицательное на штучном | ◯ todo | — | [pos/orders](../zones/pos/orders.md) |
| TC-CHAIN-111 | Цена опции NULL → выбор | ⚠ см. F44 | — | [catalog/modifiers](../zones/catalog/modifiers.md) |
| TC-CHAIN-112 | Часы POS уехали → создание | ◯ todo | — | [pos/orders](../zones/pos/orders.md) |

---

## Условные обозначения

- ✅ **pass** — кейс прошёл успешно
- ❌ **fail** — нашли баг (см. колонку Findings)
- ⚠ **partial** — частично прошёл / нашли что-то незавершённое
- ⛔ **blocked** — невозможно прогнать на текущем стенде (нужны вводные)
- ⏳ **partial pass** — прогнали частично (например только admin часть, POS-side не подтверждена)
- ◯ **todo** — не запускали ещё
- ⚪ **N/A by-design** — кейс неприменим в текущей архитектуре
