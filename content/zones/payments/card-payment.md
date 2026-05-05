# Payments · Card payment (PayKeeper)

Оплата картой через PayKeeper. Фискализация, RRN, PayKeeper integration.

## Что попадает в БД при card-оплате

Заказ #026 (id `85777593-...`, оплачен 14:59:26) даёт референс:

```json
{
  "status": "closed",
  "payment_method": "card",
  "paid_amount": 347.5,
  "paid_at": "2026-05-05T14:59:26Z",
  "completed_at": "2026-05-05T14:59:26Z",
  "rrn": null,                              // ❌ F26
  "card_last4": null,                       // ❌ F26
  "fiscal_data": {
    "fn": "9999078902018961",
    "ts": "20260505T1455",
    "fnd": "35",
    "fpd": "3512313478",
    "rnkkt": "1840159071038290",
    "shift_number": 7,
    "receipt_number": 5
  },
  "fiscal_failed": false,
  "pk_invoice_id": "20260505145510997",
  "pk_invoice_url": "https://alfa-kassa-mgm-2.server.paykeeper.ru/bill/20260505145510997",
  "pk_payment_id": 34,
  "pk_fop_receipt_key": "b5788387-0a01-40bc-8c64-f75d54a7770b"
}
```

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F25 | 🔴 | Заказ исчезает из POS UI после оплаты (см. также pos/orders.md, F37) |
| F26 | 🟡 | RRN и `card_last4` = null после оплаты картой через PayKeeper |

## Тест-кейсы

### TC-PAY-CARD-026 — Структура заказа после card-оплаты
**Status:** ✅ partial (заказ #026)
fiscal_data заполнен, paykeeper-поля все есть. Но F26 — RRN и card_last4 null.

### TC-PAY-CARD-026-RRN — Регресс F26 после фикса
**Status:** verify-needed
1. Card-оплата на стенде
2. Через 1-2 сек GET /orders/{id} → rrn заполнено
3. card_last4 = 4 цифры

### TC-PAY-CARD-PK-FAIL — PayKeeper недоступен → fallback
**Status:** ◯ todo
Если можно симулировать недоступность PK (например через manual block) — что POS показывает кассиру?
