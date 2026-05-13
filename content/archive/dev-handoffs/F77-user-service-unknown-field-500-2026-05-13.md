# F77 → user-service возвращает 500 на unknown поле в body

> Контекст для разработчика (и его Claude Code).
> Дата: 2026-05-13. Стенд: **admin.nirbi.ru** (PROD).
> Финдинг записан как F77 «нет валидации ИНН/ОГРН/Email», но **причина другая** — см. ниже.

## TL;DR

POST `/api/v1/admin/legal-entities` (и потенциально **все остальные POST/PATCH user-service**) возвращают **500 INTERNAL_ERROR** на любое unknown поле в JSON body.

Должны возвращать **400 INVALID_REQUEST_BODY** (как делают `erp-store-service` и `erp-catalog-service`).

## Воспроизведение

```bash
curl -sS -X POST https://admin.nirbi.ru/api/v1/admin/legal-entities \
  -H "Authorization: Bearer <admin-token>" \
  -H "Content-Type: application/json" \
  -d '{"name":"X","inn":"12345","ogrn":"9999999","email":"noatsign","type":"franchisee"}'
# → 500 {"error":{"code":"INTERNAL_ERROR","message":"Internal server error"}}
```

Поле `email` — **не из DTO** (DTO ожидает `contact_email`). Уберите `email` — получите корректный 400 VALIDATION_ERROR (с details inn/ogrn/legal_address).

## Cut-by-cut изоляция (выполнено 2026-05-13)

| Body | Status | Code |
|------|--------|------|
| `{}` | 400 | VALIDATION_ERROR (5 details — все @NotBlank поля) |
| `{name,inn:"12345",ogrn:"9999999",type:"franchisee"}` (без unknown) | 400 | VALIDATION_ERROR (inn/ogrn/legalAddress) |
| `{name,inn:"1234567890",ogrn:"...",legal_address:"x",type:"franchise"}` | 422 | INVALID_INN_CHECKSUM |
| `{name,...,email:"noatsign"}` (**unknown `email`**) | **500** | **INTERNAL_ERROR** |
| `{name,...,контактный_номер:"+7..."}` (любое unknown) | вероятно 500 | INTERNAL_ERROR |

## Корневая причина (два связанных дефекта)

### 1. `erp-user-service/src/main/java/com/erp/user/config/JacksonConfig.java`

```java
@Bean @Primary
public ObjectMapper objectMapper() {
    ObjectMapper mapper = new ObjectMapper();   // ← raw Jackson, БЕЗ Spring Boot defaults
    mapper.registerModule(timeModule);
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    return mapper;
}
```

`new ObjectMapper()` стартует с native-defaults Jackson, в т.ч. **`FAIL_ON_UNKNOWN_PROPERTIES = true`**. Когда определяется `@Bean ObjectMapper`, Spring Boot **не применяет свои customizers** к нему. В `erp-store-service` и `erp-catalog-service` Jackson-конфиг другой — там Spring Boot defaults сохраняются (`FAIL_ON_UNKNOWN_PROPERTIES = false`, и unknown поля silent-ignored).

### 2. `erp-user-service/src/main/java/com/erp/user/exception/GlobalExceptionHandler.java`

Нет обработчика `HttpMessageNotReadableException` (parent класса `UnrecognizedPropertyException`). Поэтому Jackson-ошибка уходит в catch-all `@ExceptionHandler(Exception.class) → 500`.

В `erp-store-service/.../GlobalExceptionHandler.java` такой handler есть:
```java
@ExceptionHandler(HttpMessageNotReadableException.class)
public ResponseEntity<ErrorResponse> handleNotReadable(...) {
    return badRequest INVALID_REQUEST_BODY;
}
```

## План фикса (рекомендованный)

**Файлы (одна ветка):**

### `JacksonConfig.java` — переписать через Builder customizer

```java
package com.erp.user.config;

import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import org.springframework.boot.autoconfigure.jackson.Jackson2ObjectMapperBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Configuration
public class JacksonConfig {
    private static final DateTimeFormatter UTC_FORMATTER =
            DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss'Z'");

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer utcDateTimeCustomizer() {
        return builder -> builder
            .serializerByType(LocalDateTime.class, new LocalDateTimeSerializer(UTC_FORMATTER))
            .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    }
}
```

Уберёт `@Bean ObjectMapper`. Spring Boot применит свои дефолты, включая `FAIL_ON_UNKNOWN_PROPERTIES=false`. Все unknown поля будут silent-ignored (стандарт проекта).

### `GlobalExceptionHandler.java` — safety net на любые Jackson-ошибки

```java
import org.springframework.http.converter.HttpMessageNotReadableException;

@ExceptionHandler(HttpMessageNotReadableException.class)
public ResponseEntity<ErrorResponse> handleNotReadable(HttpMessageNotReadableException ex) {
    log.debug("Malformed request body: {}", ex.getMessage());
    return ResponseEntity.badRequest().body(
        ErrorResponse.of("INVALID_REQUEST_BODY",
            "Request body is missing or malformed"));
}
```

## Регресс-сурфейс

- **Корректные body** (всё известные поля, валидные) — поведение **не меняется** (200/201 OK).
- **Невалидные значения** (Pattern/Size/Email не подходят) — поведение **не меняется** (400 VALIDATION_ERROR с details).
- **Unknown поля** — раньше 500, теперь silent-ignore (как в остальных сервисах). **Это улучшение**, не регресс.
- **Malformed JSON** — раньше 500, теперь 400 INVALID_REQUEST_BODY.

## Что F77 закрывает по факту

ВСЕ POST/PATCH endpoint'ы user-service страдают тем же дефектом:
- `/legal-entities` (F77)
- `/employees` (потенциально F102/F103 — нужно перепроверить, возможно та же причина)
- `/shift-templates`, `/salary-formulas`, `/payroll/calculate`, `/roles`, `/schedules`
- любой другой POST/PATCH с unknown полями

Один фикс — потенциально закрывает класс.

## Тест (без БД)

`src/test/java/com/erp/user/integration/JacksonUnknownFieldTest.java`:

```java
@SpringBootTest
@AutoConfigureMockMvc
class JacksonUnknownFieldTest {
    @Autowired MockMvc mvc;
    @Autowired ObjectMapper mapper;

    @Test
    void objectMapper_should_silently_ignore_unknown_fields() {
        // FAIL_ON_UNKNOWN_PROPERTIES must be false (Spring Boot default)
        assertThat(mapper.isEnabled(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES))
            .isFalse();
    }

    @Test
    void post_with_unknown_field_should_return_400_or_silent_ignore_not_500() throws Exception {
        // Любой POST с unknown полем не должен валиться 500
        // (после правильной настройки → silent-ignore + потом 400 VALIDATION_ERROR за нехватку required)
        // ...
    }
}
```

## Деплой

Один сервис: `erp-store-service`... нет, **`erp-user-service`**. `gh workflow run -R nearbyErp/erp-user-service deploy.yml` (по аналогии с BUG-026).

## После раскатки

1. Регресс на проде: POST `/legal-entities` с unknown полем `email` → ожидаем 400 (либо 201/422/409 в зависимости от остальных данных, но не 500).
2. Регресс F102/F103 (PATCH employees 500) — возможно та же причина, проверить.
3. Если F102/F103 тоже починен — обновить findings.md.

## Ссылки

- [[D:/Project/ERP/test-base/findings|F77 в findings.md]]
- `BUG-026` (паттерн нашего предыдущего фикса в `erp-store-service`)
- `feedback_read_service_not_signature` (memory) — урок F95
