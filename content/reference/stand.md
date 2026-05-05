# Тестовый стенд — справочник

Всё что нужно чтобы быстро войти в стенд: URL, креды, идентификаторы, helper-команды.

## URL

```
Base:           https://erp-test.nirbi.ru
Admin SPA:      https://erp-test.nirbi.ru/admin/
POS SPA:        https://erp-test.nirbi.ru/pos/   (Tauri-десктоп — основной POS)
KDS:            нет web (только Tauri-десктоп)
Health:         GET /api/v1/health
```

**⚠️ pos-bff (`/api/v1/pos/*`) снаружи стенда НЕ доступен** — все запросы к pos-routes падают в catch-all admin-bff. Тестируется только через POS desktop координацию.

## Учётки

| Email | Роль | Scope | Permissions |
|-------|------|-------|-------------|
| `admin@erp.local` | Администратор | `all_franchise` | 50+ (всё) |
| `petr@test.local` | Курьер | 1 ТТ (Арбат) | 4: customers.create_quick, customers.read, orders.read, pos.access |
| `ivan@test.local` | Менеджер ТТ | 3 ТТ (включая Бауманскую — другое ЮЛ) | 41 |
| `anna@test.local` | Кассир | — | admin-login отказан by-design |
| **POS PIN-логин** | — | — | через PIN на POS-desktop |
| **KDS PIN-логин** | — | — | через PIN на KDS-desktop |

> ⚠ **Пароли и PIN-коды не в публичном репо.** Хранятся в `private/creds.md` (исключено через `.gitignore`). Скопируй из `private/creds.md.example` если файла нет, или попроси у тест-лида.

## Идентификаторы (стенда)

### ТТ
| store_id | Название | ЮЛ | Опубл. | price_list_id |
|----------|----------|----|--------|----------------|
| `fe4b54a9-2cc1-458f-9d0e-338bbc51df76` | Арбат-флагман | ООО Франшиза Главное | да | `null` (но default применяется) |
| `47c128b9-1f7d-4f01-b6ad-795795b50e7a` | Партнёр — Бауманская | Шаурма Арбат — Партнёр Сокольники | нет | `null` |
| `6e02dffe-e344-4859-909b-68059cb39385` | Сокольники | ООО Франшиза Главное | нет | `null` |

### Юр.лица
| id | Название | Тип |
|----|----------|-----|
| `10000000-0000-0000-0000-000000000001` | ООО Франшиза Главное | franchise (primary) |
| `c64da7bd-7534-4c75-809b-bcc7e71abe87` | Шаурма Арбат — Партнёр Сокольники | franchisee |

### Кухонные станции
| id | Название | products |
|----|----------|----------|
| `743cd003-7b65-4642-835d-1a1b8e1631e1` | Кухня | 11 |
| `7c573693-53b0-4eae-8793-4a683091baf3` | Бар | 0 |
| `7604bc24-6984-415a-936a-490fa2698333` | Горячий цех | 0 |
| `ed6d8302-ea0d-4d8d-bdf6-fb6fc1670a78` | Холодный цех | 0 |

### Категории
| id | Название |
|----|----------|
| `63c7a446-c73f-45d7-a808-76f219e815ee` | Шаурма |
| `6e6d98a2-b109-47f4-9f0d-50d6f4487410` | Бургеры |
| `cd608646-2495-46b8-8067-7886bd2c9b11` | Снеки |
| `e1ef4f4b-1b40-47ce-9e66-108104c1577e` | Напитки |
| `10c0f0e3-dfe8-4b62-8bb9-f353ea815f58` | Десерты |
| `fccb7038-98fd-4d35-9df6-d...` | Кофе (имеет menu-availability «Завтрак» 07-12) |

### Прейскурант
- Default `5dcc6666-987a-4f22-84e0-7227b2c79c7a` («Базовый») — `stores: []` но применяется неявно

### Товар-«канарейка» для денормализации тестов
- `0478828f-75af-4763-8ba4-37799d2344ce` — **Кола 0.5** — не-кухонная, без menu-availability, всегда видна на POS

## Helper-curl

### Логин и токен
```bash
# Подгрузить пароль из private/creds.md (см. private/) или ввести вручную
ADMIN_EMAIL="admin@erp.local"
ADMIN_PASS="<см. private/creds.md>"

TOKEN=$(curl -sS -X POST https://erp-test.nirbi.ru/api/v1/admin/auth/login \
  -H "Content-Type: application/json" \
  -d "{\"email\":\"$ADMIN_EMAIL\",\"password\":\"$ADMIN_PASS\"}" \
  | node -e "let s='';process.stdin.on('data',d=>s+=d);process.stdin.on('end',()=>console.log(JSON.parse(s).data.access_token))")
H="Authorization: Bearer $TOKEN"
B="https://erp-test.nirbi.ru/api/v1/admin"
```

JWT истекает через 15 мин — пере-логинься скриптом если 401.

### Под другую роль
```bash
PETR=$(curl -sS -X POST $B/auth/login -H "Content-Type: application/json" \
  -d "{\"email\":\"petr@test.local\",\"password\":\"$PETR_PASS\"}" \
  | node -e "...JSON.parse(s).data.access_token...")
```
Пароль `$PETR_PASS` — см. `private/creds.md`.

### Создание тестового объекта (Windows-кириллица)
Bash на Windows плохо считает Content-Length для UTF-8 в `-d`. Используй файл:
```bash
echo '{"name":"тест","type":"dish",...}' > /tmp/body.json
curl -sS -X POST "$B/catalog/products" -H "$H" -H "Content-Type: application/json" \
  --data-binary @/tmp/body.json
```

## Тех долг стенда (нужно почистить тест-лиду)

1. Orphan тест-продукт `f9cfbec8-5dba-4de2-9ee5-453076d1d2e2` («[CHAIN-TEST] Бургер RENAMED») — застрял после F9, не удаляется через API
2. 8 orphan записей в `price_list_modifier_items` от удалённых тестовых групп — F45
3. KDS-устройства зарегистрированы без `kitchen_station_id` (8 устройств) — F3
4. Кофе на станции «Кухня» вместо «Бар» — F7
5. Бар-станция пустая — нет товаров для multi-station тестов (F2)
6. Default-прейскурант не привязан к ТТ через UI (BUG-051) — все 3 ТТ имеют `price_list_id: null`
