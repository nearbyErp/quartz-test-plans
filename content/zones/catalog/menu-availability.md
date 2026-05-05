# Catalog · Menu-availability

Временные правила доступности категорий/товаров (окна типа «Завтрак 07-12», «Десерты 14-22»).

## Endpoints

```
GET   /catalog/menu-availabilities                список правил
POST  /catalog/menu-availabilities                create
PATCH /catalog/menu-availabilities/{id}           update (status: "active" | "disabled" — regex enforced)
```

## Структура правила

```json
{
  "id": "uuid",
  "franchise_id": "uuid",
  "name": "Завтрак",
  "days_of_week": [1,2,3,4,5,6,7],
  "time_from": "07:00:00",
  "time_to": "12:00:00",
  "scope": "categories",       // categories | products
  "target_ids": ["<category_or_product_id>"],
  "store_ids": ["<store_id>"],
  "status": "active"            // active | disabled
}
```

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F40 | 🟡 | Active правило **всегда скрывает** target, не учитывая окно времени |

## Тест-кейсы

### TC-CATALOG-MA-024 — Изменение настроек KDS
**Status:** ✅ pass (PATCH работает) · **TC-CHAIN-024**

### TC-CATALOG-MA-040 — Правила работают по окну (F40)
**Status:** ❌ fail · **session 2026-05-05**
1. Правило «Десерты 14:00-22:00» status=active. Текущее время в окне.
2. Десерты должны быть видны на POS.
3. **Факт:** Десерты НЕ видны.
4. Деактивация правила (status=disabled) → Десерты появляются.

**Гипотеза:** active = всегда скрыто, время окна не учитывается.

### TC-CATALOG-MA-COMPARE — «Завтрак» 07-12 vs «Десерты 14-22»
**Status:** ⚠ partially observed
| Правило | Сейчас | Ожидание | Факт |
|---------|--------|----------|------|
| Завтрак (Кофе) 07-12 | вне окна | Кофе скрыт | скрыт ✅ |
| Десерты 14-22 | в окне | Десерты видны | скрыты ❌ |

Ровно то что F40 описывает.

## Snippets

```bash
# Список правил
curl -sS "$B/catalog/menu-availabilities" -H "$H"

# Деактивировать правило
echo '{"status":"disabled"}' > /tmp/r.json
curl -sS -X PATCH "$B/catalog/menu-availabilities/<id>" -H "$H" -H "Content-Type: application/json" --data-binary @/tmp/r.json
```
