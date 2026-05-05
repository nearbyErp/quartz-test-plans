# Payments · Fiscalization (ОФД)

Фискальные поля заказа: fiscal_data структура, fiscal_failed flag, retry-логика.

## Структура `fiscal_data`

```json
{
  "fn":     "9999078902018961",   // номер ФН (фискального накопителя)
  "ts":     "20260505T1455",      // timestamp
  "fnd":    "35",                  // ?
  "fpd":    "3512313478",          // фискальный признак данных
  "rnkkt":  "1840159071038290",    // регистрационный номер ККТ
  "shift_number":   7,
  "receipt_number": 5
}
```

## Findings

(нет findings пока специфичных к фискалке. F26 — про RRN, скорее payment, не fiscal.)

## Тест-кейсы

### TC-FISCAL-001 — Структура fiscal_data корректна
**Status:** ✅ partial (заказ #026)
Все поля заполнены, типы корректны.

### TC-FISCAL-FAIL — Фискализация не прошла → fiscal_failed=true
**Status:** ◯ todo
Если можно симулировать ОФД-проблему — что POS показывает кассиру?
Если backend записывает заказ с `fiscal_failed: true` — есть ли retry?

### TC-FISCAL-Z-REPORT — Z-отчёт (закрытие смены)
**Status:** ⛔ blocked F15 (shift endpoints 500)
После фикса F15 — закрыть смену → проверить структуру Z-отчёта.

### TC-FISCAL-RECEIPT-NUMBER — receipt_number монотонно
**Status:** ◯ todo
Проверить что `receipt_number` инкрементируется монотонно в рамках смены.

### TC-FISCAL-SHIFT-NUMBER — shift_number смены
**Status:** ◯ todo
При закрытии и открытии новой смены — `shift_number` должен увеличиваться.
