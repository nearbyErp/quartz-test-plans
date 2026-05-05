# TODO — что хочется покрыть дальше

Очередь следующих прогонов. После каждой сессии — отметить что закрыто, добавить новое.

## Завтра (2026-05-06)

### Регресс по фиксам
- [ ] Когда коллега-разраб задеплоит — пройти F-номера через готовые curl из `reference/endpoints.md` + zones-файлов. Целиться на:
  - F8 (PATCH product store_ids потеря) — критичен
  - F9 (DELETE 500) — связан с F8
  - F13 (RBAC leak 7 эндпоинтов)
  - F15 (HR 500 4 эндпоинта)
  - F6a (PATCH product тихо ignores) — потребует решения «реализовать или отвечать 400»
- [ ] После прогона: пометить findings как `fixed` или оставить `open` с пометкой «не сошёлся фикс»

### E2E с нулевой франшизой ([scenarios/new-franchisee.md](scenarios/new-franchisee.md))
- [ ] Сценарий: создать ЮЛ → ТТ → сотрудников → каталог → прейскурант → привязать к ТТ → KDS-устройство → POS-устройство → выдать PIN → провести первый заказ → закрыть → выдать
- [ ] **Подводные ожидаемые камни:**
  - BUG-051 / F1 — привязка прейскуранта к ТТ
  - F3 — KDS device без `kitchen_station_id` при регистрации
  - F8 — PATCH product при ограничении доступности
  - F6a / F44 — управление ценами (только через price-list endpoints)
  - F25 / F37 — куда денется первый закрытый заказ
- [ ] Нужны: учётка root-админа платформы (создавать франшизу = уровень выше Franchise scope)

## Не покрыто из chain-admin-pos-kds (39/60)

См. галочки в [scenarios/chain-admin-pos-kds.md](scenarios/chain-admin-pos-kds.md).

Главное:
- TC-CHAIN-032 (микс кухня+бар) — заблокировано F2 (Бар пустой) + F3
- TC-CHAIN-038 (race на 2 POS одновременно) — нужен второй POS
- TC-CHAIN-061 (полночь — переход номера) — отложено
- TC-CHAIN-100..112 (edge cases — POS оффлайн, длинные строки, пустой заказ) — частично делали
- F26 регрессия cash payment — посмотреть `fiscal_data` для оплаты наличными (без PayKeeper)
- TC-CHAIN-070..076 денорм — частично сделано (имя+цена), осталось: модификаторы (denorm), удаление товара после создания заказа, и т.д.

## Идеи для exploratory-сессий

- [ ] **Aggregator интеграция** — ничего пока не тестировали. Если есть hooks Yandex.Eda / Koala — попробовать заказ оттуда
- [ ] **External menus** (BR 4.1) — конструктор меню для kiosk-mode + JSON-export. Endpoint видели (`/admin/external-menus`), TC нет
- [ ] **PayKeeper deep dive** — почему RRN не приходит (F26). Логи?
- [ ] **Manager approval flow** — `X-Approval-Token` header, `/api/v1/pos/manager-auth/verify-pin`. Когда требуется на POS?
- [ ] **Стоп-лист с временным окном** — F40 показал что menu-availability сломан. Можно ли стоп с reasonom auto-снятся через N часов?
- [ ] **Соратники** (HR — payroll, schedules, shift-templates) — все 500. После фикса F15 — большой объём непокрытого

## Технический долг (попросить тест-лида)

См. также [reference/stand.md#тех-долг-стенда](reference/stand.md):
- Удалить orphan продукт `f9cfbec8` (от F9)
- Очистить 8 orphan modifier_items (от F45)
- Назначить станции 8 KDS-устройствам (F3)
- Назначить кофе на станцию «Бар» (F7) или создать барные товары (F2)
- Привязать default-прейскурант к ТТ (BUG-051) или подтвердить что null = неявный default

## Когда настанет TMS-миграция

- Каждый файл `zones/<domain>/<feature>.md` → Suite в TMS
- Каждый `## TC-XXX` внутри → Test Case
- `findings.md` строки → Issues в багтрекере (через CSV-export)
- `sessions/*.md` → Test Run records
- Скрипт миграции пока не нужен — пишем структурно так, чтобы потом mapping был механический
