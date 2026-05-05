# Catalog · Price-lists

Прейскуранты, цены товаров и опций модификаторов, привязка к ТТ.

## Endpoints

```
GET    /catalog/price-lists                                    список (мета без цен)
GET    /catalog/price-lists/{id}                               детали (мета: name, status, stores, is_default)
GET    /catalog/price-lists/{id}/items                         цены: {items, modifier_items, price_list_id}
GET    /catalog/price-lists/{id}/items/hierarchical            с иерархией
PATCH  /catalog/price-lists/{id}/items                         обновить цены товаров
                                                               body: {items: [{product_id, price}]}
PATCH  /catalog/price-lists/{id}/modifier-items                обновить цены опций модификаторов
                                                               body: {items: [{modifier_option_id, price}]}
PATCH  /catalog/price-lists/{id}                               update мета (но игнорирует items в body)
```

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F1 | 🟡 | Default-прейскурант имеет `stores: []`, ни одна ТТ не привязана. Цены применяются неявно через default |
| BUG-050 | 🟡 | Прейскурант — нельзя редактировать «Название», «Активность», «Назначено ТТ» (только цены) |
| BUG-051 | 🟡 | Создание прейскуранта — невозможно вручную привязать к ТТ |

## Тест-кейсы

### TC-CATALOG-PL-017 — Прейскурант с override цены, привязать к ТТ
**Status:** ⛔ blocked (BUG-051) · **TC-CHAIN-017**
Привязать прейскурант к ТТ через UI/API не получается.
Workaround исследовать: можно ли через `PATCH /price-lists/{id}` передать `stores: [...]`?

### TC-CATALOG-PL-018 — Изменение цены → новые заказы получают новую
**Status:** ✅ pass · **TC-CHAIN-018**
1. PATCH `/price-lists/{id}/items` с новой ценой
2. Создать новый заказ — `unit_price` = новая
3. Старые заказы — старая (денормализация, см. orders/denormalization.md)

### TC-CATALOG-PL-019 — Цена 0
**Status:** ◯ todo · **TC-CHAIN-019**
PATCH `items` с `price: 0` → товар на POS с ценой 0 или скрыт?

### TC-CATALOG-PL-051 — Workaround для BUG-051
**Status:** ◯ todo
Через `PATCH /catalog/price-lists/{id}` body `{stores: ["<store_id>"]}` — пробовать.

## Snippets

```bash
# Получить все цены default-прейскуранта
curl -sS "$B/catalog/price-lists/5dcc6666-987a-4f22-84e0-7227b2c79c7a/items" -H "$H" | node -e "..."

# Изменить цену товара
echo '{"items":[{"product_id":"<id>","price":100}]}' > /tmp/p.json
curl -sS -X PATCH "$B/catalog/price-lists/5dcc6666-987a-4f22-84e0-7227b2c79c7a/items" -H "$H" -H "Content-Type: application/json" --data-binary @/tmp/p.json
```
