---
title: ERP Test Base
---

# ERP — Тестовая база

Рабочее пространство тестирования цепочки Admin → POS → KDS платформы Obsidian ERP. Не для разработчиков (для них — `archive/BUGS-FOR-DEV*.md`), а для нашей с тобой работы.

## Быстрый доступ

| Куда | Зачем |
|------|-------|
| [findings.md](findings.md) | Все известные баги (BUG-NNN backlog + F-NN наши, единая таблица) |
| [todo.md](todo.md) | Что хочется покрыть в следующий раз |
| [sessions/](sessions/) | Хронология прогонов |
| [zones/](zones/) | Тест-кейсы по доменам приложения |
| [scenarios/](scenarios/) | Сквозные e2e сценарии (ссылаются на zones) |
| [checklists/](checklists/) | Универсальные чек-листы (smoke, form-validation) |
| [reference/stand.md](reference/stand.md) | URL, креды, тестовые id, helper-curl |
| [reference/endpoints.md](reference/endpoints.md) | Карта реальных API endpoints |
| [templates/](templates/) | Шаблоны новых TC, багов, сессий |
| [archive/](archive/) | Старое: BUGS-FOR-DEV* для коллеги, оригиналы файлов |

## Структура

```
test-plans/
├── index.md                     ← ты здесь
├── findings.md                  все баги
├── todo.md                      план дальнейшего
├── sessions/                    хронология (один файл = одна сессия)
├── zones/                       тест-кейсы по доменам
│   ├── catalog/
│   ├── orders/
│   ├── payments/
│   ├── kds/
│   ├── pos/
│   └── auth-rbac/
├── scenarios/                   e2e сценарии (списки ссылок на TC)
├── checklists/                  универсальные (smoke, form-validation)
├── reference/                   стенд + endpoints
├── templates/                   шаблоны
└── archive/                     старое
```

## Как мы работаем

### Начало сессии
1. Открыть `index.md` (этот файл) — посмотреть последнюю сессию + todo
2. `reference/stand.md` — обновить токен если истёк
3. Скопировать `templates/session.md` → `sessions/YYYY-MM-DD-<тип>.md`
4. По ходу прогона — заполнять сессию (что делал, что нашёл)

### Найденный баг
1. Присвоить ID: следующий `F46` (см. `findings.md`)
2. Добавить строку в `findings.md` (severity, status=open, краткое описание, zone, source=session)
3. Подробности — внутри сессии (репро + ожидание + факт + скрин-фрагмент)
4. Если кейс относится к конкретному zones-файлу — добавить ссылку на F-номер в нём

### Прогон сценария
1. Открыть `scenarios/<сценарий>.md` — список ссылок на TC из zones
2. Идти по списку, проставлять `pass / fail / blocked / skipped` в чек-листе сценария
3. Падающие → завести как findings
4. По итогу сессии — обновить `todo.md` (что дальше)

### Регресс по фиксу
1. Найти баг в `findings.md` — поставить status=`verify-needed`
2. Прогнать соответствующий TC из `zones/<zone>/<feature>.md`
3. Если фикс работает → status=`fixed`. Если нет → описать в новой сессии что не сошлось

## Конвенции

### ID
- **`BUG-NNN`** — backlog regression-баги. Следующий: `BUG-067`. Cross-cutting: `BUG-X05+`
- **`F-NN`** — наши e2e-сессионные. Следующий: `F46`
- **`TC-CHAIN-NNN`** — кейсы из сценария chain-admin-pos-kds (legacy). Новые TC именуем `TC-<ZONE>-NNN` (например `TC-CATALOG-001`)
- **Сессии:** `YYYY-MM-DD-<тип>` (типы: `e2e`, `regression`, `smoke`, `exploratory`, `onboarding`)

### Severity
- 🔴 **Critical** — блокирует операцию, потеря данных, security-leak
- 🟡 **Major** — функциональный баг, missing feature, важное расхождение со спекой
- 🟢 **Minor** — UX, валидация без блокера, doc-rot
- ⚪ **Info / by-design** — наблюдение, не баг

### Status
- `open` — нашли, не фиксили
- `in-progress` — разработчик в работе
- `verify-needed` — фикс задеплоен, нужен прогон регресса
- `fixed` — подтверждён фикс
- `retracted` — отозван (не баг при детальном разборе)

## Текущий статус (на 2026-05-05)

- Сессий проведено: **1** ([2026-05-05-e2e](sessions/2026-05-05-e2e.md))
- Покрытие: ~25% запланированного scope `chain-admin-pos-kds`
- Active findings: **100** (27 Critical, 41 Major, 32 Minor)
- Что в плане далее: см. [todo.md](todo.md)
