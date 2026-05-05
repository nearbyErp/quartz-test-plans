# Findings — все известные баги

Единая таблица всех известных проблем. Две схемы ID живут вместе:
- **BUG-NNN** — backlog из доквартцовой базы (`archive/04-known-bugs-index.md`), накопленный до e2e сессий
- **F-NN** — найденные в наших e2e сессиях

Колонка `Source` указывает откуда вылез баг.

**Условные обозначения:**
- 🔴 Critical · 🟡 Major · 🟢 Minor · ⚪ Info / by-design
- Status: `open` (не фиксили) · `in-progress` · `fixed` · `retracted` · `verify-needed` (нужно подтвердить после фикса)

---

## 🔴 CRITICAL

| ID | Severity | Status | Title | Zone | Source |
|----|----------|--------|-------|------|--------|
| BUG-005 | 🔴 | open | ЮЛ Карточка — вкладки «Информация», «Сотрудники», «Документы» не реагируют на клик | legal-entities | backlog |
| BUG-006 | 🔴 | open | Редактирование своей франшизы → 404 после Save | franchise | backlog |
| BUG-008 | 🔴 | open | Создание сотрудника → 500 INTERNAL_ERROR | employees-hr | backlog |
| BUG-013 | 🔴 | open | Manager → Юр.лица возвращает 403 (по спеке: read по permission) | auth-rbac | backlog |
| BUG-014 | 🔴 | open | Manager → Торговые точки возвращает 403 (по спеке: своя ТТ) | auth-rbac | backlog |
| BUG-015 | 🔴 | open | Создание ТТ возвращает 403 для Franchisee | auth-rbac | backlog |
| BUG-016 | 🔴 | open | Manager → Склад все вкладки → 403 (на самом деле 404 — см. F16) | auth-rbac/warehouse | backlog |
| BUG-017 | 🔴 | open | Manager → Заказы все вкладки → 403 | auth-rbac/orders | backlog |
| BUG-018 | 🔴 | open | Шаблоны смен → редактирование → 500 при Save | employees-hr | backlog |
| BUG-019 | 🔴 | open | Шаблон смены: повторный Save → 500 | employees-hr | backlog |
| BUG-023 | 🔴 | open | Формулы зарплаты → редактирование «по роли» → 500 | employees-hr | backlog |
| BUG-024 | 🔴 | open | Экспорт CSV ведомости → 422 | employees-hr | backlog |
| BUG-049 | 🔴 | open | Стоп-листы → 403 у Manager | auth-rbac/catalog | backlog |
| BUG-060 | 🔴 | open | Переход во вкладку «Меню» в карточке ТТ → 500 | catalog/menu-in-store | backlog |
| BUG-064 | 🔴 | open | При назначении официанта → 404 | orders/tables | backlog |
| BUG-065 | 🔴 | open | Перемещение стола в карте зала → белая страница | orders/tables | backlog |
| BUG-066 | 🔴 | open | «Склад не найден для выбранной ТТ» — блокирует создание приёмки | warehouse | backlog |
| F3 | 🔴 | open | KDS-устройства зарегистрированы без `kitchen_station_id` (8 устройств `null`) | kds | session-2026-05-05 |
| F6a | 🔴 | open | `PATCH /catalog/products/{id}` тихо игнорирует `base_price` (200 OK, но БД не меняется) | catalog/products | session-2026-05-05 |
| F8 | 🔴 | open | PATCH product теряет переданный `store_ids` при `available_in_all_stores: false` | catalog/products | session-2026-05-05 |
| F9 | 🔴 | open | DELETE product → 500 для товара в состоянии F8 (orphan в БД) | catalog/products | session-2026-05-05 |
| F13 | 🔴 | open | RBAC read-leak: 7 эндпоинтов доступны без permissions у Курьера | auth-rbac | session-2026-05-05 |
| F14 | 🔴 | open | Refunds доступны Курьеру без `pos.refund` (часть F13, выделено по фин-чувствительности) | auth-rbac/payments | session-2026-05-05 |
| F15 | 🔴 | open | HR-эндпоинты → 500 для всех ролей (4 шт: payroll, shift-templates, shift-records, schedules) | employees-hr | session-2026-05-05 |
| F25 | 🔴 | open | Заказ исчезает из POS после оплаты | pos | session-2026-05-05 |
| F27 | 🔴 | open | На POS нет UI-кнопки отмены заказа на любом статусе (API работает) | pos | session-2026-05-05 |
| F37 | 🔴 | open | На POS нет журнала закрытых заказов смены (только агрегированная аналитика) | pos | session-2026-05-05 |

## 🟡 MAJOR

| ID | Severity | Status | Title | Zone | Source |
|----|----------|--------|-------|------|--------|
| BUG-001 | 🟡 | open | Импорт ЮЛ из xlsx — `Unsupported Media Type` | legal-entities | backlog |
| BUG-002 | 🟡 | open | ЮЛ Создать — после 1 символа теряется фокус (все label) | legal-entities | backlog |
| BUG-007 | 🟡 | open | Поиск по ФИО не работает | employees-hr | backlog |
| BUG-009 | 🟡 | open | Создание сотрудника — нет валидации (Имя, Фамилия, Пароль, Телефон, ТТ) | employees-hr | backlog |
| BUG-011 | 🟡 | open | Деактивированный сотрудник — пункты бургера неактивны | employees-hr | backlog |
| BUG-012 | 🟡 | open | Не редактируется роль сотрудника с аккаунта администратора | employees-hr | backlog |
| BUG-020 | 🟡 | open | Создание смены «Вручную» → 422 при Save | employees-hr | backlog |
| BUG-021 | 🟡 | open | Отсутствует Clock in/out (фактическое время) | employees-hr | backlog |
| BUG-025 | 🟡 | open | UI позволяет подтвердить ведомость с некорректными данными | employees-hr | backlog |
| BUG-031 | 🟡 | open | «Доступно во всех точках» — после Save галочка возвращается (UI-симптом F8) | catalog/products | backlog |
| BUG-032 | 🟡 | open | Фильтр «Ингредиент» в Товарах — пусто | catalog/products | backlog |
| BUG-033 | 🟡 | open | Изменение товара не создаёт новую версию (по ADR-011 нужна) | catalog/products | backlog |
| BUG-034 | 🟡 | open | В Каталоге отсутствует фильтр с версионным списком | catalog/products | backlog |
| BUG-035 | 🟡 | open | Смена категории на «Без категории» — после Save возвращается исходная | catalog/products | backlog |
| BUG-036 | 🟡 | open | Дублирование товара не переносит фото, модификаторы, техкарту | catalog/products | backlog |
| BUG-039 | 🟡 | open | На странице категорий нельзя редактировать большинство полей | catalog/categories | backlog |
| BUG-040 | 🟡 | open | Каскадная активация категории не работает (только деактивация) | catalog/categories | backlog |
| BUG-041 | 🟡 | open | Категории — 255 символов без пробелов ломают вёрстку, при редактировании — 409 | catalog/categories | backlog |
| BUG-044 | 🟡 | open | Модификаторы — нет валидации min/max (см. F30, F31, F32) | catalog/modifiers | backlog |
| BUG-047 | 🟡 | open | Технология приготовления синхронизирована между опциями модификатора | catalog/modifiers | backlog |
| BUG-048 | 🟡 | open | Не сохраняется удаление текста из «Технология приготовления» | catalog/modifiers | backlog |
| BUG-050 | 🟡 | open | Прейскурант — нельзя редактировать «Название», «Активность», «Назначено ТТ» | catalog/price-lists | backlog |
| BUG-051 | 🟡 | open | Создание прейскуранта — невозможно вручную привязать к ТТ | catalog/price-lists | backlog |
| BUG-052 | 🟡 | open | Поиск по адресу не работает | stores | backlog |
| BUG-058 | 🟡 | open | Фильтр статус «Приостановлена» есть, функционала остановки ТТ нет | stores | backlog |
| BUG-061 | 🟡 | open | В разделе «Меню» отсутствует кнопка «Добавить товар» | catalog/menu-in-store | backlog |
| BUG-062 | 🟡 | open | В «Меню» вместо описания товара модификаторы | catalog/menu-in-store | backlog |
| BUG-063 | 🟡 | open | При назначении официанта показываются ВСЕ сотрудники системы | orders/tables | backlog |
| F1 | 🟡 | open | Прейскурант не привязан ни к одной ТТ (default применяется неявно — связано с BUG-051) | catalog/price-lists | session-2026-05-05 |
| F4 | 🟡 | open | 6 admin-маршрутов → 404: tables, warehouse, shifts, aggregators, paykeeper, dashboard | admin-bff routes | session-2026-05-05 |
| F6b | 🟡 | open | `GET /catalog/products/{id}` не возвращает текущую цену | catalog/products | session-2026-05-05 |
| F10 | 🟡 | open | `GET /catalog/categories/{id}` → 404 (PATCH/DELETE работают) | catalog/categories | session-2026-05-05 |
| F16 | 🟡 | open | `/admin/warehouse` → 404 даже у Manager с warehouse.read | warehouse | session-2026-05-05 |
| F21 | 🟡 | open | `assembly_time_seconds` в позиции заказа = null (денорм не сработала) | orders/lifecycle | session-2026-05-05 |
| F24 | 🟡 | verify-needed | Пропуски в нумерации заказов (#22, #25 потеряны) — гипотеза: симптом F25/F37 | orders/lifecycle | session-2026-05-05 |
| F26 | 🟡 | open | RRN и `card_last4` = null после оплаты картой через PayKeeper | payments/card-payment | session-2026-05-05 |
| F30 | 🟡 | open | Модификатор: `max < min` проходит (BUG-044 confirmed) | catalog/modifiers | session-2026-05-05 |
| F31 | 🟡 | open | Модификатор: отрицательный `min_amount` проходит | catalog/modifiers | session-2026-05-05 |
| F32 | 🟡 | open | Опция модификатора: отрицательная цена проходит (потенциальный обход скидок) | catalog/modifiers | session-2026-05-05 |
| F40 | 🟡 | open | `menu-availability` всегда скрывает категорию, не учитывает окно | catalog/menu-availability | session-2026-05-05 |
| F41 | 🟡 | open | PATCH product тихо игнорирует `modifier_group_ids` (правильный путь — POST /products/{id}/modifiers) | catalog/products | session-2026-05-05 |
| F44 | 🟡 | open | POST modifier-groups тихо игнорирует `options[].price` (правильный путь — PATCH /price-lists/{id}/modifier-items) | catalog/modifiers | session-2026-05-05 |

## 🟢 MINOR

| ID | Severity | Status | Title | Zone | Source |
|----|----------|--------|-------|------|--------|
| BUG-003 | 🟢 | open | КПП: валидация на латинице | legal-entities | backlog |
| BUG-004 | 🟢 | open | Нет сообщений об ошибке (Телефон, Расч.счёт, БИК, Корр.счёт) | legal-entities | backlog |
| BUG-010 | 🟢 | open | Создание сотрудника — PIN дублируется в верхней строке и в блоке ниже | employees-hr | backlog |
| BUG-022 | 🟢 | open | Расписание сотрудников — страница выглядит неполноценно | employees-hr | backlog |
| BUG-026 | 🟢 | open | Формулы зарплаты — у label «Ставка в час» виден технический код | employees-hr | backlog |
| BUG-027 | 🟢 | open | Формулы зарплаты — поле «Ставка в час» принимает -1 | employees-hr | backlog |
| BUG-028 | 🟢 | open | Ведомости — в колонке «Действия» технический код | employees-hr | backlog |
| BUG-029 | 🟢 | open | Страница «Терминалы» требует переосмысления | employees-hr | backlog |
| BUG-030 | 🟢 | open | Создание товара — поле «Описание» в шрифте textarea | catalog/products | backlog |
| BUG-037 | 🟢 | open | Поле «Описание» — растягивается без предела | catalog/products | backlog |
| BUG-038 | 🟢 | open | Поле «Описание» — длинные строки выходят за пределы | catalog/products | backlog |
| BUG-042 | 🟢 | open | Категории — пустое название сохраняется без ошибки | catalog/categories | backlog |
| BUG-043 | 🟢 | open | Модификатор — Описание в шрифте textarea | catalog/modifiers | backlog |
| BUG-045 | 🟢 | open | Модификатор — название опции: нет maxlength, принимает «Пробел» | catalog/modifiers | backlog |
| BUG-046 | 🟢 | open | Ингредиенты — Описание в шрифте textarea | catalog/ingredients | backlog |
| BUG-053 | 🟢 | open | График работы — добавить «Круглосуточно» рядом с «Выходной» | stores | backlog |
| BUG-054 | 🟢 | open | Создание ТТ — нет валидации Телефон, Адрес, Название | stores | backlog |
| BUG-055 | 🟢 | open | Широта/долгота из 2gis — должны вставляться обе одним действием | stores | backlog |
| BUG-056 | 🟢 | open | «Одинаковый для всех дней» — колонка «Выходной» должна быть неактивна | stores | backlog |
| BUG-057 | 🟢 | open | Часовые пояса — добавить Магадан (UTC+11), убрать Новосибирск | stores | backlog |
| BUG-059 | 🟢 | open | Фильтр по городу — требует 100% совпадение | stores | backlog |
| BUG-X01 | 🟢 | open | Тексты ошибок при невалидных данных — на латинице | cross-cutting | backlog |
| BUG-X02 | 🟢 | open | Шрифт textarea не консистентен с input | cross-cutting | backlog |
| BUG-X03 | 🟢 | open | Технические коды видны в UI вместо названий | cross-cutting | backlog |
| BUG-X04 | 🟢 | open | Длинные строки без пробелов выходят за границы | cross-cutting | backlog |
| F2 | 🟢 | open | Тестовые данные: все 11 кухонных товаров на одной станции «Кухня» | test-data | session-2026-05-05 |
| F5 | 🟢 | open | Refunds реализованы, документация устарела | docs | session-2026-05-05 |
| F7 | 🟢 | open | Тестовые данные: кофе на станции «Кухня» вместо «Бар» | test-data | session-2026-05-05 |
| F33 | 🟢 | open | Модификатор: max=999999 без верхнего предела | catalog/modifiers | session-2026-05-05 |
| F34 | 🟢 | open | Модификатор: можно создать без `options[]` | catalog/modifiers | session-2026-05-05 |
| F35 | 🟢 | open | Товар: `is_open_price=true` + `is_by_weight=true` одновременно (mutex отсутствует) | catalog/products | session-2026-05-05 |
| F39 | 🟢 | open | Стоп-лист: принимает `reason: null` без валидации | catalog/stop-lists | session-2026-05-05 |
| F42 | 🟢 | open | UX: при `max_amount` опций — нет подсказки на POS | pos/modifiers | session-2026-05-05 |
| F45 | 🟢 | open | DELETE modifier-group оставляет orphan-записи в `price-list/modifier-items` | catalog/modifiers | session-2026-05-05 |

## ⚪ INFO / BY-DESIGN / RETRACTED

Не баги, для контекста:

| ID | Что | Вердикт |
|----|-----|---------|
| F11 | Manager scope per-store (cross-LE возможен) | by-design — scope назначается per-store |
| F12 | Cashier admin-login отказан | by-design — кассиры через PIN на POS |
| F19 | `created_at == accepted_at == kitchen_started_at` на быстром заказе | observation — не баг |
| F20 | KDS одна кнопка «Готово», без двух фаз | by-design — нет start/finish |
| F22 | Семантика `kitchen_started_at` отличается на order/item | doc/UX — naming путаный, не блокер |
| F28 | Заказ только из «не-кухонных» позиций → сразу `ready` | by-design — обходит KDS |
| F29 | Status-машина `new → accepted → ready → closed/cancelled` | architectural — подтверждено |
| F38 | POS фильтрует товары без активных KDS на станции | retracted — оказалось F40 (категория Кофе скрыта) |
| F43 | POS не считает цену опций при показе total | retracted — оказалось F44 (опция реально стоила 0 в БД) |

---

## Сводка

| Severity | Active count |
|----------|--------------|
| 🔴 Critical | 27 |
| 🟡 Major | 41 |
| 🟢 Minor | 32 |
| ⚪ Info | 9 |
| **Total active** | **100** |

## Следующий ID для новой находки

- `BUG-NNN` (старая схема, для backlog/regression): следующий **BUG-067** (или BUG-X05 для cross-cutting)
- `F-NN` (наша схема, для e2e сессий): следующий **F46**
