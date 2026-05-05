# Auth-RBAC · Login

Email/password логин в admin BFF и POS BFF. JWT, refresh, logout.

## Endpoints

```
POST /api/v1/admin/auth/login           email + password → JWT (HS256, exp 15min)
POST /api/v1/admin/auth/refresh         refresh_token → новый JWT
GET  /api/v1/admin/auth/me              текущий user
POST /api/v1/admin/auth/logout
POST /api/v1/admin/auth/forgot-password
POST /api/v1/admin/auth/reset-password
POST /api/v1/pos/auth/login             email + password (POS-web; недоступно снаружи стенда)
```

## JWT payload

```json
{
  "sub": "<user_id>",
  "franchise_id": "00000000-0000-0000-0000-000000000001",
  "role_ids": ["..."],
  "iat": 1777978913,
  "exp": 1777979813
}
```

Алгоритм: HS256. Exp = 15 минут (`expires_in: 900`).
Refresh token — opaque UUID (не JWT).

## Findings

(пока пусто)

## Тест-кейсы

### TC-AUTH-LOGIN-001 — Валидный логин
**Status:** ✅ pass · session 2026-05-05
admin@erp.local + правильный пароль (см. `private/creds.md`) → 200, токен валиден

### TC-AUTH-LOGIN-002 — Неверный пароль
**Status:** ◯ todo
Ответ: `INVALID_CREDENTIALS` 401

### TC-AUTH-LOGIN-003 — Несуществующий email
**Status:** ◯ todo

### TC-AUTH-LOGIN-EXPIRED — JWT истёк
**Status:** ✅ partial (наблюдали в session)
Любой запрос → 401 «Invalid token». Нужно перезалогиниться.

### TC-AUTH-LOGIN-REFRESH — Refresh token
**Status:** ◯ todo
POST /auth/refresh с refresh_token → новый JWT.

### TC-AUTH-LOGIN-LOGOUT — Logout инвалидирует токен
**Status:** ◯ todo
POST /auth/logout → следующий запрос с тем же JWT → 401.

### TC-AUTH-LOGIN-CASHIER — Cashier admin-login отказан (F12)
**Status:** ✅ pass by-design
anna@test.local + любой пароль (см. `private/creds.md`) → INVALID_CREDENTIALS.
