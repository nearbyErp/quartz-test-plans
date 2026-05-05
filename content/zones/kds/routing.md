# KDS · Routing

Маршрутизация позиций заказа по `kitchen_station_id` на разные KDS-экраны.

## Endpoints

```
GET   /kitchen-queue?store_ids=X        кухонная очередь (admin view)
GET   /kds/devices                      список зарегистрированных KDS
GET   /kds/settings                     глобальные настройки звука/таймеров
PATCH /kds/settings                     update
```

POS-side (недоступно снаружи):
```
GET /api/v1/pos/kitchen-queue/
GET /api/v1/pos/kitchen-stations/
```

## Тестовые данные

⚠ Сейчас на стенде:
- 11 «кухонных» товаров (`requires_kitchen=true`) → все на станции **Кухня**
- 4 не-кухонных (Кола, Сок, Маффин, Чизкейк) — без станции
- Бар, Горячий цех, Холодный цех — пустые (F2)
- 8 KDS-устройств без `kitchen_station_id` (F3)

→ Микс кухня+бар не воспроизводится без подготовки (см. F2, F3, F40).

## Findings

| ID | Severity | Title |
|----|----------|-------|
| F2 | 🟢 | Все 11 кухонных товаров на одной станции «Кухня» |
| F3 | 🔴 | KDS-устройства зарегистрированы без `kitchen_station_id` |
| F7 | 🟢 | Кофе на станции «Кухня» вместо «Бар» |

## Тест-кейсы

### TC-KDS-ROUTE-030 — Заказ с 1 кухонным товаром → KDS-Кухня
**Status:** ✅ pass · **TC-CHAIN-030**
Через kitchen-queue API: позиция с `kitchen_station_id=Кухня` появляется в очереди ≤2 сек.

### TC-KDS-ROUTE-031 — Заказ только из не-кухонных
**Status:** ✅ pass · **TC-CHAIN-031**
Кола 0.5 одним → status=ready сразу, в kitchen-queue не появляется (F28).

### TC-KDS-ROUTE-032 — Микс кухня+бар (две станции)
**Status:** ⛔ blocked · **TC-CHAIN-032**
Не воспроизводится из-за F2 (нет товаров на Бар) + F3 (KDS без станций) + F40 (категория Кофе скрыта).
Workaround: после фикса F40 + назначения станций — переназначить Эспрессо на Бар временно.

### TC-KDS-ROUTE-033 — Микс кухня+некухонный
**Status:** ✅ pass UX · **TC-CHAIN-033**
KDS-Кухня показывает кухонную позицию + строкой «(поз. на других станциях)».

### TC-KDS-ROUTE-034 — Модификаторы на KDS
**Status:** ✅ pass (заказ #026 Шаурма + Размер M) · **TC-CHAIN-034**

### TC-KDS-ROUTE-035 — Комментарий к позиции на KDS
**Status:** ◯ todo · **TC-CHAIN-035**

### TC-KDS-ROUTE-036 — Стол / номер заказа на KDS
**Status:** ◯ todo · **TC-CHAIN-036**

### TC-KDS-ROUTE-037 — qty > 1 (×N или N карточек)
**Status:** ◯ todo · **TC-CHAIN-037**

### TC-KDS-ROUTE-040 — Изменить kitchen_station_id во время in_progress
**Status:** ◯ todo · **TC-CHAIN-040**
Существующая позиция на исходном KDS, новые — на новой?

### TC-KDS-ROUTE-041 — Удалить позицию в new — KDS обновляется
**Status:** ◯ todo · **TC-CHAIN-041**

### TC-KDS-ROUTE-042 — Добавить позицию в new — KDS обновляется
**Status:** ◯ todo · **TC-CHAIN-042**

### TC-KDS-ROUTE-043 — KDS оффлайн → reconnect → catch-up
**Status:** ◯ todo · **TC-CHAIN-043**

### TC-KDS-ROUTE-044 — Refresh KDS — состояние восстанавливается
**Status:** ◯ todo · **TC-CHAIN-044**

### TC-KDS-ROUTE-003 — Назначить станции 8 KDS-устройствам (F3)
**Status:** ◯ todo
Через UI или API. Найти endpoint.
