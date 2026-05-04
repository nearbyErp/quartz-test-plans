---
title: ERP Test Plans
---

# ERP Test Plans

Документация ручного тестирования платформы **Obsidian ERP**.

> Этот сайт — навигатор: куда смотреть, что делать, где искать. Сами баги — в Google Docs (трекер команды).

---

## Кто ты?

### 🧪 Я тестировщик

Открой свою зону:

- **[Тестер 1 (T1) — Доступы и структура](testers/T1/)**  
  Авторизация, Франшизы, Юр.лица, Роли, Сотрудники-CRUD. Владелец RBAC-матрицы.
- **[Тестер 2 (T2) — HR + Каталог](testers/T2/)**  
  Расписание, Time tracking, Зарплата, Активность, и весь Каталог (товары/категории/модификаторы/прейскуранты/стоп-листы/техкарты).
- **[Тестер 3 (T3) — Операционный поток](testers/T3/)**  
  ТТ, Меню ТТ, Столы, Склад, Заказы, Aggregator integrations.

Не знаешь свою зону — спроси тест-лида.

### 🛠 Я разработчик

Мне дали баг (например `BUG-013`):

1. Открой [Известные баги](04-known-bugs-index.md) → найди по ID контекст, severity, тип
2. Если баг про RBAC → [RBAC matrix](01-rbac-matrix.md) расписывает «что должно быть по спеке» в той ячейке
3. Источник правды по бизнес-логике — [dev-документация](https://nearbyerp.github.io/quartz-site/)

### 📊 Я PM / тест-лид

- [Известные баги](04-known-bugs-index.md) — сводка 70 багов: разнесены по T1/T2/T3, severity, типу
- [Smoke pass](03-smoke-pass.md) — 30-мин круг для проверки стенда перед циклом
- [RBAC matrix](01-rbac-matrix.md) — основной regression-артефакт

---

## Общие артефакты (используют все)

| Артефакт | Когда нужен |
|----------|-------------|
| [01 RBAC Matrix](01-rbac-matrix.md) | Каждый regression-цикл. Владелец — T1, используют все при проверке RBAC своих разделов |
| [02 Form Validation Checklist](02-form-validation-checklist.md) | Применяется к **каждой форме** во всех зонах. Прогон при первом тестировании формы + после фиксов |
| [03 Smoke Pass](03-smoke-pass.md) | **Первое действие** в каждом цикле тестирования. 30 минут. Если упало — стоп, репорт blocker |
| [04 Known Bugs Index](04-known-bugs-index.md) | Перед заведением нового бага — поиск дублей. Также общая картина для PM |

## Шаблоны

- [Bug Report Template](00-templates/bug-report-template.md) — формат для записи в Google Docs
- [Test Case Template](00-templates/test-case-template.md) — формат для новых test cases

---

## Где что живёт

| Что | Где |
|-----|-----|
| Сами баги (статусы, история) | Google Docs (трекер команды) |
| Индекс багов для поиска | Здесь, [04-known-bugs-index](04-known-bugs-index.md) |
| Тест-кейсы по тестировщикам | Здесь, [папки тестировщиков](testers/T1/) |
| Доки бизнеса и архитектуры ERP | https://nearbyerp.github.io/quartz-site/ |
| Тест-стенд | https://erp-test.nirbi.ru |

## Соглашения по ID

| Сущность | Формат | Пример |
|----------|--------|--------|
| Test case | `TC-{T#}-{AREA}-{NNN}` | `TC-T1-AUTH-001` |
| Bug | `BUG-{NNN}` | `BUG-042` |
| Test run | `RUN-{YYYY-MM-DD}-{N}` | `RUN-2026-05-04-1` |

Полные конвенции и AREA-коды — в [README репозитория](https://github.com/nearbyErp/quartz-test-plans/blob/main/README.md).
