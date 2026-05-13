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
| BUG-005 | 🔴 | retracted | ~~ЮЛ Карточка — вкладки «Информация», «Сотрудники», «Документы» не реагируют на клик~~ — UI переделан (Реквизиты/ТТ/PayKeeper), описанные вкладки удалены. Retracted 2026-05-12. | legal-entities | backlog |
| F81 | ⚪ | retracted | ~~«Чаевые (Нетмонет)» отображают frontend-hardcoded mock-данные на проде~~ — by-design по [[obsidian_erp/07-Tasks/Business Requirements/3.2 Интеграция с Нетмонет (чаевые)|BR 3.2]] §R5: Волна 1 mock-API готов, Волна 2 (webhook) заблокирована — ждём contract от `support@netmonet.co`. Retracted 2026-05-13 (Александр: «сознательно промокано»). | finance/tips | session-2026-05-12 |
| F86 | ⚪ | fixed | ~~ТТ → вкладка «Меню»: `GET /api/v1/admin/catalog/menu/<store>` → 500~~ — дубликат BUG-060 из legacy backlog, который **был починен деплоем 2026-05-12** (см. CONTEXT.md). Финдинг записан в той же сессии после деплоя, симптом не отыграли заново. Подтверждено 2026-05-13 через Playwright: endpoint возвращает 200 OK с 12 категориями и 50 товарами (ТТ Smoke 001). Код в `InternalCatalogMenuController` нашпигован null-safety защитами с комментариями про прошлые NPE. Закрыто как retracted-duplicate. | stores/menu | session-2026-05-12 |
| F91 | 🔴 | fixed | ~~PATCH ТТ с длинным названием (516 char + emoji) → 500~~ — fixed 2026-05-13 через [[obsidian_erp/07-Tasks/Bugs/BUG-026 Store name PATCH POST 500 на длинной строке/Баг\|BUG-026]]: `@Size(max=255)` в Create/UpdateStoreRequest. Регресс на admin.nirbi.ru: 516+emoji → `400 VALIDATION_ERROR` с `details:[{field:"name",message:"name must not exceed 255 characters"}]`, 255 → 200 OK. UI мапит на `errors.name` (EditPage.tsx:161-164). | stores | session-2026-05-12 |
| F93 | ⚪ | fixed | ~~Save шаблона смены → 500~~ — фикс в `521e343 fix(user): BR 1.4.4 — migrate security layer to Auth Service delegation`. `JwtUser.getRole()` получил backward-compat (маппит scope `all_franchise → admin_franchise`), legacy-switch в `ShiftTemplateService` теперь работает. Подтверждено 2026-05-13 на проде: POST→201, PATCH→200, DELETE→204. PROPOSAL (не в скоупе): migrate 9 HR-сервисов с `user.getRole()` на `user.hasPermission(...)`. | employees-hr | session-2026-05-12 |
| F94 | ⚪ | fixed | ~~Save формулы зарплаты «По роли» → 500~~ — 500 больше не воспроизводится. POST/PATCH `/salary-formulas` на проде отдаёт корректные 400 VALIDATION_ERROR с понятным сообщением. Тип `by_role` (легаси) больше не валиден — сейчас допустимы `hourly | fixed | mixed`. Это часть BR 1.4.4 security миграции (`521e343`) + апдейта формул. Если UI всё ещё показывает «По роли» — это отдельный UI/spec-issue, не F94. | employees-hr | session-2026-05-12 |
| F95 | 🔴 | fixed | ~~Экспорт CSV ведомости → 422 (broken URL)~~ — fixed 2026-05-13 через [[obsidian_erp/07-Tasks/Bugs/BUG-027 Payroll Export CSV broken URL/Баг\|BUG-027]] (v2): endpoint оказался bulk per-store-period (читает store_id+period_start из record по id), а не per-record. PayrollPage логика: 0 ведомостей → disabled; «Все ТТ» → toast «Выберите ТТ»; ТТ + N≥1 → `exportPayrollCsv(items[0].id)` → CSV всех сотрудников ТТ. Regress на проде: реальный CSV `payroll-2026-05.csv` скачан Playwright'ом со всеми 3 сотрудниками Столовой №1. | employees-hr | session-2026-05-12 |
| F98 | ⚪ | fixed | ~~Дублирование товара → 500~~ — fixed в commit `7fe6cd3 feat(catalog): дубликат товара — backend deep-copy`. `ProductService.duplicateProduct:153-242` делает полный deep-copy: product + modifiers + price_list_items (0₽) + Kafka publish + best-effort tech_card. Подтверждено 2026-05-13 на проде: POST без CT (как делает фронт `catalog.ts:70`) → 201, дубликат создан корректно. Дублирует часть BUG-036 (фото/модификаторы/техкарта переносятся). Zombie-копии после теста удалены. | catalog/products | session-2026-05-12 |
| F99 | ⚪ | fixed | ~~«+ Дочерняя» создаёт корневую — parent_id не сохраняется~~ — на проде иерархия работает корректно. Подтверждено 2026-05-13: (1) UI-клик «+ Дочерняя» на «Напитки» → POST body содержит `parent_id: <Напитки.id>`; (2) бэк `CategoryService.createCategory:66` сохраняет `parent_id` в Entity; (3) GET возвращает nested tree через `buildCategoryTree`, новая категория в `Напитки.children[2]`. Финдинг устарел/некорректное воспроизведение. | catalog/categories | session-2026-05-12 |
| F100 | ⚪ | fixed | ~~PayKeeper accounts → 502 Bad Gateway~~ — fixed by infra deployment между 2026-05-12 и 2026-05-13. Подтверждено: `GET /paykeeper/accounts` → 200 `{data:[], meta:{total:0}}`. paykeeper-adapter pod жив, отвечает. UI «Нет активных интеграций PayKeeper» теперь корректно отражает пустое состояние, не скрывает ошибку. Memory `project_prod_stand_admin` обновлён. | employees/integration | session-2026-05-12 |
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
| BUG-066 | 🔴 | fixed | «Склад не найден для выбранной ТТ» — блокирует создание приёмки | warehouse | backlog |
| F3 | 🔴 | fixed | KDS-устройства зарегистрированы без `kitchen_station_id` (8 устройств `null`) | kds | session-2026-05-05 |
| F6a | ⚪ | fixed | ~~`PATCH /catalog/products/{id}` тихо игнорирует `base_price`~~ — by-design after BR 1.10 (миграция 013-create-price-lists.xml: `ALTER TABLE products DROP COLUMN base_price CASCADE`). Цены в `price_list_items`, меняются через `PATCH /api/v1/admin/catalog/price-lists/{id}/items`. Поле `base_price` нет ни в БД, ни в Entity, ни в `UpdateProductRequest` — Jackson по умолчанию `FAIL_ON_UNKNOWN_PROPERTIES=false` его игнорит. Финдинг записан до раскатки миграции 013. Retracted 2026-05-13. | catalog/products | session-2026-05-05 |
| F8 | 🔴 | fixed | PATCH product теряет переданный `store_ids` при `available_in_all_stores: false` | catalog/products | session-2026-05-05 |
| F9 | 🔴 | fixed | DELETE product → 500 для товара в состоянии F8 (orphan в БД) | catalog/products | session-2026-05-05 |
| F13 | 🔴 | fixed | RBAC read-leak: 7 эндпоинтов доступны без permissions у Курьера | auth-rbac | session-2026-05-05 |
| F14 | 🔴 | fixed | Refunds доступны Курьеру без `pos.refund` (часть F13, выделено по фин-чувствительности) | auth-rbac/payments | session-2026-05-05 |
| F15 | 🔴 | fixed | HR-эндпоинты → 500 для всех ролей (4 шт: payroll, shift-templates, shift-records, schedules) | employees-hr | session-2026-05-05 |
| F25 | 🔴 | open | Заказ исчезает из POS после оплаты | pos | session-2026-05-05 |
| F27 | 🔴 | open | На POS нет UI-кнопки отмены заказа на любом статусе (API работает) | pos | session-2026-05-05 |
| F37 | 🔴 | open | На POS нет журнала закрытых заказов смены (только агрегированная аналитика) | pos | session-2026-05-05 |
| F26 | 🔴 | open | После оплаты картой rrn/card_last4 = null всегда; fiscal_data + pk_fop_receipt_key — нестабильно (регресс?). Compliance/fiscal-блокер | payments/card-payment | session-2026-05-05 |
| F60 | ⚪ | fixed | ~~Уникальность `fs_number` не проверяется~~ — на текущей версии защита работает на двух уровнях: preflight `findByFsNumberAndDeletedAtIsNull` в `POSTerminalService:42` (→ 409 FS_NUMBER_DUPLICATE) + БД unique-index `uq_pos_terminals_fs_number` из миграции 007. Подтверждено 2026-05-13 на проде через Playwright: дубликат → 409 как по спеке. Финдинг устарел/некорректное воспроизведение. Side-нюанс: unique-index без `WHERE deleted_at IS NULL` — после soft-delete тот же ФН нельзя переиспользовать (отдельный риск, не F60). | stores/terminals | session-2026-05-06-e2e |

## 🟡 MAJOR

| ID | Severity | Status | Title | Zone | Source |
|----|----------|--------|-------|------|--------|
| F77 | 🔴 | open | **ПЕРЕСМОТРЕН до Critical**: POST `/legal-entities` с inn=5digits/ogrn=7digits/email=без@ → **500 INTERNAL_ERROR** (не VALIDATION). Та же категория что F91 — отсутствие @Pattern/@Size в CreateLegalEntityRequest + DB-constraint violation → catch-all 500. Кандидат на фикс по паттерну BUG-026. Подтверждено 2026-05-13. | legal-entities | session-2026-05-12 |
| F78 | ⚪ | fixed | ~~ТТ → Интеграции: `GET /paykeeper/accounts/by-store/<id>` → 500~~ — fixed by infra deployment (тот же fix что F100). Подтверждено: → 200 `{account_id:null, status:"not_configured"}`. ТТ Smoke 001 корректно отдаёт «не настроено» вместо 500. | stores/integrations | session-2026-05-12 |
| F80 | 🟡 | open | **Подтверждено многократно сегодня**: 8+ API-запросов в одном Playwright-evaluate → JWT истёк → 401 → cleanup упал. Refresh-flow на фронте отсутствует. Требуется: фронт `client.ts` должен ловить 401, дёргать `/auth/refresh` через refresh_token, повторять запрос. Архитектурный фикс затрагивает все API-вызовы. | auth-rbac | session-2026-05-12 |
| F84 | 🟡 | open | Симптом про UI — бэк работает по спеке (POST пустого → 400 INVALID_REQUEST_BODY, валидные DTO → 400 VALIDATION_ERROR с details). Требует UI-проверки `ProductCreatePage.tsx:handleSubmit`. Не подтверждён через API. | catalog/products | session-2026-05-12 |
| F85 | 🔴 | open | **Подтверждено: backend XSS** — POST product `name: "<script>alert(1)</script>"` → 201 OK, name сохранён as-is. React по умолчанию escape'ит, но: (1) если где-то `dangerouslySetInnerHTML`, (2) если экспорт в email/PDF/CSV, (3) если внешний клиент не React — есть исполнение. Нужен HTML-sanitize на бэке (StripHtml / OWASP) или строгий @Pattern для name. Severity повышен до Critical (security). | catalog/products | session-2026-05-12 |
| F90 | 🟡 | open | **Подтверждено**: GET response не содержит ETag/If-Match/version headers. Last-write-wins без optimistic locking. Архитектурный вопрос — добавление поля `version` во все entities + If-Match header + 412 Precondition Failed на конфликте. Большой scope. | all | session-2026-05-12 |
| F92 | 🟡 | open | **Подтверждено на бэке**: POST `/stores` с email=«noatsign» → 201 OK accepted. `CreateStoreRequest` не имеет `@Email`. Связано с F77 (тот же класс — defensive validation). | all/forms | session-2026-05-12 |
| F96 | 🟡 | open | **Подтверждено**: 409 NAME_DUPLICATE → `"Role with this name already exists"` (англ) в русском UI. Все error message из ApiException — на английском. Требуется i18n-слой (например MessageSource с `error.NAME_DUPLICATE` ключами + Accept-Language header). Архитектурный фикс — затрагивает все сервисы. | all/i18n | session-2026-05-12 |
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
| BUG-051 | 🟡 | open | Прейскурант: PATCH `/price-lists/{id}` со `stores`/`store_ids` тихо игнорится. Workaround — PATCH `/stores/{id}` c `price_list_id` (это работает) | catalog/price-lists | backlog |
| BUG-052 | 🟡 | open | Поиск по адресу не работает | stores | backlog |
| BUG-058 | 🟡 | open | Фильтр статус «Приостановлена» есть, функционала остановки ТТ нет | stores | backlog |
| BUG-061 | 🟡 | open | В разделе «Меню» отсутствует кнопка «Добавить товар» | catalog/menu-in-store | backlog |
| BUG-062 | 🟡 | open | В «Меню» вместо описания товара модификаторы | catalog/menu-in-store | backlog |
| BUG-063 | 🟡 | open | При назначении официанта показываются ВСЕ сотрудники системы | orders/tables | backlog |
| F1 | ⚪ | fixed | ~~Прейскурант не привязан, default неявно~~ — by-design: `InternalCatalogMenuController:60-66` использует `findByFranchiseIdAndIsDefaultTrue` как fallback при null price_list_id. Подтверждено 2026-05-13: Smoke 001 имеет `price_list_id=null`, /menu отдаёт меню по default «Базовый». Архитектурный fallback. | catalog/price-lists | session-2026-05-05 |
| F4 | 🟡 | retracted | 6 admin-маршрутов → 404: tables, warehouse, shifts, aggregators, paykeeper, dashboard | admin-bff routes | session-2026-05-05 |
| F6b | ⚪ | fixed | ~~`GET /catalog/products/{id}` не возвращает текущую цену~~ — by-design after BR 1.10, зеркало F6a. ProductResponse не содержит цены, потому что цена живёт в `price_list_items` и резолвится per-prislist/per-store через `GET /internal/catalog/menu` (учитывает прейскурант ТТ, time tariffs, stop-lists). Retracted 2026-05-13. | catalog/products | session-2026-05-05 |
| F10 | 🟡 | fixed | `GET /catalog/categories/{id}` → 404 (PATCH/DELETE работают) | catalog/categories | session-2026-05-05 |
| F16 | 🟡 | verify-needed | Требует **Manager-аккаунта**. На проде только admin@erp.local (по memory project_prod_stand_admin — новых аккаунтов на проде не создаём). Перенести в backlog. | warehouse | session-2026-05-05 |
| F21 | 🟡 | fixed | `assembly_time_seconds` в позиции заказа = null (денорм не сработала) | orders/lifecycle | session-2026-05-05 |
| F24 | 🟡 | verify-needed | Пропуски в нумерации заказов (#22, #25 потеряны) — гипотеза: симптом F25/F37 | orders/lifecycle | session-2026-05-05 |
| F30 | 🟡 | fixed | Модификатор: `max < min` проходит (BUG-044 confirmed) | catalog/modifiers | session-2026-05-05 |
| F31 | 🟡 | fixed | Модификатор: отрицательный `min_amount` проходит | catalog/modifiers | session-2026-05-05 |
| F32 | 🟡 | retracted | Опция модификатора: отрицательная цена проходит (потенциальный обход скидок) | catalog/modifiers | session-2026-05-05 |
| F40 | 🟡 | open | `menu-availability` всегда скрывает категорию, не учитывает окно | catalog/menu-availability | session-2026-05-05 |
| F41 | ⚪ | fixed | ~~PATCH product silent ignore `modifier_group_ids`~~ — by-design: модификаторы через `POST/PATCH /products/{id}/modifiers/{groupId}`. Тот же паттерн что F6a (Jackson `FAIL_ON_UNKNOWN_PROPERTIES=false`). Подтверждено 2026-05-13. | catalog/products | session-2026-05-05 |
| F44 | 🟡 | retracted | POST modifier-groups тихо игнорирует `options[].price` (правильный путь — PATCH /price-lists/{id}/modifier-items) | catalog/modifiers | session-2026-05-05 |

## 🟢 MINOR

| ID | Severity | Status | Title | Zone | Source |
|----|----------|--------|-------|------|--------|
| F76 | 🟢 | open | **Подтверждено**: `legal-entities/ViewPage.tsx:373` — `<Link to="/legal-entities">{le.store_count} ТТ</Link>`. Ведёт на список ЮЛ, должно на `/stores?legal_entity_id=...`. Тривиальный фикс. | legal-entities | session-2026-05-12 |
| F79 | 🟢 | open | **Подтверждено**: POST categories с name=`"тестовая категория"` → name сохранён as-is с кавычками. Нет санитизации/трима/escape. Низкая severity, скорее UX-нюанс. | catalog/employees | session-2026-05-12 |
| F82 | 🟡 | open | **Подтверждено на бэке**: PATCH product `kcal=-100, protein=-50` → 200 OK, отрицательные значения сохраняются. Нужны `@PositiveOrZero` на kcal/protein/fat/carbs/grossWeight/netWeight в Create/UpdateProductRequest. Severity повышена до Major (бэк-валидация недостаёт). | catalog/products | session-2026-05-12 |
| F83 | 🟢 | open | **Подтверждено в коде**: `useState("")` для color в ProductEditPage без `pattern="^#[0-9A-Fa-f]{6}$"`. Принимает любой текст. Фикс: input pattern на фронте + регекс-валидация на бэке. | catalog/products | session-2026-05-12 |
| F87 | 🟢 | open | **Подтверждено**: `ProductViewPage.tsx` не содержит блока «Категория»/`category_id`/`category_name` (grep пустой). Поле есть в response, но не отрисовано в карточке. Простой UI-фикс. | catalog/products | session-2026-05-12 |
| F88 | 🟢 | open | **Подтверждено**: `CategoriesPage.tsx:506` и `ProductListPage.tsx:388` оборачивают `<strong>"{deleteTarget?.name}"</strong>` — если имя уже с кавычками, получаются двойные. Зависит от фикса F79 (санитизация на бэке) либо escape на фронте. | catalog | session-2026-05-12 |
| F89 | 🟢 | open | **Подтверждено**: `stores/EditPage.tsx:262-264` — placeholder `<option value="">По умолчанию (дефолтный)</option>` + цикл по prelist'ам который добавляет `" (дефолтный)"` к `is_default=true`. Две опции с одним суффиксом. Фикс: убрать суффикс у placeholder или у реального default. Тривиально. | stores/edit | session-2026-05-12 |
| F97 | 🟢 | verify-needed | grep `history.back` и `navigate(-1)` в roles/employees пуст — возможно фикс уже был. Нужна UI-проверка через клик «Отмена» в форме создания роли. | employees/roles | session-2026-05-12 |
| F101 | 🟢 | open | **Подтверждено в коде**: `ProductEditPage.tsx:253,264` — кнопка «Улучшить с помощью AI» есть. Реальный handler не проверен (требует UI клик). Скорее всего заглушка — фича не реализована. Скрыть кнопку под feature-flag или реализовать через openclaw-agent. | catalog/products | session-2026-05-12 |
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
| F2 | ⚪ | retracted | Тестовые данные — не баг кода. Можно поправить через UI/seed. | test-data | session-2026-05-05 |
| F5 | ⚪ | retracted | Documentation rot — не баг кода. Обновляется в рабочем порядке через obsidian_erp. | docs | session-2026-05-05 |
| F7 | ⚪ | retracted | Тестовые данные — не баг кода. | test-data | session-2026-05-05 |
| F33 | 🟢 | fixed | Модификатор: max=999999 без верхнего предела | catalog/modifiers | session-2026-05-05 |
| F34 | 🟢 | fixed | Модификатор: можно создать без `options[]` | catalog/modifiers | session-2026-05-05 |
| F35 | 🟢 | fixed | Товар: `is_open_price=true` + `is_by_weight=true` одновременно (mutex отсутствует) | catalog/products | session-2026-05-05 |
| F39 | ⚪ | fixed | ~~Стоп-лист: принимает `reason: null`~~ — by-design, поле optional (нет @NotBlank в DTO). Подтверждено 2026-05-13: POST с reason=null → 201 accepted. Не блокер. | catalog/stop-lists | session-2026-05-05 |
| F42 | 🟢 | open | UX: при `max_amount` опций — нет подсказки на POS | pos/modifiers | session-2026-05-05 |
| F45 | 🟢 | retracted | DELETE modifier-group оставляет orphan-записи в `price-list/modifier-items` | catalog/modifiers | session-2026-05-05 |
| F46 | 🟢 | open | `POST /modifier-groups`: `type` принимает любую строку (нет enum-валидации). 2026-05-13: GET endpoint → 404 через BFF, требуется проверка path/permission через /modifiers route в BFF. Финдинг остаётся открытым. | catalog/modifiers | session-2026-05-06 |
| F47 | 🟢 | verify-needed | UI-симптом, требует воспроизведения с удалёнными модификаторами (на проде сейчас 4 активных модификатора, без удалённых). Без deep UI-теста не подтвердить. | catalog/modifiers/ui | session-2026-05-06 |
| F48 | 🟢 | verify-needed | На admin сейчас тоже видно «Роль: —» (видел в snapshot главной страницы). Возможно подтверждается. Но `/me` для админа отдаёт scope=`all_franchise` (не legacy role). Это UI-косметика. Низкая severity. | dashboard/ui | session-2026-05-06 |
| F49 | 🟢 | open | `/admin/warehouse` (без подраздела) → молчаливый редирект на dashboard, ни 404, ни «выберите подраздел» | warehouse/ui | session-2026-05-06 |
| F50 | 🟡 | open | На POS активные заказы не сортируются по `created_at` — свежий заказ попадает в конец списка вместо верха | pos/orders | session-2026-05-06-pos |
| F51 | 🟡 | open | На POS вкладка «Готовы» смешивает `ready` (готов, не оплачен) и `closed` (оплачен и выдан). `handed_over` в потоке не используется. Кассир не различает «надо ещё выдать клиенту» vs «уже закрыт» | pos/orders | session-2026-05-06-pos |
| F52 | 🟡 | open | На POS нет выбора способа оплаты (наличка/карта) — всё уходит через кассу 3-в-1, в БД `payment_method: "card"` независимо от фактического канала. Ломает бух-отчёт по выручке | pos/payments | session-2026-05-06-pos |
| F53 | 🟡 | open | **Подтверждено**: `Layout.tsx:410-424` — `<button onClick={handleLogout}>` без `type` атрибута. По HTML default `type="submit"`. Если страница содержит `<form>` где-либо — Enter триггерит submit, ищет ближайшую submit-button → логаут. Фикс: добавить `type="button"`. | admin/ui-cross-cutting | session-2026-05-06-e2e |
| F54 | ⚪ | fixed | ~~PATCH /stores с `is_published: true` silent ignore~~ — by-design: `UpdateStoreRequest` не имеет поля `is_published`, отдельные endpoint'ы `POST /stores/{id}/publish` и `unpublish`. Подтверждено через код. Тот же паттерн F6a/F41. | stores/api | session-2026-05-06-e2e |
| F55 | 🟢 | open | При создании новой ТТ default-прейскурант не привязывается автоматически — `store.price_list_id: null`, надо отдельным PATCH | stores/onboarding | session-2026-05-06-e2e |
| F56 | 🟢 | open | KDS показывает не-кухонные позиции заказа в развёрнутом виде серым с пометкой «без станции» — выглядит как «не сделано», хотя позиция уже `ready`. Лучше «не требует приготовления» / явный ✓ | kds/ui | session-2026-05-06-e2e |
| F57 | 🔴 | retracted | На свежей ТТ нет UI/API для привязки кассы 3-в-1 — на самом деле UI есть в карточке ТТ → вкладка «Интеграции» → «Подключить PayKeeper» | pos/onboarding | session-2026-05-06-e2e |
| F58 | 🟢 | open | POS при отсутствии настройки кассы говорит «проверить подключение кассы 3-в-1», но не подсказывает что нужно: 1) PayKeeper в Интеграциях ТТ 2) Терминал с ФН во вкладке «Терминалы». Кассир/менеджер сам не догадается | pos/onboarding | session-2026-05-06-e2e |
| F59 | 🟢 | open | Список терминалов на карточке ТТ показывает stale-данные после DELETE — удалённый терминал остаётся в UI до перезагрузки страницы | stores/terminals/ui | session-2026-05-06-e2e |
| F61 | 🟡 | open | `product.assembly_time` без явного unit — UI отображает как «N мин», а в order item то же значение приходит как `assembly_time_seconds` (явно секунды). Неясность единиц → таймеры готовности и ожидания клиента могут считаться неверно | catalog/products | session-2026-05-06-coverage |
| F62 | ⚪ | fixed | ~~«Чаевые (Нетмонет)» mock-данные~~ — дубликат [[#F81]] (тот же симптом). By-design по BR 3.2 §R5: «фронт + mock-API готовы, real backend отложен» (waiting webhook от support@netmonet.co). Retracted as duplicate F81. | finance/tips | session-2026-05-06-coverage |
| F63 | 🟢 | open | В «Истории заказов» колонка «Кассир» показывает обрезанный UUID (`e0c0ffee`) вместо имени сотрудника. Должно быть имя по созданному заказу | orders/history | session-2026-05-06-coverage |
| F64 | 🟢 | verify-needed | **Backend сейчас корректен**: «ИП Иванов Партнёр» store_count=0 (=API count), «ООО Франшиза Главное» store_count=5 (=число ТТ). Если симптом «3 vs 1» сохраняется в UI на каком-то ЮЛ — это специфичный case, нужен повторный воспроизвод через UI. На сейчас не воспроизводится через API. | legal-entities | session-2026-05-06-coverage |
| F65 | 🟢 | open | После DELETE товара его запись в `price-list/items` остаётся как orphan с ценой 0 — вотчина видна через API, но в превью меню фильтруется. Тех. долг данных | catalog/price-lists | session-2026-05-06-coverage |
| F66 | 🟡 | open | Текст ошибки «Действие доступно только менеджеру ТТ» — пользователю с ролью «Менеджер», который и есть менеджер. Реально проверка завязана на permission, не на роль — текст вводит в заблуждение | pos/tables | session-2026-05-07-waiter-table |
| F67 | 🔴 | open | После того как официант открыл стол и пробил заказ — имя официанта на столе не появляется автоматически. Менеджер на схеме зала не видит, кто работает за каким столом. Авто-привязки `current_waiter_id` нет | pos/tables, orders/lifecycle | session-2026-05-07-waiter-table |
| F69 | 🔴 | open | Z-отчёт смены формируется без предупреждения о висящих неоплаченных заказах. Стол остаётся `occupied`, заказ `accepted/paid_at:null` переходит между сменами | pos/shift, orders/lifecycle | session-2026-05-07-waiter-table |
| F70 | 🟡 | open | В POS «Назначить официанта» показывает **всех** сотрудников с `pos.access` (Админ, Бариста, Кассир, Менеджер, Повар, Официант). Должны быть только официанты | pos/tables, auth-rbac | session-2026-05-07-waiter-table |
| F71 | 🟢 | open | Резерв стола в админке через нативный `prompt()` — одно поле, без структурированных полей (гость, время, число человек). UI antipattern. Зависит от PROPOSAL-1 (оставлять ли резерв в админке вообще) | admin/stores/tables | session-2026-05-07-waiter-table |
| F72 | 🟡 | open | На KDS у заказа в подписи стола показано **`FB`** (последние 2 hex-символа `table_id`), а не реальный номер стола (№26). Повар не понимает за каким столом готовить | kds, orders/lifecycle | session-2026-05-07-waiter-table |
| F73 | 🟡 | open | Таймер на KDS «через 114 мин» для блюда со временем приготовления 10 — нерелевантное число. Похоже тянется из `yellow_threshold_minutes` станции, а не из `assembly_time`. Связано с F61 (единицы `assembly_time`) | kds, catalog/products | session-2026-05-07-waiter-table |
| F74 | 🟡 | open | На схеме зала всего 2 цвета: зелёный=свободен, красный=занят. Когда заказ готов к выдаче — стол остаётся таким же красным. Официант не видит «у тебя на столе пора брать блюда» при обходе зала. Зависит от PROPOSAL-5 (фаза handover) | pos/tables, orders/lifecycle | session-2026-05-07-waiter-table |
| F75 | 🔴 | open | После того как повар отметил «готово» на последней кухонной позиции — заказ автоматически становится **«Выдан»** в POS. Никто не вручал блюдо клиенту. Audit trail неверный, время реального handover не фиксируется. Связано с F25, F51 — три симптома отсутствующей фазы handover (PROPOSAL-5) | orders/lifecycle, pos/tables | session-2026-05-07-waiter-table |

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

| Severity | Active count | Fixed | Retracted (UI-validation) |
|----------|--------------|-------|---------------------------|
| 🔴 Critical | 24 (F26 поднят) | 7 (BUG-066) | — |
| 🟡 Major | 41 | 3 | 2 (F4, F44) |
| 🟢 Minor | 32 | 4 | 1 (F45) — by-design soft-delete |
| ⚪ Info | 9 | — | — |
| **Total active** | **105** | **14** | **3** |

После UI-регресса 2026-05-06: F32, F44, F45 → retracted (не воспроизводятся через UI, так как поля отсутствуют в форме либо это by-design soft-delete). F4 retracted — это были не SPA-роуты, а API-only пути. Добавлены F47, F48, F49.

**Backlog-проверки 2026-05-06:**
- BUG-066 → fixed (warehouse автосоздаётся при создании ТТ; orphan после DELETE store — отдельное замечание).
- BUG-051 → остаётся open, но симптом уточнён: со стороны прейскуранта silent no-op, со стороны ТТ работает.

**Сессия 2026-05-07 (waiter-table walkthrough):** добавлены F66, F67, F69, F70, F71, F72, F73, F74, F75 (4 Critical, 4 Major, 1 Minor; F68 не использован — отозван без записи) + 5 PROPOSAL'ов в `sessions/2026-05-07-waiter-table-walkthrough.md`. PROPOSAL'ы — архитектурные/UX вопросы, **НЕ фиксить автоматически**. Также подтверждены вчерашние F56, F61 на KDS-стороне.

## Следующий ID для новой находки

- `BUG-NNN` (старая схема, для backlog/regression): следующий **BUG-067** (или BUG-X05 для cross-cutting)
- `F-NN` (наша схема, для e2e сессий): следующий **F76**
