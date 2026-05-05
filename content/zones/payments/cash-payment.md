# Payments · Cash payment

Оплата наличными. Без PayKeeper, но с фискализацией.

## Что хочется проверить

- Структура `fiscal_data` для cash (отличается ли от card?)
- Поля `pk_*` — должны быть `null` (нет PK при cash)
- Поле `rrn` — для cash должно быть `null` (это банковский RRN, к cash не относится)
- Поле `paid_amount` равно `total`
- Сдача (если оплата ≥ total)?

## Findings

(нет — пока не тестировали)

## Тест-кейсы

### TC-PAY-CASH-001 — Базовая оплата наличными
**Status:** ◯ todo
1. Создать заказ
2. Оплатить наличными точную сумму
3. GET /orders/{id} → payment_method=cash, paid_at=есть, fiscal_data заполнен, pk_* = null

### TC-PAY-CASH-CHANGE — Сдача
**Status:** ◯ todo
1. Заказ на 100₽
2. Оплата 200₽
3. UI должен показать сдачу 100₽
4. В БД paid_amount = 100 или 200?

### TC-PAY-CASH-PARTIAL — Частичная (если поддерживается)
**Status:** ◯ todo
Mixed payment cash + card?
