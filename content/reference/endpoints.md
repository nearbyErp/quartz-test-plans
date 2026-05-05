# Карта API endpoints стенда

Реверс из admin SPA bundle и проб. **Источник правды для контракта** — что куда дёргать.

## ⚠️ Silent no-op endpoints (принимают body, но не пишут в БД)

Эти эндпоинты возвращают **200 OK**, но **молча игнорируют** часть полей. Связанные баги: F6a, F41, F44.

| Endpoint | Игнорируемое поле | Правильный путь |
|----------|--------------------|-----------------|
| `PATCH /catalog/products/{id}` | `base_price` | `PATCH /catalog/price-lists/{id}/items` body `{items: [...]}` |
| `PATCH /catalog/products/{id}` | `modifier_group_ids` | `POST /catalog/products/{id}/modifiers` body `{modifier_group_id, binding_type}` |
| `POST /modifier-groups` | `options[].price` | `PATCH /catalog/price-lists/{id}/modifier-items` body `{items: [{modifier_option_id, price}]}` |

## Auth

```
POST /api/v1/admin/auth/login           email + password → JWT
POST /api/v1/admin/auth/refresh         refresh_token
GET  /api/v1/admin/auth/me              текущий user + permissions
POST /api/v1/admin/auth/logout
POST /api/v1/admin/auth/forgot-password
POST /api/v1/admin/auth/reset-password
```

POS / KDS:
```
POST /api/v1/pos/auth/pin               PIN-логин (требует X-Device-Id header)
POST /api/v1/pos/auth/login             email + password (POS-web)
POST /api/v1/pos/manager-auth/verify-pin   manager-approval flow → X-Approval-Token
```

## Catalog

### Products
```
GET    /catalog/products                список (с пагинацией ?per_page=N)
GET    /catalog/products/{id}           детали (НЕ возвращает цену — F6b)
POST   /catalog/products                create
PATCH  /catalog/products/{id}           update (silent no-op для base_price/modifier_group_ids)
DELETE /catalog/products/{id}           soft-delete (500 для товара в состоянии F8)
GET    /catalog/products/{id}/modifiers список привязанных модификаторов
POST   /catalog/products/{id}/modifiers привязать модификатор
                                        body: {modifier_group_id, binding_type: "free"|"structural", override_min_amount?, override_max_amount?}
```

### Categories
```
GET    /catalog/categories              список (с детализацией — у каждой is_active, color, и т.д.)
GET    /catalog/categories/{id}         404 — не реализован (F10)
POST   /catalog/categories
PATCH  /catalog/categories/{id}         update
DELETE /catalog/categories/{id}         204
```

### Modifier-groups
```
GET    /modifier-groups                 список
GET    /modifier-groups/{id}            детали (поле price у options НЕ возвращается)
POST   /modifier-groups                 create (silent no-op для options[].price)
PATCH  /modifier-groups/{id}
DELETE /modifier-groups/{id}            204 (НЕ чистит modifier_items в price-list — F45)
```

### Price-lists
```
GET    /catalog/price-lists                                    список прейскурантов (только мета)
GET    /catalog/price-lists/{id}                               один прейскурант (мета: name, status, stores, is_default)
GET    /catalog/price-lists/{id}/items                         цены товаров + modifier_items
GET    /catalog/price-lists/{id}/items/hierarchical            цены с иерархией категорий
PATCH  /catalog/price-lists/{id}/items                         update цен товаров
                                                               body: {items: [{product_id, price}]}
PATCH  /catalog/price-lists/{id}/modifier-items                update цен опций модификаторов
                                                               body: {items: [{modifier_option_id, price}]}
PATCH  /catalog/price-lists/{id}                               update мета (но игнорирует items в body — silent)
```

### Stop-lists
```
GET    /catalog/stop-lists/stores/{store_id}                          стоп-лист ТТ (products[] + categories[])
POST   /catalog/stop-lists/stores/{store_id}/products                 добавить товар
                                                                       body: {product_id, reason?}
DELETE /catalog/stop-lists/stores/{store_id}/products/{product_id}    снять
POST   /catalog/stop-lists/stores/{store_id}/categories               категория в стоп
                                                                       body: {category_id, reason?}
DELETE /catalog/stop-lists/stores/{store_id}/categories/{category_id} снять категорию
```

### Menu-availability (временные окна доступности)
```
GET   /catalog/menu-availabilities         список правил
POST  /catalog/menu-availabilities         create правило
PATCH /catalog/menu-availabilities/{id}    update (status: "active" | "disabled" — regex enforced)
```

### Kitchen-stations
```
GET   /catalog/kitchen-stations            список станций (с product_count)
POST  /catalog/kitchen-stations
PATCH /catalog/kitchen-stations/{id}
```

## Orders

```
GET    /orders                          список (фильтры: store_id, status, date)
GET    /orders/{id}                     детали (со всеми timestamps + items + modifiers + fiscal + paykeeper)
POST   /orders/{id}/cancel              отмена (body обязателен: {reason})
                                        UI кнопки нет — F27
GET    /kitchen-queue?store_ids=X       кухонная очередь
GET    /refunds                         список рефандов
```

POS-side (через pos-bff, недоступен снаружи):
```
POST /api/v1/pos/orders/                создание заказа
POST /api/v1/pos/orders/submit
GET  /api/v1/pos/orders/by-table/
GET  /api/v1/pos/orders/open
GET  /api/v1/pos/orders/recent-paid     последние оплаченные
POST /api/v1/pos/refunds/
GET  /api/v1/pos/kitchen-queue/
POST /api/v1/pos/shifts/open
POST /api/v1/pos/shifts/close
GET  /api/v1/pos/reports/shift-report
```

## Stores / Legal entities / Employees

```
GET   /stores                           список ТТ
PATCH /stores/{id}
GET   /legal-entities
GET   /employees
GET   /employees/{id}
POST  /employees
PATCH /employees/{id}
GET   /roles
```

## KDS / POS devices

```
GET   /kds/devices                      список зарегистрированных KDS (8 устройств с kitchen_station_id: null — F3)
GET   /kds/settings                     глобальные настройки KDS (sound, volume, auto_logout)
PATCH /kds/settings                     update настроек (требует kds.settings.edit)
GET   /pos/devices                      список POS-устройств
POST  /pos/devices/register             регистрация POS
```

## 404 routes (зарегистрированы в SPA, не реализованы) — F4 / F16

```
GET /admin/tables                       (F4)
GET /admin/warehouse                    (F16)
GET /admin/shifts                       (F4-ext)
GET /admin/aggregators                  (F4-ext)
GET /admin/paykeeper                    (F4-ext)
GET /admin/dashboard                    (F4-ext)
```

## Headers POS BFF

POS desktop при запросах добавляет:
```
Authorization: Bearer <jwt>
Content-Type: application/json
X-Device-Id: <зарегистрированный device_id POS/KDS устройства>
X-Active-Store: <store_id>
X-App-Version: <0.1.1 etc.>
X-Approval-Token: <от manager-auth/verify-pin для escalations>
```

## Kafka топики (по docs)

```
catalog.product.upserted          producer: Catalog Service
catalog.product.deleted           producer
catalog.modifier_group.upserted   producer
catalog.modifier_group.deleted    producer
catalog.kds_settings.updated      producer (BR 5.1, P1 consumer = pos-bff)
external_menu.updated             producer
```

Outbox: `catalog_outbox` table — transactional доставка.
