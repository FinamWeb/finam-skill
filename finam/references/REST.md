# Finam Trade-API REST

Токены и аутентификация в Finam Trade API (REST)

1. Получение ключа:
Сгенерируйте постоянный секретный ключ в разделе «Токены» вашего профиля.

2. Получение JWT:
Отправьте POST-запрос к методу `Auth`, передав секретный ключ в теле запроса. В ответ сервер вернет временный JWT.

3. Использование в заголовках:
Добавляйте заголовок во все последующие HTTP-запросы: `Authorization: Bearer <ваш_jwt_токен>`. Без этого заголовка сервер отклонит запрос.

4. Управление жизненным циклом:
В REST отсутствует механизм автоматической подписки на обновления. Для получения свежего токена вам необходимо повторно вызывать метод `Auth` вручную до того, как текущий JWT станет недействительным. Разработчик должен самостоятельно реализовать логику контроля времени жизни токена в коде.

**Server:** https://api.finam.ru

---

## AuthService

- [POST /v1/sessions](#получение-jwt-токена-из-api-токена)
- [POST /v1/sessions/details](#получение-информации-о-токене-сессии)

---

### Получение JWT токена из API токена

`POST /v1/sessions`

Все поля передаются в теле запроса.

**Body** (required, application/json)

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| secret | string | yes | API токен (secret key) |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl https://api.finam.ru/v1/sessions \
  --request POST \
  --header 'Content-Type: application/json' \
  --header 'Authorization: YOUR_SECRET_TOKEN' \
  --data '{
  "secret": ""
}'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "token": {
      "type": "string",
      "title": "Полученный JWT-токен"
    }
  },
  "title": "Информация об авторизации"
}
```

---

### Получение информации о токене сессии

`POST /v1/sessions/details`

Токен передается в теле запроса для безопасности. Получение информации о токене. Также включает список доступных счетов.

**Body** (required, application/json)

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| token | string | yes | JWT-токен |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl https://api.finam.ru/v1/sessions/details \
  --request POST \
  --header 'Content-Type: application/json' \
  --header 'Authorization: YOUR_SECRET_TOKEN' \
  --data '{
  "token": ""
}'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "created_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время создания"
    },
    "expires_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время экспирации"
    },
    "md_permissions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "quote_level": {
            "type": "string",
            "enum": [
              "QUOTE_LEVEL_UNSPECIFIED",
              "QUOTE_LEVEL_LAST_PRICE",
              "QUOTE_LEVEL_BEST_BID_OFFER",
              "QUOTE_LEVEL_DEPTH_OF_MARKET",
              "QUOTE_LEVEL_DEPTH_OF_BOOK",
              "QUOTE_LEVEL_ACCESS_FORBIDDEN"
            ],
            "default": "QUOTE_LEVEL_UNSPECIFIED",
            "description": "- QUOTE_LEVEL_UNSPECIFIED: Значение не указано\n - QUOTE_LEVEL_LAST_PRICE: Последняя цена\n - QUOTE_LEVEL_BEST_BID_OFFER: Бид аск\n - QUOTE_LEVEL_DEPTH_OF_MARKET: Агрегированный стакан\n - QUOTE_LEVEL_DEPTH_OF_BOOK: Полный стакан\n - QUOTE_LEVEL_ACCESS_FORBIDDEN: Доступ запрещен",
            "title": "Уровень котировок"
          },
          "delay_minutes": {
            "type": "integer",
            "format": "int32",
            "title": "Задержка в минутах"
          },
          "mic": {
            "type": "string",
            "title": "Идентификатор биржи mic"
          },
          "country": {
            "type": "string",
            "title": "Страна"
          },
          "continent": {
            "type": "string",
            "title": "Континент"
          },
          "worldwide": {
            "type": "boolean",
            "title": "Весь мир"
          }
        },
        "title": "Информация о доступе к рыночным данным"
      },
      "title": "Информация о доступе к рыночным данным"
    },
    "account_ids": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "title": "Идентификаторы аккаунтов"
    },
    "readonly": {
      "type": "boolean",
      "title": "Сессия и торговые счета в токене будут помечены readonly"
    }
  },
  "title": "Информация о токене"
}
```

---

## AccountsService

- [GET /v1/accounts/{accountId}](#получение-информации-по-конкретному-аккаунту)
- [GET /v1/accounts/{accountId}/trades](#получение-истории-по-сделкам-аккаунта)
- [GET /v1/accounts/{accountId}/transactions](#получение-списка-транзакций-аккаунта)

---

### Получение информации по конкретному аккаунту

`GET /v1/accounts/{accountId}`

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| accountId | string | yes | Идентификатор аккаунта |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/accounts/{accountId}' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "account_id": {
      "type": "string",
      "title": "Идентификатор аккаунта"
    },
    "type": {
      "type": "string",
      "title": "Тип аккаунта"
    },
    "status": {
      "type": "string",
      "title": "Статус аккаунта"
    },
    "equity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "unrealized_profit": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "positions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "symbol": {
            "type": "string",
            "title": "Символ инструмента"
          },
          "quantity": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "average_price": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "current_price": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "maintenance_margin": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "daily_pnl": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "unrealized_pnl": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          }
        },
        "title": "Информация о позиции"
      },
      "title": "Позиции. Открытые, плюс теоретические (по неисполненным активным заявкам)"
    },
    "cash": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "currency_code": {
            "type": "string",
            "description": "The three-letter currency code defined in ISO 4217."
          },
          "units": {
            "type": "string",
            "format": "int64",
            "description": "The whole units of the amount.\nFor example if `currencyCode` is `\"USD\"`, then 1 unit is one US dollar."
          },
          "nanos": {
            "type": "integer",
            "format": "int32",
            "description": "Number of nano (10^-9) units of the amount.\nThe value must be between -999,999,999 and +999,999,999 inclusive.\nIf `units` is positive, `nanos` must be positive or zero.\nIf `units` is zero, `nanos` can be positive, zero, or negative.\nIf `units` is negative, `nanos` must be negative or zero.\nFor example $-1.75 is represented as `units`=-1 and `nanos`=-750,000,000."
          }
        },
        "description": "Represents an amount of money with its currency type."
      },
      "description": "Сумма собственных денежных средств на счете, доступная для торговли. Не включает маржинальные средства."
    },
    "portfolio_mc": {
      "type": "object",
      "properties": {
        "available_cash": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "initial_margin": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "maintenance_margin": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        }
      },
      "description": "Общий тип для счетов Московской Биржи. Включает в себя как единые, так и специализированные (моно) счета для разных секций биржи.\nЕдиный торговый счет (ЕТС): Позволяет торговать на нескольких рынках (фондовый, валютный. срочный, spb, иностранные бумаги, иностранные фьючерсы) с единой денежной позиции.\nМоно-счет фондового рынка MOEX: Изолированный счет для торговли акциями, облигациями и паями.\nМоно-счет валютного рынка MOEX: Изолированный счет для операций с валютными парами (например, CNYRUB_TOM)."
    },
    "portfolio_mct": {
      "type": "object",
      "description": "Тип портфеля для счетов на американских рынках.\nПредоставляет доступ к биржам США: NYSE, NASDAQ, CBOE, CME, сделки с американскими акциями, фьючерсами и опционами."
    },
    "portfolio_forts": {
      "type": "object",
      "properties": {
        "available_cash": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "money_reserved": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        }
      },
      "description": "Тип портфеля для торговли на срочном рынке Московской Биржи.\nПредназначен для работы с производными финансовыми инструментами: фьючерсами и опционами."
    },
    "open_account_date": {
      "type": "string",
      "format": "date-time",
      "title": "Дата открытия счета"
    },
    "first_trade_date": {
      "type": "string",
      "format": "date-time",
      "title": "Дата первой торговой транзакции"
    },
    "first_non_trade_date": {
      "type": "string",
      "format": "date-time",
      "title": "Дата первой неторговой транзакции"
    }
  },
  "title": "Информация о конкретном аккаунте"
}
```

---

### Получение истории по сделкам аккаунта

`GET /v1/accounts/{accountId}/trades`

Параметры:
- `accountId` — передается в URL пути
- `limit` и `interval` — передаются как query-параметры

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| accountId | string | yes | Идентификатор аккаунта |

**Query Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| limit | integer (int32) | no | Лимит количества сделок |
| interval.start_time | string (date-time) | no | Inclusive start of the interval. If specified, a Timestamp matching this interval will have to be the same or after the start. |
| interval.end_time | string (date-time) | no | Exclusive end of the interval. If specified, a Timestamp matching this interval will have to be before the end. |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 400 | Неверно передан интервал |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/accounts/{accountId}/trades?limit=1&interval.start_time=&interval.end_time=' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "trades": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "trade_id": {
            "type": "string",
            "title": "Идентификатор сделки"
          },
          "symbol": {
            "type": "string",
            "title": "Символ инструмента"
          },
          "price": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "size": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "side": {
            "type": "string",
            "enum": [
              "SIDE_UNSPECIFIED",
              "SIDE_BUY",
              "SIDE_SELL"
            ],
            "default": "SIDE_UNSPECIFIED",
            "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
            "title": "Сторона сделки"
          },
          "timestamp": {
            "type": "string",
            "format": "date-time",
            "title": "Метка времени"
          },
          "order_id": {
            "type": "string",
            "title": "Идентификатор заявки"
          },
          "account_id": {
            "type": "string",
            "title": "Идентификатор аккаунта"
          },
          "comment": {
            "type": "string",
            "title": "Метка заявки. (максимум 128 символов)"
          }
        },
        "title": "Информация о сделке"
      },
      "title": "Сделки по аккаунту"
    }
  },
  "title": "История по сделкам"
}
```

---

### Получение списка транзакций аккаунта

`GET /v1/accounts/{accountId}/transactions`

Параметры:
- `accountId` — передается в URL пути
- `limit` и `interval` — передаются как query-параметры

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| accountId | string | yes | Идентификатор аккаунта |

**Query Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| limit | integer (int32) | no | Лимит количества транзакций |
| interval.start_time | string (date-time) | no | Inclusive start of the interval. If specified, a Timestamp matching this interval will have to be the same or after the start. |
| interval.end_time | string (date-time) | no | Exclusive end of the interval. If specified, a Timestamp matching this interval will have to be before the end. |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 400 | Неверно передан интервал |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/accounts/{accountId}/transactions?limit=1&interval.start_time=&interval.end_time=' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "transactions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string",
            "title": "Идентификатор транзакции"
          },
          "category": {
            "type": "string",
            "title": "Тип транзакции из TransactionCategory"
          },
          "timestamp": {
            "type": "string",
            "format": "date-time",
            "title": "Метка времени"
          },
          "symbol": {
            "type": "string",
            "title": "Символ инструмента"
          },
          "change": {
            "type": "object",
            "properties": {
              "currency_code": {
                "type": "string",
                "description": "The three-letter currency code defined in ISO 4217."
              },
              "units": {
                "type": "string",
                "format": "int64",
                "description": "The whole units of the amount.\nFor example if `currencyCode` is `\"USD\"`, then 1 unit is one US dollar."
              },
              "nanos": {
                "type": "integer",
                "format": "int32",
                "description": "Number of nano (10^-9) units of the amount.\nThe value must be between -999,999,999 and +999,999,999 inclusive.\nIf `units` is positive, `nanos` must be positive or zero.\nIf `units` is zero, `nanos` can be positive, zero, or negative.\nIf `units` is negative, `nanos` must be negative or zero.\nFor example $-1.75 is represented as `units`=-1 and `nanos`=-750,000,000."
              }
            },
            "description": "Represents an amount of money with its currency type."
          },
          "trade": {
            "type": "object",
            "properties": {
              "size": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "price": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "accrued_interest": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              }
            },
            "title": "Объект заполняется для торговых типов транзакций"
          },
          "transaction_category": {
            "type": "string",
            "enum": [
              "OTHERS",
              "DEPOSIT",
              "WITHDRAW",
              "INCOME",
              "COMMISSION",
              "TAX",
              "INHERITANCE",
              "TRANSFER",
              "CONTRACT_TERMINATION",
              "OUTCOMES",
              "FINE",
              "LOAN"
            ],
            "default": "OTHERS",
            "description": "Категории транзакции.\n\n - OTHERS: Прочее\n - DEPOSIT: Ввод ДС\n - WITHDRAW: Вывод ДС\n - INCOME: Доход\n - COMMISSION: Комиссия\n - TAX: Налог\n - INHERITANCE: Наследство\n - TRANSFER: Перевод ДС\n - CONTRACT_TERMINATION: Расторжение договора\n - OUTCOMES: Расходы\n - FINE: Штраф\n - LOAN: Займ"
          },
          "transaction_name": {
            "type": "string",
            "title": "Наименование транзакции"
          },
          "change_qty": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          }
        },
        "title": "Информация о транзакции"
      },
      "title": "Транзакции по аккаунту"
    }
  },
  "title": "Список транзакций"
}
```

---

## OrdersService

- [GET /v1/accounts/{accountId}/orders](#получение-списка-заявок-для-аккаунта)
- [POST /v1/accounts/{accountId}/orders](#выставление-биржевой-заявки)
- [GET /v1/accounts/{accountId}/orders/{orderId}](#получение-информации-о-конкретном-ордере)
- [DELETE /v1/accounts/{accountId}/orders/{orderId}](#отмена-биржевой-заявки)
- [POST /v1/accounts/{accountId}/sltp-orders](#выставление-sltp-заявки)

---

### Получение списка заявок для аккаунта

`GET /v1/accounts/{accountId}/orders`

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| accountId | string | yes | Идентификатор аккаунта |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/accounts/{accountId}/orders' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "orders": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "order_id": {
            "type": "string",
            "title": "Идентификатор заявки"
          },
          "exec_id": {
            "type": "string",
            "title": "Идентификатор исполнения"
          },
          "status": {
            "type": "string",
            "enum": [
              "ORDER_STATUS_UNSPECIFIED",
              "ORDER_STATUS_NEW",
              "ORDER_STATUS_PARTIALLY_FILLED",
              "ORDER_STATUS_FILLED",
              "ORDER_STATUS_DONE_FOR_DAY",
              "ORDER_STATUS_CANCELED",
              "ORDER_STATUS_REPLACED",
              "ORDER_STATUS_PENDING_CANCEL",
              "ORDER_STATUS_REJECTED",
              "ORDER_STATUS_SUSPENDED",
              "ORDER_STATUS_PENDING_NEW",
              "ORDER_STATUS_EXPIRED",
              "ORDER_STATUS_FAILED",
              "ORDER_STATUS_FORWARDING",
              "ORDER_STATUS_WAIT",
              "ORDER_STATUS_DENIED_BY_BROKER",
              "ORDER_STATUS_REJECTED_BY_EXCHANGE",
              "ORDER_STATUS_WATCHING",
              "ORDER_STATUS_EXECUTED",
              "ORDER_STATUS_DISABLED",
              "ORDER_STATUS_LINK_WAIT",
              "ORDER_STATUS_SL_GUARD_TIME",
              "ORDER_STATUS_SL_EXECUTED",
              "ORDER_STATUS_SL_FORWARDING",
              "ORDER_STATUS_TP_GUARD_TIME",
              "ORDER_STATUS_TP_EXECUTED",
              "ORDER_STATUS_TP_CORRECTION",
              "ORDER_STATUS_TP_FORWARDING",
              "ORDER_STATUS_TP_CORR_GUARD_TIME"
            ],
            "default": "ORDER_STATUS_UNSPECIFIED",
            "description": "- ORDER_STATUS_UNSPECIFIED: Неопределенное значение\n - ORDER_STATUS_NEW: Новая заявка\n - ORDER_STATUS_PARTIALLY_FILLED: Частично исполненная\n - ORDER_STATUS_FILLED: Исполненная\n - ORDER_STATUS_DONE_FOR_DAY: Действует в течение дня\n - ORDER_STATUS_CANCELED: Отменена\n - ORDER_STATUS_REPLACED: Заменена на другую\n - ORDER_STATUS_PENDING_CANCEL: Ожидает отмены\n - ORDER_STATUS_REJECTED: Отклонена\n - ORDER_STATUS_SUSPENDED: Приостановлена\n - ORDER_STATUS_PENDING_NEW: В ожидании новой\n - ORDER_STATUS_EXPIRED: Истекла\n - ORDER_STATUS_FAILED: Ошибка\n - ORDER_STATUS_FORWARDING: Пересылка\n - ORDER_STATUS_WAIT: Ожидает\n - ORDER_STATUS_DENIED_BY_BROKER: Отклонено брокером\n - ORDER_STATUS_REJECTED_BY_EXCHANGE: Отклонено биржей\n - ORDER_STATUS_WATCHING: Наблюдение\n - ORDER_STATUS_EXECUTED: Исполнена\n - ORDER_STATUS_DISABLED: Отключена\n - ORDER_STATUS_LINK_WAIT: Ожидание ссылки\n - ORDER_STATUS_SL_GUARD_TIME: Защитное время SL\n - ORDER_STATUS_SL_EXECUTED: Исполнена по SL\n - ORDER_STATUS_SL_FORWARDING: Пересылка SL\n - ORDER_STATUS_TP_GUARD_TIME: Защитное время TP\n - ORDER_STATUS_TP_EXECUTED: Исполнена по TP\n - ORDER_STATUS_TP_CORRECTION: Коррекция TP\n - ORDER_STATUS_TP_FORWARDING: Пересылка TP\n - ORDER_STATUS_TP_CORR_GUARD_TIME: Коррекция TP в защитное время",
            "title": "Статус заявки"
          },
          "order": {
            "type": "object",
            "properties": {
              "account_id": {
                "type": "string",
                "title": "Идентификатор аккаунта"
              },
              "symbol": {
                "type": "string",
                "title": "Символ инструмента"
              },
              "quantity": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "side": {
                "type": "string",
                "enum": [
                  "SIDE_UNSPECIFIED",
                  "SIDE_BUY",
                  "SIDE_SELL"
                ],
                "default": "SIDE_UNSPECIFIED",
                "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
                "title": "Сторона сделки"
              },
              "type": {
                "type": "string",
                "enum": [
                  "ORDER_TYPE_UNSPECIFIED",
                  "ORDER_TYPE_MARKET",
                  "ORDER_TYPE_LIMIT",
                  "ORDER_TYPE_STOP",
                  "ORDER_TYPE_STOP_LIMIT",
                  "ORDER_TYPE_MULTI_LEG"
                ],
                "default": "ORDER_TYPE_UNSPECIFIED",
                "description": "- ORDER_TYPE_UNSPECIFIED: Значение не указано\n - ORDER_TYPE_MARKET: Рыночная\n - ORDER_TYPE_LIMIT: Лимитная\n - ORDER_TYPE_STOP: Стоп заявка рыночная\n - ORDER_TYPE_STOP_LIMIT: Стоп заявка лимитная\n - ORDER_TYPE_MULTI_LEG: Мульти лег заявка",
                "title": "Тип заявки"
              },
              "time_in_force": {
                "type": "string",
                "enum": [
                  "TIME_IN_FORCE_UNSPECIFIED",
                  "TIME_IN_FORCE_DAY",
                  "TIME_IN_FORCE_GOOD_TILL_CANCEL",
                  "TIME_IN_FORCE_GOOD_TILL_CROSSING",
                  "TIME_IN_FORCE_EXT",
                  "TIME_IN_FORCE_ON_OPEN",
                  "TIME_IN_FORCE_ON_CLOSE",
                  "TIME_IN_FORCE_IOC",
                  "TIME_IN_FORCE_FOK"
                ],
                "default": "TIME_IN_FORCE_UNSPECIFIED",
                "description": "- TIME_IN_FORCE_UNSPECIFIED: Значение не указано\n - TIME_IN_FORCE_DAY: До конца дня\n - TIME_IN_FORCE_GOOD_TILL_CANCEL: Действителен до отмены\n - TIME_IN_FORCE_GOOD_TILL_CROSSING: Действителен до пересечения\n - TIME_IN_FORCE_EXT: Внебиржевая торговля\n - TIME_IN_FORCE_ON_OPEN: На открытии биржи\n - TIME_IN_FORCE_ON_CLOSE: На закрытии биржи\n - TIME_IN_FORCE_IOC: Исполнить немедленно или отменить\n - TIME_IN_FORCE_FOK: Исполнить полностью или отменить",
                "title": "Срок действия заявки"
              },
              "limit_price": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "stop_price": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "stop_condition": {
                "type": "string",
                "enum": [
                  "STOP_CONDITION_UNSPECIFIED",
                  "STOP_CONDITION_LAST_UP",
                  "STOP_CONDITION_LAST_DOWN"
                ],
                "default": "STOP_CONDITION_UNSPECIFIED",
                "description": "- STOP_CONDITION_UNSPECIFIED: Значение не указано\n - STOP_CONDITION_LAST_UP: Цена срабатывания больше текущей цены\n - STOP_CONDITION_LAST_DOWN: Цена срабатывания меньше текущей цены",
                "title": "Условие срабатывания стоп заявки"
              },
              "legs": {
                "type": "array",
                "items": {
                  "type": "object",
                  "properties": {
                    "symbol": {
                      "type": "string",
                      "title": "Символ инструмента"
                    },
                    "quantity": {
                      "type": "object",
                      "properties": {
                        "value": {
                          "type": "string",
                          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                        }
                      },
                      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
                    },
                    "side": {
                      "type": "string",
                      "enum": [
                        "SIDE_UNSPECIFIED",
                        "SIDE_BUY",
                        "SIDE_SELL"
                      ],
                      "default": "SIDE_UNSPECIFIED",
                      "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
                      "title": "Сторона сделки"
                    }
                  },
                  "title": "Лег"
                },
                "title": "Необходимо для мульти лег заявки"
              },
              "client_order_id": {
                "type": "string",
                "title": "Уникальный идентификатор заявки. Автоматически генерируется, если не отправлен. (максимум 20 символов)"
              },
              "valid_before": {
                "type": "string",
                "enum": [
                  "VALID_BEFORE_UNSPECIFIED",
                  "VALID_BEFORE_END_OF_DAY",
                  "VALID_BEFORE_GOOD_TILL_CANCEL",
                  "VALID_BEFORE_GOOD_TILL_DATE"
                ],
                "default": "VALID_BEFORE_UNSPECIFIED",
                "description": "- VALID_BEFORE_UNSPECIFIED: Значение не указано\n - VALID_BEFORE_END_OF_DAY: До конца торгового дня\n - VALID_BEFORE_GOOD_TILL_CANCEL: До отмены\n - VALID_BEFORE_GOOD_TILL_DATE: До указанной даты-времени. Данный тип поддерживается только при выставлении SL/TP заявок",
                "title": "Срок действия условной заявки"
              },
              "comment": {
                "type": "string",
                "title": "Метка заявки. (максимум 128 символов)"
              }
            },
            "title": "Информация о заявке"
          },
          "transact_at": {
            "type": "string",
            "format": "date-time",
            "title": "Дата и время выставления заявки"
          },
          "accept_at": {
            "type": "string",
            "format": "date-time",
            "title": "Дата и время принятия заявки"
          },
          "withdraw_at": {
            "type": "string",
            "format": "date-time",
            "title": "Дата и время  отмены заявки"
          },
          "initial_quantity": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "executed_quantity": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "remaining_quantity": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "sltp_order": {
            "type": "object",
            "properties": {
              "account_id": {
                "type": "string",
                "title": "Идентификатор аккаунта"
              },
              "symbol": {
                "type": "string",
                "title": "Символ инструмента"
              },
              "side": {
                "type": "string",
                "enum": [
                  "SIDE_UNSPECIFIED",
                  "SIDE_BUY",
                  "SIDE_SELL"
                ],
                "default": "SIDE_UNSPECIFIED",
                "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
                "title": "Сторона сделки"
              },
              "quantity_sl": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "sl_price": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "limit_price": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "quantity_tp": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "tp_price": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "tp_guard_spread": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "tp_spread_measure": {
                "type": "string",
                "enum": [
                  "TP_SPREAD_MEASURE_UNDEFINED",
                  "TP_SPREAD_MEASURE_VALUE",
                  "TP_SPREAD_MEASURE_PERCENT"
                ],
                "default": "TP_SPREAD_MEASURE_UNDEFINED",
                "description": "- TP_SPREAD_MEASURE_UNDEFINED: Значение не указано\n - TP_SPREAD_MEASURE_VALUE: в единицах цены\n - TP_SPREAD_MEASURE_PERCENT: в процентах, с максимальной точностью до сотых процента",
                "title": "Единица измерения величины защитного спреда для цены исполнения TP"
              },
              "client_order_id": {
                "type": "string",
                "title": "Уникальный идентификатор заявки. Автоматически генерируется, если не отправлен. (максимум 20 символов)"
              },
              "valid_before": {
                "type": "string",
                "enum": [
                  "VALID_BEFORE_UNSPECIFIED",
                  "VALID_BEFORE_END_OF_DAY",
                  "VALID_BEFORE_GOOD_TILL_CANCEL",
                  "VALID_BEFORE_GOOD_TILL_DATE"
                ],
                "default": "VALID_BEFORE_UNSPECIFIED",
                "description": "- VALID_BEFORE_UNSPECIFIED: Значение не указано\n - VALID_BEFORE_END_OF_DAY: До конца торгового дня\n - VALID_BEFORE_GOOD_TILL_CANCEL: До отмены\n - VALID_BEFORE_GOOD_TILL_DATE: До указанной даты-времени. Данный тип поддерживается только при выставлении SL/TP заявок",
                "title": "Срок действия условной заявки"
              },
              "valid_expiry_time": {
                "type": "string",
                "format": "date-time",
                "title": "Временная метка прекращения действия SL/TP заявки"
              },
              "comment": {
                "type": "string",
                "title": "Метка заявки. (максимум 128 символов)"
              }
            },
            "title": "Информация о SL/TP заявке"
          }
        },
        "title": "Состояние заявки"
      },
      "title": "Заявки"
    }
  },
  "title": "Список торговых заявок"
}
```

---

### Выставление биржевой заявки

`POST /v1/accounts/{accountId}/orders`

Поле `accountId` берется из URL-пути, остальные поля передаются в теле запроса.

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| accountId | string | yes | Идентификатор аккаунта |

**Body** (required, application/json)

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| symbol | string | yes | Символ инструмента |
| quantity | object | yes | Количество (decimal value) |
| side | string (enum) | yes | Сторона сделки: `SIDE_UNSPECIFIED`, `SIDE_BUY` (покупка), `SIDE_SELL` (продажа) |
| type | string (enum) | yes | Тип заявки: `ORDER_TYPE_UNSPECIFIED`, `ORDER_TYPE_MARKET` (рыночная), `ORDER_TYPE_LIMIT` (лимитная), `ORDER_TYPE_STOP` (стоп рыночная), `ORDER_TYPE_STOP_LIMIT` (стоп лимитная), `ORDER_TYPE_MULTI_LEG` (мульти лег) |
| timeInForce | string (enum) | no | Срок действия: `TIME_IN_FORCE_UNSPECIFIED`, `TIME_IN_FORCE_DAY` (до конца дня), `TIME_IN_FORCE_GOOD_TILL_CANCEL` (до отмены), `TIME_IN_FORCE_GOOD_TILL_CROSSING` (до пересечения), `TIME_IN_FORCE_EXT` (внебиржевая), `TIME_IN_FORCE_ON_OPEN` (на открытии), `TIME_IN_FORCE_ON_CLOSE` (на закрытии), `TIME_IN_FORCE_IOC` (исполнить или отменить), `TIME_IN_FORCE_FOK` (исполнить полностью или отменить) |
| limitPrice | object | no | Лимитная цена (decimal value) |
| stopPrice | object | no | Стоп цена (decimal value) |
| stopCondition | string (enum) | no | Условие срабатывания стопа: `STOP_CONDITION_UNSPECIFIED`, `STOP_CONDITION_LAST_UP` (цена выше текущей), `STOP_CONDITION_LAST_DOWN` (цена ниже текущей) |
| legs | array | no | Ноги для мульти лег заявки |
| clientOrderId | string | no | Уникальный идентификатор заявки (максимум 20 символов). Автоматически генерируется, если не отправлен. |
| validBefore | string (enum) | no | Срок действия условной заявки: `VALID_BEFORE_UNSPECIFIED`, `VALID_BEFORE_END_OF_DAY` (до конца торгового дня), `VALID_BEFORE_GOOD_TILL_CANCEL` (до отмены), `VALID_BEFORE_GOOD_TILL_DATE` (до указанной даты-времени, только для SL/TP) |
| comment | string | no | Метка заявки (максимум 128 символов) |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 400 | Неверно переданы торговые параметры |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт или инструмент не были найдены |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/accounts/{accountId}/orders' \
  --request POST \
  --header 'Content-Type: application/json' \
  --header 'Authorization: YOUR_SECRET_TOKEN' \
  --data '{
  "symbol": "",
  "quantity": {
    "value": ""
  },
  "side": "SIDE_UNSPECIFIED",
  "type": "ORDER_TYPE_UNSPECIFIED",
  "timeInForce": "TIME_IN_FORCE_UNSPECIFIED",
  "limitPrice": {
    "value": ""
  },
  "stopPrice": {
    "value": ""
  },
  "stopCondition": "STOP_CONDITION_UNSPECIFIED",
  "legs": [
    {
      "symbol": "",
      "quantity": {
        "value": ""
      },
      "side": "SIDE_UNSPECIFIED"
    }
  ],
  "clientOrderId": "",
  "validBefore": "VALID_BEFORE_UNSPECIFIED",
  "comment": ""
}'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "order_id": {
      "type": "string",
      "title": "Идентификатор заявки"
    },
    "exec_id": {
      "type": "string",
      "title": "Идентификатор исполнения"
    },
    "status": {
      "type": "string",
      "enum": [
        "ORDER_STATUS_UNSPECIFIED",
        "ORDER_STATUS_NEW",
        "ORDER_STATUS_PARTIALLY_FILLED",
        "ORDER_STATUS_FILLED",
        "ORDER_STATUS_DONE_FOR_DAY",
        "ORDER_STATUS_CANCELED",
        "ORDER_STATUS_REPLACED",
        "ORDER_STATUS_PENDING_CANCEL",
        "ORDER_STATUS_REJECTED",
        "ORDER_STATUS_SUSPENDED",
        "ORDER_STATUS_PENDING_NEW",
        "ORDER_STATUS_EXPIRED",
        "ORDER_STATUS_FAILED",
        "ORDER_STATUS_FORWARDING",
        "ORDER_STATUS_WAIT",
        "ORDER_STATUS_DENIED_BY_BROKER",
        "ORDER_STATUS_REJECTED_BY_EXCHANGE",
        "ORDER_STATUS_WATCHING",
        "ORDER_STATUS_EXECUTED",
        "ORDER_STATUS_DISABLED",
        "ORDER_STATUS_LINK_WAIT",
        "ORDER_STATUS_SL_GUARD_TIME",
        "ORDER_STATUS_SL_EXECUTED",
        "ORDER_STATUS_SL_FORWARDING",
        "ORDER_STATUS_TP_GUARD_TIME",
        "ORDER_STATUS_TP_EXECUTED",
        "ORDER_STATUS_TP_CORRECTION",
        "ORDER_STATUS_TP_FORWARDING",
        "ORDER_STATUS_TP_CORR_GUARD_TIME"
      ],
      "default": "ORDER_STATUS_UNSPECIFIED",
      "description": "- ORDER_STATUS_UNSPECIFIED: Неопределенное значение\n - ORDER_STATUS_NEW: Новая заявка\n - ORDER_STATUS_PARTIALLY_FILLED: Частично исполненная\n - ORDER_STATUS_FILLED: Исполненная\n - ORDER_STATUS_DONE_FOR_DAY: Действует в течение дня\n - ORDER_STATUS_CANCELED: Отменена\n - ORDER_STATUS_REPLACED: Заменена на другую\n - ORDER_STATUS_PENDING_CANCEL: Ожидает отмены\n - ORDER_STATUS_REJECTED: Отклонена\n - ORDER_STATUS_SUSPENDED: Приостановлена\n - ORDER_STATUS_PENDING_NEW: В ожидании новой\n - ORDER_STATUS_EXPIRED: Истекла\n - ORDER_STATUS_FAILED: Ошибка\n - ORDER_STATUS_FORWARDING: Пересылка\n - ORDER_STATUS_WAIT: Ожидает\n - ORDER_STATUS_DENIED_BY_BROKER: Отклонено брокером\n - ORDER_STATUS_REJECTED_BY_EXCHANGE: Отклонено биржей\n - ORDER_STATUS_WATCHING: Наблюдение\n - ORDER_STATUS_EXECUTED: Исполнена\n - ORDER_STATUS_DISABLED: Отключена\n - ORDER_STATUS_LINK_WAIT: Ожидание ссылки\n - ORDER_STATUS_SL_GUARD_TIME: Защитное время SL\n - ORDER_STATUS_SL_EXECUTED: Исполнена по SL\n - ORDER_STATUS_SL_FORWARDING: Пересылка SL\n - ORDER_STATUS_TP_GUARD_TIME: Защитное время TP\n - ORDER_STATUS_TP_EXECUTED: Исполнена по TP\n - ORDER_STATUS_TP_CORRECTION: Коррекция TP\n - ORDER_STATUS_TP_FORWARDING: Пересылка TP\n - ORDER_STATUS_TP_CORR_GUARD_TIME: Коррекция TP в защитное время",
      "title": "Статус заявки"
    },
    "order": {
      "type": "object",
      "properties": {
        "account_id": {
          "type": "string",
          "title": "Идентификатор аккаунта"
        },
        "symbol": {
          "type": "string",
          "title": "Символ инструмента"
        },
        "quantity": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "side": {
          "type": "string",
          "enum": [
            "SIDE_UNSPECIFIED",
            "SIDE_BUY",
            "SIDE_SELL"
          ],
          "default": "SIDE_UNSPECIFIED",
          "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
          "title": "Сторона сделки"
        },
        "type": {
          "type": "string",
          "enum": [
            "ORDER_TYPE_UNSPECIFIED",
            "ORDER_TYPE_MARKET",
            "ORDER_TYPE_LIMIT",
            "ORDER_TYPE_STOP",
            "ORDER_TYPE_STOP_LIMIT",
            "ORDER_TYPE_MULTI_LEG"
          ],
          "default": "ORDER_TYPE_UNSPECIFIED",
          "description": "- ORDER_TYPE_UNSPECIFIED: Значение не указано\n - ORDER_TYPE_MARKET: Рыночная\n - ORDER_TYPE_LIMIT: Лимитная\n - ORDER_TYPE_STOP: Стоп заявка рыночная\n - ORDER_TYPE_STOP_LIMIT: Стоп заявка лимитная\n - ORDER_TYPE_MULTI_LEG: Мульти лег заявка",
          "title": "Тип заявки"
        },
        "time_in_force": {
          "type": "string",
          "enum": [
            "TIME_IN_FORCE_UNSPECIFIED",
            "TIME_IN_FORCE_DAY",
            "TIME_IN_FORCE_GOOD_TILL_CANCEL",
            "TIME_IN_FORCE_GOOD_TILL_CROSSING",
            "TIME_IN_FORCE_EXT",
            "TIME_IN_FORCE_ON_OPEN",
            "TIME_IN_FORCE_ON_CLOSE",
            "TIME_IN_FORCE_IOC",
            "TIME_IN_FORCE_FOK"
          ],
          "default": "TIME_IN_FORCE_UNSPECIFIED",
          "description": "- TIME_IN_FORCE_UNSPECIFIED: Значение не указано\n - TIME_IN_FORCE_DAY: До конца дня\n - TIME_IN_FORCE_GOOD_TILL_CANCEL: Действителен до отмены\n - TIME_IN_FORCE_GOOD_TILL_CROSSING: Действителен до пересечения\n - TIME_IN_FORCE_EXT: Внебиржевая торговля\n - TIME_IN_FORCE_ON_OPEN: На открытии биржи\n - TIME_IN_FORCE_ON_CLOSE: На закрытии биржи\n - TIME_IN_FORCE_IOC: Исполнить немедленно или отменить\n - TIME_IN_FORCE_FOK: Исполнить полностью или отменить",
          "title": "Срок действия заявки"
        },
        "limit_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "stop_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "stop_condition": {
          "type": "string",
          "enum": [
            "STOP_CONDITION_UNSPECIFIED",
            "STOP_CONDITION_LAST_UP",
            "STOP_CONDITION_LAST_DOWN"
          ],
          "default": "STOP_CONDITION_UNSPECIFIED",
          "description": "- STOP_CONDITION_UNSPECIFIED: Значение не указано\n - STOP_CONDITION_LAST_UP: Цена срабатывания больше текущей цены\n - STOP_CONDITION_LAST_DOWN: Цена срабатывания меньше текущей цены",
          "title": "Условие срабатывания стоп заявки"
        },
        "legs": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "symbol": {
                "type": "string",
                "title": "Символ инструмента"
              },
              "quantity": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "side": {
                "type": "string",
                "enum": [
                  "SIDE_UNSPECIFIED",
                  "SIDE_BUY",
                  "SIDE_SELL"
                ],
                "default": "SIDE_UNSPECIFIED",
                "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
                "title": "Сторона сделки"
              }
            },
            "title": "Лег"
          },
          "title": "Необходимо для мульти лег заявки"
        },
        "client_order_id": {
          "type": "string",
          "title": "Уникальный идентификатор заявки. Автоматически генерируется, если не отправлен. (максимум 20 символов)"
        },
        "valid_before": {
          "type": "string",
          "enum": [
            "VALID_BEFORE_UNSPECIFIED",
            "VALID_BEFORE_END_OF_DAY",
            "VALID_BEFORE_GOOD_TILL_CANCEL",
            "VALID_BEFORE_GOOD_TILL_DATE"
          ],
          "default": "VALID_BEFORE_UNSPECIFIED",
          "description": "- VALID_BEFORE_UNSPECIFIED: Значение не указано\n - VALID_BEFORE_END_OF_DAY: До конца торгового дня\n - VALID_BEFORE_GOOD_TILL_CANCEL: До отмены\n - VALID_BEFORE_GOOD_TILL_DATE: До указанной даты-времени. Данный тип поддерживается только при выставлении SL/TP заявок",
          "title": "Срок действия условной заявки"
        },
        "comment": {
          "type": "string",
          "title": "Метка заявки. (максимум 128 символов)"
        }
      },
      "title": "Информация о заявке"
    },
    "transact_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время выставления заявки"
    },
    "accept_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время принятия заявки"
    },
    "withdraw_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время  отмены заявки"
    },
    "initial_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "executed_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "remaining_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "sltp_order": {
      "type": "object",
      "properties": {
        "account_id": {
          "type": "string",
          "title": "Идентификатор аккаунта"
        },
        "symbol": {
          "type": "string",
          "title": "Символ инструмента"
        },
        "side": {
          "type": "string",
          "enum": [
            "SIDE_UNSPECIFIED",
            "SIDE_BUY",
            "SIDE_SELL"
          ],
          "default": "SIDE_UNSPECIFIED",
          "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
          "title": "Сторона сделки"
        },
        "quantity_sl": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "sl_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "limit_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "quantity_tp": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_guard_spread": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_spread_measure": {
          "type": "string",
          "enum": [
            "TP_SPREAD_MEASURE_UNDEFINED",
            "TP_SPREAD_MEASURE_VALUE",
            "TP_SPREAD_MEASURE_PERCENT"
          ],
          "default": "TP_SPREAD_MEASURE_UNDEFINED",
          "description": "- TP_SPREAD_MEASURE_UNDEFINED: Значение не указано\n - TP_SPREAD_MEASURE_VALUE: в единицах цены\n - TP_SPREAD_MEASURE_PERCENT: в процентах, с максимальной точностью до сотых процента",
          "title": "Единица измерения величины защитного спреда для цены исполнения TP"
        },
        "client_order_id": {
          "type": "string",
          "title": "Уникальный идентификатор заявки. Автоматически генерируется, если не отправлен. (максимум 20 символов)"
        },
        "valid_before": {
          "type": "string",
          "enum": [
            "VALID_BEFORE_UNSPECIFIED",
            "VALID_BEFORE_END_OF_DAY",
            "VALID_BEFORE_GOOD_TILL_CANCEL",
            "VALID_BEFORE_GOOD_TILL_DATE"
          ],
          "default": "VALID_BEFORE_UNSPECIFIED",
          "description": "- VALID_BEFORE_UNSPECIFIED: Значение не указано\n - VALID_BEFORE_END_OF_DAY: До конца торгового дня\n - VALID_BEFORE_GOOD_TILL_CANCEL: До отмены\n - VALID_BEFORE_GOOD_TILL_DATE: До указанной даты-времени. Данный тип поддерживается только при выставлении SL/TP заявок",
          "title": "Срок действия условной заявки"
        },
        "valid_expiry_time": {
          "type": "string",
          "format": "date-time",
          "title": "Временная метка прекращения действия SL/TP заявки"
        },
        "comment": {
          "type": "string",
          "title": "Метка заявки. (максимум 128 символов)"
        }
      },
      "title": "Информация о SL/TP заявке"
    }
  },
  "title": "Состояние заявки"
}
```

---

### Получение информации о конкретном ордере

`GET /v1/accounts/{accountId}/orders/{orderId}`

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| accountId | string | yes | Идентификатор аккаунта |
| orderId | string | yes | Идентификатор заявки |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт или заявка не были найдены |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/accounts/{accountId}/orders/{orderId}' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "order_id": {
      "type": "string",
      "title": "Идентификатор заявки"
    },
    "exec_id": {
      "type": "string",
      "title": "Идентификатор исполнения"
    },
    "status": {
      "type": "string",
      "enum": [
        "ORDER_STATUS_UNSPECIFIED",
        "ORDER_STATUS_NEW",
        "ORDER_STATUS_PARTIALLY_FILLED",
        "ORDER_STATUS_FILLED",
        "ORDER_STATUS_DONE_FOR_DAY",
        "ORDER_STATUS_CANCELED",
        "ORDER_STATUS_REPLACED",
        "ORDER_STATUS_PENDING_CANCEL",
        "ORDER_STATUS_REJECTED",
        "ORDER_STATUS_SUSPENDED",
        "ORDER_STATUS_PENDING_NEW",
        "ORDER_STATUS_EXPIRED",
        "ORDER_STATUS_FAILED",
        "ORDER_STATUS_FORWARDING",
        "ORDER_STATUS_WAIT",
        "ORDER_STATUS_DENIED_BY_BROKER",
        "ORDER_STATUS_REJECTED_BY_EXCHANGE",
        "ORDER_STATUS_WATCHING",
        "ORDER_STATUS_EXECUTED",
        "ORDER_STATUS_DISABLED",
        "ORDER_STATUS_LINK_WAIT",
        "ORDER_STATUS_SL_GUARD_TIME",
        "ORDER_STATUS_SL_EXECUTED",
        "ORDER_STATUS_SL_FORWARDING",
        "ORDER_STATUS_TP_GUARD_TIME",
        "ORDER_STATUS_TP_EXECUTED",
        "ORDER_STATUS_TP_CORRECTION",
        "ORDER_STATUS_TP_FORWARDING",
        "ORDER_STATUS_TP_CORR_GUARD_TIME"
      ],
      "default": "ORDER_STATUS_UNSPECIFIED",
      "description": "- ORDER_STATUS_UNSPECIFIED: Неопределенное значение\n - ORDER_STATUS_NEW: Новая заявка\n - ORDER_STATUS_PARTIALLY_FILLED: Частично исполненная\n - ORDER_STATUS_FILLED: Исполненная\n - ORDER_STATUS_DONE_FOR_DAY: Действует в течение дня\n - ORDER_STATUS_CANCELED: Отменена\n - ORDER_STATUS_REPLACED: Заменена на другую\n - ORDER_STATUS_PENDING_CANCEL: Ожидает отмены\n - ORDER_STATUS_REJECTED: Отклонена\n - ORDER_STATUS_SUSPENDED: Приостановлена\n - ORDER_STATUS_PENDING_NEW: В ожидании новой\n - ORDER_STATUS_EXPIRED: Истекла\n - ORDER_STATUS_FAILED: Ошибка\n - ORDER_STATUS_FORWARDING: Пересылка\n - ORDER_STATUS_WAIT: Ожидает\n - ORDER_STATUS_DENIED_BY_BROKER: Отклонено брокером\n - ORDER_STATUS_REJECTED_BY_EXCHANGE: Отклонено биржей\n - ORDER_STATUS_WATCHING: Наблюдение\n - ORDER_STATUS_EXECUTED: Исполнена\n - ORDER_STATUS_DISABLED: Отключена\n - ORDER_STATUS_LINK_WAIT: Ожидание ссылки\n - ORDER_STATUS_SL_GUARD_TIME: Защитное время SL\n - ORDER_STATUS_SL_EXECUTED: Исполнена по SL\n - ORDER_STATUS_SL_FORWARDING: Пересылка SL\n - ORDER_STATUS_TP_GUARD_TIME: Защитное время TP\n - ORDER_STATUS_TP_EXECUTED: Исполнена по TP\n - ORDER_STATUS_TP_CORRECTION: Коррекция TP\n - ORDER_STATUS_TP_FORWARDING: Пересылка TP\n - ORDER_STATUS_TP_CORR_GUARD_TIME: Коррекция TP в защитное время",
      "title": "Статус заявки"
    },
    "order": {
      "type": "object",
      "properties": {
        "account_id": {
          "type": "string",
          "title": "Идентификатор аккаунта"
        },
        "symbol": {
          "type": "string",
          "title": "Символ инструмента"
        },
        "quantity": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "side": {
          "type": "string",
          "enum": [
            "SIDE_UNSPECIFIED",
            "SIDE_BUY",
            "SIDE_SELL"
          ],
          "default": "SIDE_UNSPECIFIED",
          "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
          "title": "Сторона сделки"
        },
        "type": {
          "type": "string",
          "enum": [
            "ORDER_TYPE_UNSPECIFIED",
            "ORDER_TYPE_MARKET",
            "ORDER_TYPE_LIMIT",
            "ORDER_TYPE_STOP",
            "ORDER_TYPE_STOP_LIMIT",
            "ORDER_TYPE_MULTI_LEG"
          ],
          "default": "ORDER_TYPE_UNSPECIFIED",
          "description": "- ORDER_TYPE_UNSPECIFIED: Значение не указано\n - ORDER_TYPE_MARKET: Рыночная\n - ORDER_TYPE_LIMIT: Лимитная\n - ORDER_TYPE_STOP: Стоп заявка рыночная\n - ORDER_TYPE_STOP_LIMIT: Стоп заявка лимитная\n - ORDER_TYPE_MULTI_LEG: Мульти лег заявка",
          "title": "Тип заявки"
        },
        "time_in_force": {
          "type": "string",
          "enum": [
            "TIME_IN_FORCE_UNSPECIFIED",
            "TIME_IN_FORCE_DAY",
            "TIME_IN_FORCE_GOOD_TILL_CANCEL",
            "TIME_IN_FORCE_GOOD_TILL_CROSSING",
            "TIME_IN_FORCE_EXT",
            "TIME_IN_FORCE_ON_OPEN",
            "TIME_IN_FORCE_ON_CLOSE",
            "TIME_IN_FORCE_IOC",
            "TIME_IN_FORCE_FOK"
          ],
          "default": "TIME_IN_FORCE_UNSPECIFIED",
          "description": "- TIME_IN_FORCE_UNSPECIFIED: Значение не указано\n - TIME_IN_FORCE_DAY: До конца дня\n - TIME_IN_FORCE_GOOD_TILL_CANCEL: Действителен до отмены\n - TIME_IN_FORCE_GOOD_TILL_CROSSING: Действителен до пересечения\n - TIME_IN_FORCE_EXT: Внебиржевая торговля\n - TIME_IN_FORCE_ON_OPEN: На открытии биржи\n - TIME_IN_FORCE_ON_CLOSE: На закрытии биржи\n - TIME_IN_FORCE_IOC: Исполнить немедленно или отменить\n - TIME_IN_FORCE_FOK: Исполнить полностью или отменить",
          "title": "Срок действия заявки"
        },
        "limit_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "stop_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "stop_condition": {
          "type": "string",
          "enum": [
            "STOP_CONDITION_UNSPECIFIED",
            "STOP_CONDITION_LAST_UP",
            "STOP_CONDITION_LAST_DOWN"
          ],
          "default": "STOP_CONDITION_UNSPECIFIED",
          "description": "- STOP_CONDITION_UNSPECIFIED: Значение не указано\n - STOP_CONDITION_LAST_UP: Цена срабатывания больше текущей цены\n - STOP_CONDITION_LAST_DOWN: Цена срабатывания меньше текущей цены",
          "title": "Условие срабатывания стоп заявки"
        },
        "legs": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "symbol": {
                "type": "string",
                "title": "Символ инструмента"
              },
              "quantity": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "side": {
                "type": "string",
                "enum": [
                  "SIDE_UNSPECIFIED",
                  "SIDE_BUY",
                  "SIDE_SELL"
                ],
                "default": "SIDE_UNSPECIFIED",
                "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
                "title": "Сторона сделки"
              }
            },
            "title": "Лег"
          },
          "title": "Необходимо для мульти лег заявки"
        },
        "client_order_id": {
          "type": "string",
          "title": "Уникальный идентификатор заявки. Автоматически генерируется, если не отправлен. (максимум 20 символов)"
        },
        "valid_before": {
          "type": "string",
          "enum": [
            "VALID_BEFORE_UNSPECIFIED",
            "VALID_BEFORE_END_OF_DAY",
            "VALID_BEFORE_GOOD_TILL_CANCEL",
            "VALID_BEFORE_GOOD_TILL_DATE"
          ],
          "default": "VALID_BEFORE_UNSPECIFIED",
          "description": "- VALID_BEFORE_UNSPECIFIED: Значение не указано\n - VALID_BEFORE_END_OF_DAY: До конца торгового дня\n - VALID_BEFORE_GOOD_TILL_CANCEL: До отмены\n - VALID_BEFORE_GOOD_TILL_DATE: До указанной даты-времени. Данный тип поддерживается только при выставлении SL/TP заявок",
          "title": "Срок действия условной заявки"
        },
        "comment": {
          "type": "string",
          "title": "Метка заявки. (максимум 128 символов)"
        }
      },
      "title": "Информация о заявке"
    },
    "transact_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время выставления заявки"
    },
    "accept_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время принятия заявки"
    },
    "withdraw_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время  отмены заявки"
    },
    "initial_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "executed_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "remaining_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "sltp_order": {
      "type": "object",
      "properties": {
        "account_id": {
          "type": "string",
          "title": "Идентификатор аккаунта"
        },
        "symbol": {
          "type": "string",
          "title": "Символ инструмента"
        },
        "side": {
          "type": "string",
          "enum": [
            "SIDE_UNSPECIFIED",
            "SIDE_BUY",
            "SIDE_SELL"
          ],
          "default": "SIDE_UNSPECIFIED",
          "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
          "title": "Сторона сделки"
        },
        "quantity_sl": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "sl_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "limit_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "quantity_tp": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_guard_spread": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_spread_measure": {
          "type": "string",
          "enum": [
            "TP_SPREAD_MEASURE_UNDEFINED",
            "TP_SPREAD_MEASURE_VALUE",
            "TP_SPREAD_MEASURE_PERCENT"
          ],
          "default": "TP_SPREAD_MEASURE_UNDEFINED",
          "description": "- TP_SPREAD_MEASURE_UNDEFINED: Значение не указано\n - TP_SPREAD_MEASURE_VALUE: в единицах цены\n - TP_SPREAD_MEASURE_PERCENT: в процентах, с максимальной точностью до сотых процента",
          "title": "Единица измерения величины защитного спреда для цены исполнения TP"
        },
        "client_order_id": {
          "type": "string",
          "title": "Уникальный идентификатор заявки. Автоматически генерируется, если не отправлен. (максимум 20 символов)"
        },
        "valid_before": {
          "type": "string",
          "enum": [
            "VALID_BEFORE_UNSPECIFIED",
            "VALID_BEFORE_END_OF_DAY",
            "VALID_BEFORE_GOOD_TILL_CANCEL",
            "VALID_BEFORE_GOOD_TILL_DATE"
          ],
          "default": "VALID_BEFORE_UNSPECIFIED",
          "description": "- VALID_BEFORE_UNSPECIFIED: Значение не указано\n - VALID_BEFORE_END_OF_DAY: До конца торгового дня\n - VALID_BEFORE_GOOD_TILL_CANCEL: До отмены\n - VALID_BEFORE_GOOD_TILL_DATE: До указанной даты-времени. Данный тип поддерживается только при выставлении SL/TP заявок",
          "title": "Срок действия условной заявки"
        },
        "valid_expiry_time": {
          "type": "string",
          "format": "date-time",
          "title": "Временная метка прекращения действия SL/TP заявки"
        },
        "comment": {
          "type": "string",
          "title": "Метка заявки. (максимум 128 символов)"
        }
      },
      "title": "Информация о SL/TP заявке"
    }
  },
  "title": "Состояние заявки"
}
```

---

### Отмена биржевой заявки

`DELETE /v1/accounts/{accountId}/orders/{orderId}`

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| accountId | string | yes | Идентификатор аккаунта |
| orderId | string | yes | Идентификатор заявки |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 400 | Заявка не может быть отменена так как она уже исполнена |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт или заявка не были найдены |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/accounts/{accountId}/orders/{orderId}' \
  --request DELETE \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "order_id": {
      "type": "string",
      "title": "Идентификатор заявки"
    },
    "exec_id": {
      "type": "string",
      "title": "Идентификатор исполнения"
    },
    "status": {
      "type": "string",
      "enum": [
        "ORDER_STATUS_UNSPECIFIED",
        "ORDER_STATUS_NEW",
        "ORDER_STATUS_PARTIALLY_FILLED",
        "ORDER_STATUS_FILLED",
        "ORDER_STATUS_DONE_FOR_DAY",
        "ORDER_STATUS_CANCELED",
        "ORDER_STATUS_REPLACED",
        "ORDER_STATUS_PENDING_CANCEL",
        "ORDER_STATUS_REJECTED",
        "ORDER_STATUS_SUSPENDED",
        "ORDER_STATUS_PENDING_NEW",
        "ORDER_STATUS_EXPIRED",
        "ORDER_STATUS_FAILED",
        "ORDER_STATUS_FORWARDING",
        "ORDER_STATUS_WAIT",
        "ORDER_STATUS_DENIED_BY_BROKER",
        "ORDER_STATUS_REJECTED_BY_EXCHANGE",
        "ORDER_STATUS_WATCHING",
        "ORDER_STATUS_EXECUTED",
        "ORDER_STATUS_DISABLED",
        "ORDER_STATUS_LINK_WAIT",
        "ORDER_STATUS_SL_GUARD_TIME",
        "ORDER_STATUS_SL_EXECUTED",
        "ORDER_STATUS_SL_FORWARDING",
        "ORDER_STATUS_TP_GUARD_TIME",
        "ORDER_STATUS_TP_EXECUTED",
        "ORDER_STATUS_TP_CORRECTION",
        "ORDER_STATUS_TP_FORWARDING",
        "ORDER_STATUS_TP_CORR_GUARD_TIME"
      ],
      "default": "ORDER_STATUS_UNSPECIFIED",
      "description": "- ORDER_STATUS_UNSPECIFIED: Неопределенное значение\n - ORDER_STATUS_NEW: Новая заявка\n - ORDER_STATUS_PARTIALLY_FILLED: Частично исполненная\n - ORDER_STATUS_FILLED: Исполненная\n - ORDER_STATUS_DONE_FOR_DAY: Действует в течение дня\n - ORDER_STATUS_CANCELED: Отменена\n - ORDER_STATUS_REPLACED: Заменена на другую\n - ORDER_STATUS_PENDING_CANCEL: Ожидает отмены\n - ORDER_STATUS_REJECTED: Отклонена\n - ORDER_STATUS_SUSPENDED: Приостановлена\n - ORDER_STATUS_PENDING_NEW: В ожидании новой\n - ORDER_STATUS_EXPIRED: Истекла\n - ORDER_STATUS_FAILED: Ошибка\n - ORDER_STATUS_FORWARDING: Пересылка\n - ORDER_STATUS_WAIT: Ожидает\n - ORDER_STATUS_DENIED_BY_BROKER: Отклонено брокером\n - ORDER_STATUS_REJECTED_BY_EXCHANGE: Отклонено биржей\n - ORDER_STATUS_WATCHING: Наблюдение\n - ORDER_STATUS_EXECUTED: Исполнена\n - ORDER_STATUS_DISABLED: Отключена\n - ORDER_STATUS_LINK_WAIT: Ожидание ссылки\n - ORDER_STATUS_SL_GUARD_TIME: Защитное время SL\n - ORDER_STATUS_SL_EXECUTED: Исполнена по SL\n - ORDER_STATUS_SL_FORWARDING: Пересылка SL\n - ORDER_STATUS_TP_GUARD_TIME: Защитное время TP\n - ORDER_STATUS_TP_EXECUTED: Исполнена по TP\n - ORDER_STATUS_TP_CORRECTION: Коррекция TP\n - ORDER_STATUS_TP_FORWARDING: Пересылка TP\n - ORDER_STATUS_TP_CORR_GUARD_TIME: Коррекция TP в защитное время",
      "title": "Статус заявки"
    },
    "order": {
      "type": "object",
      "properties": {
        "account_id": {
          "type": "string",
          "title": "Идентификатор аккаунта"
        },
        "symbol": {
          "type": "string",
          "title": "Символ инструмента"
        },
        "quantity": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "side": {
          "type": "string",
          "enum": [
            "SIDE_UNSPECIFIED",
            "SIDE_BUY",
            "SIDE_SELL"
          ],
          "default": "SIDE_UNSPECIFIED",
          "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
          "title": "Сторона сделки"
        },
        "type": {
          "type": "string",
          "enum": [
            "ORDER_TYPE_UNSPECIFIED",
            "ORDER_TYPE_MARKET",
            "ORDER_TYPE_LIMIT",
            "ORDER_TYPE_STOP",
            "ORDER_TYPE_STOP_LIMIT",
            "ORDER_TYPE_MULTI_LEG"
          ],
          "default": "ORDER_TYPE_UNSPECIFIED",
          "description": "- ORDER_TYPE_UNSPECIFIED: Значение не указано\n - ORDER_TYPE_MARKET: Рыночная\n - ORDER_TYPE_LIMIT: Лимитная\n - ORDER_TYPE_STOP: Стоп заявка рыночная\n - ORDER_TYPE_STOP_LIMIT: Стоп заявка лимитная\n - ORDER_TYPE_MULTI_LEG: Мульти лег заявка",
          "title": "Тип заявки"
        },
        "time_in_force": {
          "type": "string",
          "enum": [
            "TIME_IN_FORCE_UNSPECIFIED",
            "TIME_IN_FORCE_DAY",
            "TIME_IN_FORCE_GOOD_TILL_CANCEL",
            "TIME_IN_FORCE_GOOD_TILL_CROSSING",
            "TIME_IN_FORCE_EXT",
            "TIME_IN_FORCE_ON_OPEN",
            "TIME_IN_FORCE_ON_CLOSE",
            "TIME_IN_FORCE_IOC",
            "TIME_IN_FORCE_FOK"
          ],
          "default": "TIME_IN_FORCE_UNSPECIFIED",
          "description": "- TIME_IN_FORCE_UNSPECIFIED: Значение не указано\n - TIME_IN_FORCE_DAY: До конца дня\n - TIME_IN_FORCE_GOOD_TILL_CANCEL: Действителен до отмены\n - TIME_IN_FORCE_GOOD_TILL_CROSSING: Действителен до пересечения\n - TIME_IN_FORCE_EXT: Внебиржевая торговля\n - TIME_IN_FORCE_ON_OPEN: На открытии биржи\n - TIME_IN_FORCE_ON_CLOSE: На закрытии биржи\n - TIME_IN_FORCE_IOC: Исполнить немедленно или отменить\n - TIME_IN_FORCE_FOK: Исполнить полностью или отменить",
          "title": "Срок действия заявки"
        },
        "limit_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "stop_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "stop_condition": {
          "type": "string",
          "enum": [
            "STOP_CONDITION_UNSPECIFIED",
            "STOP_CONDITION_LAST_UP",
            "STOP_CONDITION_LAST_DOWN"
          ],
          "default": "STOP_CONDITION_UNSPECIFIED",
          "description": "- STOP_CONDITION_UNSPECIFIED: Значение не указано\n - STOP_CONDITION_LAST_UP: Цена срабатывания больше текущей цены\n - STOP_CONDITION_LAST_DOWN: Цена срабатывания меньше текущей цены",
          "title": "Условие срабатывания стоп заявки"
        },
        "legs": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "symbol": {
                "type": "string",
                "title": "Символ инструмента"
              },
              "quantity": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "side": {
                "type": "string",
                "enum": [
                  "SIDE_UNSPECIFIED",
                  "SIDE_BUY",
                  "SIDE_SELL"
                ],
                "default": "SIDE_UNSPECIFIED",
                "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
                "title": "Сторона сделки"
              }
            },
            "title": "Лег"
          },
          "title": "Необходимо для мульти лег заявки"
        },
        "client_order_id": {
          "type": "string",
          "title": "Уникальный идентификатор заявки. Автоматически генерируется, если не отправлен. (максимум 20 символов)"
        },
        "valid_before": {
          "type": "string",
          "enum": [
            "VALID_BEFORE_UNSPECIFIED",
            "VALID_BEFORE_END_OF_DAY",
            "VALID_BEFORE_GOOD_TILL_CANCEL",
            "VALID_BEFORE_GOOD_TILL_DATE"
          ],
          "default": "VALID_BEFORE_UNSPECIFIED",
          "description": "- VALID_BEFORE_UNSPECIFIED: Значение не указано\n - VALID_BEFORE_END_OF_DAY: До конца торгового дня\n - VALID_BEFORE_GOOD_TILL_CANCEL: До отмены\n - VALID_BEFORE_GOOD_TILL_DATE: До указанной даты-времени. Данный тип поддерживается только при выставлении SL/TP заявок",
          "title": "Срок действия условной заявки"
        },
        "comment": {
          "type": "string",
          "title": "Метка заявки. (максимум 128 символов)"
        }
      },
      "title": "Информация о заявке"
    },
    "transact_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время выставления заявки"
    },
    "accept_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время принятия заявки"
    },
    "withdraw_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время  отмены заявки"
    },
    "initial_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "executed_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "remaining_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "sltp_order": {
      "type": "object",
      "properties": {
        "account_id": {
          "type": "string",
          "title": "Идентификатор аккаунта"
        },
        "symbol": {
          "type": "string",
          "title": "Символ инструмента"
        },
        "side": {
          "type": "string",
          "enum": [
            "SIDE_UNSPECIFIED",
            "SIDE_BUY",
            "SIDE_SELL"
          ],
          "default": "SIDE_UNSPECIFIED",
          "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
          "title": "Сторона сделки"
        },
        "quantity_sl": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "sl_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "limit_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "quantity_tp": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_guard_spread": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_spread_measure": {
          "type": "string",
          "enum": [
            "TP_SPREAD_MEASURE_UNDEFINED",
            "TP_SPREAD_MEASURE_VALUE",
            "TP_SPREAD_MEASURE_PERCENT"
          ],
          "default": "TP_SPREAD_MEASURE_UNDEFINED",
          "description": "- TP_SPREAD_MEASURE_UNDEFINED: Значение не указано\n - TP_SPREAD_MEASURE_VALUE: в единицах цены\n - TP_SPREAD_MEASURE_PERCENT: в процентах, с максимальной точностью до сотых процента",
          "title": "Единица измерения величины защитного спреда для цены исполнения TP"
        },
        "client_order_id": {
          "type": "string",
          "title": "Уникальный идентификатор заявки. Автоматически генерируется, если не отправлен. (максимум 20 символов)"
        },
        "valid_before": {
          "type": "string",
          "enum": [
            "VALID_BEFORE_UNSPECIFIED",
            "VALID_BEFORE_END_OF_DAY",
            "VALID_BEFORE_GOOD_TILL_CANCEL",
            "VALID_BEFORE_GOOD_TILL_DATE"
          ],
          "default": "VALID_BEFORE_UNSPECIFIED",
          "description": "- VALID_BEFORE_UNSPECIFIED: Значение не указано\n - VALID_BEFORE_END_OF_DAY: До конца торгового дня\n - VALID_BEFORE_GOOD_TILL_CANCEL: До отмены\n - VALID_BEFORE_GOOD_TILL_DATE: До указанной даты-времени. Данный тип поддерживается только при выставлении SL/TP заявок",
          "title": "Срок действия условной заявки"
        },
        "valid_expiry_time": {
          "type": "string",
          "format": "date-time",
          "title": "Временная метка прекращения действия SL/TP заявки"
        },
        "comment": {
          "type": "string",
          "title": "Метка заявки. (максимум 128 символов)"
        }
      },
      "title": "Информация о SL/TP заявке"
    }
  },
  "title": "Состояние заявки"
}
```

---

### Выставление SL/TP заявки

`POST /v1/accounts/{accountId}/sltp-orders`

Поле `accountId` берется из URL-пути, остальные поля передаются в теле запроса.

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| accountId | string | yes | Идентификатор аккаунта |

**Body** (required, application/json)

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| symbol | string | yes | Символ инструмента |
| side | string (enum) | yes | Сторона сделки: `SIDE_UNSPECIFIED`, `SIDE_BUY` (покупка), `SIDE_SELL` (продажа) |
| quantitySl | object | no | Количество для стоп-лосс (decimal value) |
| slPrice | object | no | Цена стоп-лосс (decimal value) |
| limitPrice | object | no | Лимитная цена (decimal value) |
| quantityTp | object | no | Количество для тейк-профит (decimal value) |
| tpPrice | object | no | Цена тейк-профит (decimal value) |
| tpGuardSpread | object | no | Величина защитного спреда для цены исполнения TP (decimal value) |
| tpSpreadMeasure | string (enum) | no | Единица измерения защитного спреда TP: `TP_SPREAD_MEASURE_UNDEFINED`, `TP_SPREAD_MEASURE_VALUE` (в единицах цены), `TP_SPREAD_MEASURE_PERCENT` (в процентах, до сотых) |
| clientOrderId | string | no | Уникальный идентификатор заявки (максимум 20 символов). Автоматически генерируется, если не отправлен. |
| validBefore | string (enum) | no | Срок действия: `VALID_BEFORE_UNSPECIFIED`, `VALID_BEFORE_END_OF_DAY` (до конца торгового дня), `VALID_BEFORE_GOOD_TILL_CANCEL` (до отмены), `VALID_BEFORE_GOOD_TILL_DATE` (до указанной даты-времени, только для SL/TP) |
| comment | string | no | Метка заявки (максимум 128 символов) |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 400 | Неверно переданы торговые параметры |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт или инструмент не были найдены |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/accounts/{accountId}/sltp-orders' \
  --request POST \
  --header 'Content-Type: application/json' \
  --header 'Authorization: YOUR_SECRET_TOKEN' \
  --data '{
  "symbol": "",
  "side": "SIDE_UNSPECIFIED",
  "quantitySl": {
    "value": ""
  },
  "slPrice": {
    "value": ""
  },
  "limitPrice": {
    "value": ""
  },
  "quantityTp": {
    "value": ""
  },
  "tpPrice": {
    "value": ""
  },
  "tpGuardSpread": {
    "value": ""
  },
  "tpSpreadMeasure": "TP_SPREAD_MEASURE_UNDEFINED",
  "clientOrderId": "",
  "validBefore": "VALID_BEFORE_UNSPECIFIED",
  "validExpiryTime": "",
  "comment": ""
}'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "order_id": {
      "type": "string",
      "title": "Идентификатор заявки"
    },
    "exec_id": {
      "type": "string",
      "title": "Идентификатор исполнения"
    },
    "status": {
      "type": "string",
      "enum": [
        "ORDER_STATUS_UNSPECIFIED",
        "ORDER_STATUS_NEW",
        "ORDER_STATUS_PARTIALLY_FILLED",
        "ORDER_STATUS_FILLED",
        "ORDER_STATUS_DONE_FOR_DAY",
        "ORDER_STATUS_CANCELED",
        "ORDER_STATUS_REPLACED",
        "ORDER_STATUS_PENDING_CANCEL",
        "ORDER_STATUS_REJECTED",
        "ORDER_STATUS_SUSPENDED",
        "ORDER_STATUS_PENDING_NEW",
        "ORDER_STATUS_EXPIRED",
        "ORDER_STATUS_FAILED",
        "ORDER_STATUS_FORWARDING",
        "ORDER_STATUS_WAIT",
        "ORDER_STATUS_DENIED_BY_BROKER",
        "ORDER_STATUS_REJECTED_BY_EXCHANGE",
        "ORDER_STATUS_WATCHING",
        "ORDER_STATUS_EXECUTED",
        "ORDER_STATUS_DISABLED",
        "ORDER_STATUS_LINK_WAIT",
        "ORDER_STATUS_SL_GUARD_TIME",
        "ORDER_STATUS_SL_EXECUTED",
        "ORDER_STATUS_SL_FORWARDING",
        "ORDER_STATUS_TP_GUARD_TIME",
        "ORDER_STATUS_TP_EXECUTED",
        "ORDER_STATUS_TP_CORRECTION",
        "ORDER_STATUS_TP_FORWARDING",
        "ORDER_STATUS_TP_CORR_GUARD_TIME"
      ],
      "default": "ORDER_STATUS_UNSPECIFIED",
      "description": "- ORDER_STATUS_UNSPECIFIED: Неопределенное значение\n - ORDER_STATUS_NEW: Новая заявка\n - ORDER_STATUS_PARTIALLY_FILLED: Частично исполненная\n - ORDER_STATUS_FILLED: Исполненная\n - ORDER_STATUS_DONE_FOR_DAY: Действует в течение дня\n - ORDER_STATUS_CANCELED: Отменена\n - ORDER_STATUS_REPLACED: Заменена на другую\n - ORDER_STATUS_PENDING_CANCEL: Ожидает отмены\n - ORDER_STATUS_REJECTED: Отклонена\n - ORDER_STATUS_SUSPENDED: Приостановлена\n - ORDER_STATUS_PENDING_NEW: В ожидании новой\n - ORDER_STATUS_EXPIRED: Истекла\n - ORDER_STATUS_FAILED: Ошибка\n - ORDER_STATUS_FORWARDING: Пересылка\n - ORDER_STATUS_WAIT: Ожидает\n - ORDER_STATUS_DENIED_BY_BROKER: Отклонено брокером\n - ORDER_STATUS_REJECTED_BY_EXCHANGE: Отклонено биржей\n - ORDER_STATUS_WATCHING: Наблюдение\n - ORDER_STATUS_EXECUTED: Исполнена\n - ORDER_STATUS_DISABLED: Отключена\n - ORDER_STATUS_LINK_WAIT: Ожидание ссылки\n - ORDER_STATUS_SL_GUARD_TIME: Защитное время SL\n - ORDER_STATUS_SL_EXECUTED: Исполнена по SL\n - ORDER_STATUS_SL_FORWARDING: Пересылка SL\n - ORDER_STATUS_TP_GUARD_TIME: Защитное время TP\n - ORDER_STATUS_TP_EXECUTED: Исполнена по TP\n - ORDER_STATUS_TP_CORRECTION: Коррекция TP\n - ORDER_STATUS_TP_FORWARDING: Пересылка TP\n - ORDER_STATUS_TP_CORR_GUARD_TIME: Коррекция TP в защитное время",
      "title": "Статус заявки"
    },
    "order": {
      "type": "object",
      "properties": {
        "account_id": {
          "type": "string",
          "title": "Идентификатор аккаунта"
        },
        "symbol": {
          "type": "string",
          "title": "Символ инструмента"
        },
        "quantity": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "side": {
          "type": "string",
          "enum": [
            "SIDE_UNSPECIFIED",
            "SIDE_BUY",
            "SIDE_SELL"
          ],
          "default": "SIDE_UNSPECIFIED",
          "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
          "title": "Сторона сделки"
        },
        "type": {
          "type": "string",
          "enum": [
            "ORDER_TYPE_UNSPECIFIED",
            "ORDER_TYPE_MARKET",
            "ORDER_TYPE_LIMIT",
            "ORDER_TYPE_STOP",
            "ORDER_TYPE_STOP_LIMIT",
            "ORDER_TYPE_MULTI_LEG"
          ],
          "default": "ORDER_TYPE_UNSPECIFIED",
          "description": "- ORDER_TYPE_UNSPECIFIED: Значение не указано\n - ORDER_TYPE_MARKET: Рыночная\n - ORDER_TYPE_LIMIT: Лимитная\n - ORDER_TYPE_STOP: Стоп заявка рыночная\n - ORDER_TYPE_STOP_LIMIT: Стоп заявка лимитная\n - ORDER_TYPE_MULTI_LEG: Мульти лег заявка",
          "title": "Тип заявки"
        },
        "time_in_force": {
          "type": "string",
          "enum": [
            "TIME_IN_FORCE_UNSPECIFIED",
            "TIME_IN_FORCE_DAY",
            "TIME_IN_FORCE_GOOD_TILL_CANCEL",
            "TIME_IN_FORCE_GOOD_TILL_CROSSING",
            "TIME_IN_FORCE_EXT",
            "TIME_IN_FORCE_ON_OPEN",
            "TIME_IN_FORCE_ON_CLOSE",
            "TIME_IN_FORCE_IOC",
            "TIME_IN_FORCE_FOK"
          ],
          "default": "TIME_IN_FORCE_UNSPECIFIED",
          "description": "- TIME_IN_FORCE_UNSPECIFIED: Значение не указано\n - TIME_IN_FORCE_DAY: До конца дня\n - TIME_IN_FORCE_GOOD_TILL_CANCEL: Действителен до отмены\n - TIME_IN_FORCE_GOOD_TILL_CROSSING: Действителен до пересечения\n - TIME_IN_FORCE_EXT: Внебиржевая торговля\n - TIME_IN_FORCE_ON_OPEN: На открытии биржи\n - TIME_IN_FORCE_ON_CLOSE: На закрытии биржи\n - TIME_IN_FORCE_IOC: Исполнить немедленно или отменить\n - TIME_IN_FORCE_FOK: Исполнить полностью или отменить",
          "title": "Срок действия заявки"
        },
        "limit_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "stop_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "stop_condition": {
          "type": "string",
          "enum": [
            "STOP_CONDITION_UNSPECIFIED",
            "STOP_CONDITION_LAST_UP",
            "STOP_CONDITION_LAST_DOWN"
          ],
          "default": "STOP_CONDITION_UNSPECIFIED",
          "description": "- STOP_CONDITION_UNSPECIFIED: Значение не указано\n - STOP_CONDITION_LAST_UP: Цена срабатывания больше текущей цены\n - STOP_CONDITION_LAST_DOWN: Цена срабатывания меньше текущей цены",
          "title": "Условие срабатывания стоп заявки"
        },
        "legs": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "symbol": {
                "type": "string",
                "title": "Символ инструмента"
              },
              "quantity": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "side": {
                "type": "string",
                "enum": [
                  "SIDE_UNSPECIFIED",
                  "SIDE_BUY",
                  "SIDE_SELL"
                ],
                "default": "SIDE_UNSPECIFIED",
                "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
                "title": "Сторона сделки"
              }
            },
            "title": "Лег"
          },
          "title": "Необходимо для мульти лег заявки"
        },
        "client_order_id": {
          "type": "string",
          "title": "Уникальный идентификатор заявки. Автоматически генерируется, если не отправлен. (максимум 20 символов)"
        },
        "valid_before": {
          "type": "string",
          "enum": [
            "VALID_BEFORE_UNSPECIFIED",
            "VALID_BEFORE_END_OF_DAY",
            "VALID_BEFORE_GOOD_TILL_CANCEL",
            "VALID_BEFORE_GOOD_TILL_DATE"
          ],
          "default": "VALID_BEFORE_UNSPECIFIED",
          "description": "- VALID_BEFORE_UNSPECIFIED: Значение не указано\n - VALID_BEFORE_END_OF_DAY: До конца торгового дня\n - VALID_BEFORE_GOOD_TILL_CANCEL: До отмены\n - VALID_BEFORE_GOOD_TILL_DATE: До указанной даты-времени. Данный тип поддерживается только при выставлении SL/TP заявок",
          "title": "Срок действия условной заявки"
        },
        "comment": {
          "type": "string",
          "title": "Метка заявки. (максимум 128 символов)"
        }
      },
      "title": "Информация о заявке"
    },
    "transact_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время выставления заявки"
    },
    "accept_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время принятия заявки"
    },
    "withdraw_at": {
      "type": "string",
      "format": "date-time",
      "title": "Дата и время  отмены заявки"
    },
    "initial_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "executed_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "remaining_quantity": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "sltp_order": {
      "type": "object",
      "properties": {
        "account_id": {
          "type": "string",
          "title": "Идентификатор аккаунта"
        },
        "symbol": {
          "type": "string",
          "title": "Символ инструмента"
        },
        "side": {
          "type": "string",
          "enum": [
            "SIDE_UNSPECIFIED",
            "SIDE_BUY",
            "SIDE_SELL"
          ],
          "default": "SIDE_UNSPECIFIED",
          "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
          "title": "Сторона сделки"
        },
        "quantity_sl": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "sl_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "limit_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "quantity_tp": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_price": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_guard_spread": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "tp_spread_measure": {
          "type": "string",
          "enum": [
            "TP_SPREAD_MEASURE_UNDEFINED",
            "TP_SPREAD_MEASURE_VALUE",
            "TP_SPREAD_MEASURE_PERCENT"
          ],
          "default": "TP_SPREAD_MEASURE_UNDEFINED",
          "description": "- TP_SPREAD_MEASURE_UNDEFINED: Значение не указано\n - TP_SPREAD_MEASURE_VALUE: в единицах цены\n - TP_SPREAD_MEASURE_PERCENT: в процентах, с максимальной точностью до сотых процента",
          "title": "Единица измерения величины защитного спреда для цены исполнения TP"
        },
        "client_order_id": {
          "type": "string",
          "title": "Уникальный идентификатор заявки. Автоматически генерируется, если не отправлен. (максимум 20 символов)"
        },
        "valid_before": {
          "type": "string",
          "enum": [
            "VALID_BEFORE_UNSPECIFIED",
            "VALID_BEFORE_END_OF_DAY",
            "VALID_BEFORE_GOOD_TILL_CANCEL",
            "VALID_BEFORE_GOOD_TILL_DATE"
          ],
          "default": "VALID_BEFORE_UNSPECIFIED",
          "description": "- VALID_BEFORE_UNSPECIFIED: Значение не указано\n - VALID_BEFORE_END_OF_DAY: До конца торгового дня\n - VALID_BEFORE_GOOD_TILL_CANCEL: До отмены\n - VALID_BEFORE_GOOD_TILL_DATE: До указанной даты-времени. Данный тип поддерживается только при выставлении SL/TP заявок",
          "title": "Срок действия условной заявки"
        },
        "valid_expiry_time": {
          "type": "string",
          "format": "date-time",
          "title": "Временная метка прекращения действия SL/TP заявки"
        },
        "comment": {
          "type": "string",
          "title": "Метка заявки. (максимум 128 символов)"
        }
      },
      "title": "Информация о SL/TP заявке"
    }
  },
  "title": "Состояние заявки"
}
```

---

## MarketDataService

- [GET /v1/instruments/{symbol}/bars](#получение-исторических-данных-по-инструменту-агрегированные-свечи)
- [GET /v1/instruments/{symbol}/orderbook](#получение-текущего-стакана-по-инструменту)
- [GET /v1/instruments/{symbol}/quotes/latest](#получение-последней-котировки-по-инструменту)
- [GET /v1/instruments/{symbol}/trades/latest](#получение-списка-последних-сделок-по-инструменту)

---

### Получение исторических данных по инструменту (агрегированные свечи)

`GET /v1/instruments/{symbol}/bars`

Параметры:
- `symbol` — передается в URL пути
- `timeframe` и `interval` — передаются как query-параметры

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| symbol | string | yes | Символ инструмента |

**Query Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| timeframe | string (enum) | no | Необходимый таймфрейм. Значения: `TIME_FRAME_UNSPECIFIED` (не указан), `TIME_FRAME_M1` (1 мин, глубина 7 дней), `TIME_FRAME_M5` (5 мин, 30 дней), `TIME_FRAME_M15` (15 мин, 30 дней), `TIME_FRAME_M30` (30 мин, 30 дней), `TIME_FRAME_H1` (1 час, 30 дней), `TIME_FRAME_H2` (2 часа, 30 дней), `TIME_FRAME_H4` (4 часа, 30 дней), `TIME_FRAME_H8` (8 часов, 30 дней), `TIME_FRAME_D` (день, 365 дней), `TIME_FRAME_W` (неделя, 365*5 дней), `TIME_FRAME_MN` (месяц, 365*5 дней), `TIME_FRAME_QR` (квартал, 365*5 дней) |
| interval.start_time | string (date-time) | no | Inclusive start of the interval. If specified, a Timestamp matching this interval will have to be the same or after the start. |
| interval.end_time | string (date-time) | no | Exclusive end of the interval. If specified, a Timestamp matching this interval will have to be before the end. |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 400 | Неверно передан символ или интервал. Символ должен быть в виде ticker@mic. Где ticker — это, например, SBER. А mic, например, MISX |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/instruments/{symbol}/bars?timeframe=TIME_FRAME_UNSPECIFIED&interval.start_time=&interval.end_time=' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "symbol": {
      "type": "string",
      "title": "Символ инструмента"
    },
    "bars": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "timestamp": {
            "type": "string",
            "format": "date-time",
            "title": "Метка времени"
          },
          "open": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "high": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "low": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "close": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "volume": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          }
        },
        "title": "Информация об агрегированной свече"
      },
      "title": "Агрегированная свеча"
    }
  },
  "title": "Список агрегированных свеч"
}
```

---

### Получение текущего стакана по инструменту

`GET /v1/instruments/{symbol}/orderbook`

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| symbol | string | yes | Символ инструмента |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/instruments/{symbol}/orderbook' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "symbol": {
      "type": "string",
      "title": "Символ инструмента"
    },
    "orderbook": {
      "type": "object",
      "properties": {
        "rows": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "price": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "sell_size": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "buy_size": {
                "type": "object",
                "properties": {
                  "value": {
                    "type": "string",
                    "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                  }
                },
                "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
              },
              "action": {
                "type": "string",
                "enum": [
                  "ACTION_UNSPECIFIED",
                  "ACTION_REMOVE",
                  "ACTION_ADD",
                  "ACTION_UPDATE"
                ],
                "default": "ACTION_UNSPECIFIED",
                "description": "- ACTION_UNSPECIFIED: Действие не указано\n - ACTION_REMOVE: Удалить\n - ACTION_ADD: Добавить\n - ACTION_UPDATE: Обновить",
                "title": "Команда"
              },
              "mpid": {
                "type": "string",
                "title": "Идентификатор участника рынка"
              },
              "timestamp": {
                "type": "string",
                "format": "date-time",
                "title": "Метка времени"
              }
            },
            "title": "Информация об уровне в стакане (строке)"
          },
          "title": "Уровни стакана"
        }
      },
      "title": "Информация о стакане"
    }
  },
  "title": "Текущий стакан по инструменту"
}
```

---

### Получение последней котировки по инструменту

`GET /v1/instruments/{symbol}/quotes/latest`

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| symbol | string | yes | Символ инструмента |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/instruments/{symbol}/quotes/latest' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "symbol": {
      "type": "string",
      "title": "Символ инструмента"
    },
    "quote": {
      "type": "object",
      "properties": {
        "symbol": {
          "type": "string",
          "title": "Символ инструмента"
        },
        "timestamp": {
          "type": "string",
          "format": "date-time",
          "title": "Метка времени"
        },
        "ask": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "ask_size": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "bid": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "bid_size": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "last": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "last_size": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "volume": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "turnover": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "open": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "high": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "low": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "close": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "change": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "open_interest": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "option": {
          "type": "object",
          "properties": {
            "open_interest": {
              "type": "object",
              "properties": {
                "value": {
                  "type": "string",
                  "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                }
              },
              "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
            },
            "implied_volatility": {
              "type": "object",
              "properties": {
                "value": {
                  "type": "string",
                  "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                }
              },
              "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
            },
            "theoretical_price": {
              "type": "object",
              "properties": {
                "value": {
                  "type": "string",
                  "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                }
              },
              "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
            },
            "delta": {
              "type": "object",
              "properties": {
                "value": {
                  "type": "string",
                  "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                }
              },
              "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
            },
            "gamma": {
              "type": "object",
              "properties": {
                "value": {
                  "type": "string",
                  "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                }
              },
              "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
            },
            "theta": {
              "type": "object",
              "properties": {
                "value": {
                  "type": "string",
                  "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                }
              },
              "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
            },
            "vega": {
              "type": "object",
              "properties": {
                "value": {
                  "type": "string",
                  "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                }
              },
              "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
            },
            "rho": {
              "type": "object",
              "properties": {
                "value": {
                  "type": "string",
                  "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
                }
              },
              "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
            }
          },
          "title": "Информация об опционе"
        }
      },
      "title": "Информация о котировке"
    }
  },
  "title": "Последняя котировка по инструменту"
}
```

---

### Получение списка последних сделок по инструменту

`GET /v1/instruments/{symbol}/trades/latest`

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| symbol | string | yes | Символ инструмента |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/instruments/{symbol}/trades/latest' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "symbol": {
      "type": "string",
      "title": "Символ инструмента"
    },
    "trades": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "trade_id": {
            "type": "string",
            "title": "Идентификатор сделки, отправленный биржей"
          },
          "mpid": {
            "type": "string",
            "title": "Идентификатор участника рынка"
          },
          "timestamp": {
            "type": "string",
            "format": "date-time",
            "title": "Метка времени"
          },
          "price": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "size": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "side": {
            "type": "string",
            "enum": [
              "SIDE_UNSPECIFIED",
              "SIDE_BUY",
              "SIDE_SELL"
            ],
            "default": "SIDE_UNSPECIFIED",
            "description": "- SIDE_UNSPECIFIED: Сторона сделки не указана\n - SIDE_BUY: Покупка\n - SIDE_SELL: Продажа",
            "title": "Сторона сделки"
          },
          "open_interest": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          }
        },
        "title": "Информация о сделке"
      },
      "title": "Список последних сделок"
    }
  },
  "title": "Список последних сделок по инструменту"
}
```

---

## AssetsService

- [GET /v1/assets](#получение-списка-доступных-инструментов-их-описание)
- [GET /v1/assets/all](#получение-списка-всех-инструментов-их-описание)
- [GET /v1/assets/clock](#получение-времени-на-сервере)
- [GET /v1/assets/{symbol}](#получение-информации-по-конкретному-инструменту)
- [GET /v1/assets/{symbol}/params](#получение-торговых-параметров-по-инструменту)
- [GET /v1/assets/{symbol}/schedule](#получение-расписания-торгов-для-инструмента)
- [GET /v1/assets/{underlyingSymbol}/options](#получение-цепочки-опционов-для-базового-актива)
- [GET /v1/exchanges](#получение-списка-доступных-бирж-названия-и-mic-коды)

---

### Получение списка доступных инструментов, их описание

`GET /v1/assets`

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl https://api.finam.ru/v1/assets \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "assets": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "symbol": {
            "type": "string",
            "title": "Символ инструмента ticker@mic"
          },
          "id": {
            "type": "string",
            "title": "Идентификатор инструмента"
          },
          "ticker": {
            "type": "string",
            "title": "Тикер инструмента"
          },
          "mic": {
            "type": "string",
            "title": "mic идентификатор биржи"
          },
          "isin": {
            "type": "string",
            "title": "Isin идентификатор инструмента"
          },
          "type": {
            "type": "string",
            "title": "Тип инструмента"
          },
          "name": {
            "type": "string",
            "title": "Наименование инструмента"
          },
          "is_archived": {
            "type": "boolean",
            "title": "Архивный инструмент или нет"
          }
        },
        "title": "Информация об инструменте"
      },
      "title": "Информация об инструменте"
    }
  },
  "title": "Список доступных инструментов"
}
```

---

### Получение списка всех инструментов, их описание

`GET /v1/assets/all`

**Query Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| cursor | string (int64) | no | Курсор для пагинации. Указывает sec_id инструмента, с которого должен начинаться список. Для первого запроса оставьте поле пустым (значение 0). Для последующих запросов используйте значение next_cursor из предыдущего ответа. |
| only_active | boolean | no | Фильтрация по статусу инструмента: выбираются только активные (неархивные) инструменты. По умолчанию: false. |
| only_disabled | boolean | no | Фильтрация по статусу инструмента: выбираются только неактивные (архивные) инструменты. По умолчанию: false. |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/assets/all?cursor=&only_active=true&only_disabled=true' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "assets": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "symbol": {
            "type": "string",
            "title": "Символ инструмента ticker@mic"
          },
          "id": {
            "type": "string",
            "title": "Идентификатор инструмента"
          },
          "ticker": {
            "type": "string",
            "title": "Тикер инструмента"
          },
          "mic": {
            "type": "string",
            "title": "mic идентификатор биржи"
          },
          "isin": {
            "type": "string",
            "title": "Isin идентификатор инструмента"
          },
          "type": {
            "type": "string",
            "title": "Тип инструмента"
          },
          "name": {
            "type": "string",
            "title": "Наименование инструмента"
          },
          "is_archived": {
            "type": "boolean",
            "title": "Архивный инструмент или нет"
          }
        },
        "title": "Информация об инструменте"
      },
      "title": "Часть списка инструментов"
    },
    "next_cursor": {
      "type": "string",
      "format": "int64",
      "description": "Курсор для получения следующей страницы. Содержит sec_id последнего инструмента в текущем списке.\nПередайте это значение в поле cursor следующего запроса, чтобы получить следующую часть данных.\nЕсли значение 0 или отсутствует — это последняя страница."
    }
  },
  "description": "Ответ, содержащий часть доступных инструментов."
}
```

---

### Получение времени на сервере

`GET /v1/assets/clock`

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl https://api.finam.ru/v1/assets/clock \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "title": "Метка времени"
    }
  },
  "title": "Время на сервере"
}
```

---

### Получение информации по конкретному инструменту

`GET /v1/assets/{symbol}`

Параметры:
- `symbol` — передается в URL пути
- `account_id` — передаётся как query-параметр

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| symbol | string | yes | Символ инструмента |

**Query Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| account_id | string | no | ID аккаунта для которого будет подбираться информация по инструменту |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 400 | Неверно передан символ или счет. Символ должен быть в виде ticker@mic. Где ticker — это, например, SBER. А mic, например, MISX |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/assets/{symbol}?account_id=' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "board": {
      "type": "string",
      "title": "Код режима торгов"
    },
    "id": {
      "type": "string",
      "title": "Идентификатор инструмента"
    },
    "ticker": {
      "type": "string",
      "title": "Тикер инструмента"
    },
    "mic": {
      "type": "string",
      "title": "mic идентификатор биржи"
    },
    "isin": {
      "type": "string",
      "title": "Isin идентификатор инструмента"
    },
    "type": {
      "type": "string",
      "title": "Тип инструмента"
    },
    "name": {
      "type": "string",
      "title": "Наименование инструмента"
    },
    "decimals": {
      "type": "integer",
      "format": "int32",
      "title": "Кол-во десятичных знаков в цене"
    },
    "min_step": {
      "type": "string",
      "format": "int64",
      "title": "Минимальный шаг цены. Для расчета финального ценового шага: min_step/(10ˆdecimals)"
    },
    "lot_size": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "expiration_date": {
      "type": "object",
      "properties": {
        "year": {
          "type": "integer",
          "format": "int32",
          "description": "Year of the date. Must be from 1 to 9999, or 0 to specify a date without\na year."
        },
        "month": {
          "type": "integer",
          "format": "int32",
          "description": "Month of a year. Must be from 1 to 12, or 0 to specify a year without a\nmonth and day."
        },
        "day": {
          "type": "integer",
          "format": "int32",
          "description": "Day of a month. Must be from 1 to 31 and valid for the year and month, or 0\nto specify a year by itself or a year and month where the day isn't\nsignificant."
        }
      },
      "description": "* A full date, with non-zero year, month, and day values\n* A month and day value, with a zero year, such as an anniversary\n* A year on its own, with zero month and day values\n* A year and month value, with a zero day, such as a credit card expiration\ndate\n\nRelated types are [google.type.TimeOfDay][google.type.TimeOfDay] and\n`google.protobuf.Timestamp`.",
      "title": "Represents a whole or partial calendar date, such as a birthday. The time of\nday and time zone are either specified elsewhere or are insignificant. The\ndate is relative to the Gregorian Calendar. This can represent one of the\nfollowing:"
    },
    "quote_currency": {
      "type": "string",
      "title": "Валюта котировки, может не совпадать с валютой режима торгов инструмента"
    },
    "future_details": {
      "type": "object",
      "properties": {
        "expiration_date": {
          "type": "string",
          "format": "date-time",
          "description": "Дата и время экспирации (исполнения) фьючерсного контракта."
        },
        "contract_size": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        }
      },
      "title": "Специфичные параметры для инструмента типа \"Фьючерс\""
    },
    "option_details": {
      "type": "object",
      "properties": {
        "expiration_date": {
          "type": "string",
          "format": "date-time",
          "description": "Дата и время экспирации (исполнения) опционного контракта."
        },
        "contract_size": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "strike": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        }
      },
      "title": "Специфичные параметры для инструмента типа \"Опцион\""
    },
    "bond_details": {
      "type": "object",
      "properties": {
        "bond_face_value": {
          "type": "object",
          "properties": {
            "value": {
              "type": "string",
              "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
            }
          },
          "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
        },
        "currency": {
          "type": "string",
          "description": "Символьный код валюты номинала облигации (например, RUB, USD)."
        }
      },
      "title": "Специфичные параметры для инструмента типа \"Облигация\""
    }
  },
  "title": "Список информации по конкретному инструменту"
}
```

---

### GetConstituents

> Получить состав биржевого индекса по его символу

`GET /v1/assets/{symbol}/constituents`


**Path Parameters**

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| symbol | string | yes | Символьный код индекса (например, "SPX@_SP", "NDX@_SCI") |

**Query Parameters**

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| cursor | string | no | Курсор для пагинации. Указывает sec_id инструмента, с которого должен начинаться список.
Для первого запроса оставьте поле пустым (значение 0).
Для последующих запросов используйте значение next_cursor из предыдущего ответа. |

**Responses**

| Code | Description |
| --- | --- |
| 200 | A successful response |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | An unexpected error response |

**Response Example(200)**

```json
{
  "constituents": [
    {
      "symbol": "string",
      "name": "string",
      "sector": "string",
      "sub_sector": "string",
      "cik": "string"
    }
  ],
  "next_cursor": "string"
}
```


**Request Example**

```shell
TOKEN=$(curl -s -X POST 'https://api.finam.ru/v1/sessions' \
  -H 'Content-Type: application/json' \
  -d '{"secret":"YOUR_API_TOKEN"}' | jq -r '.token')

curl -X GET 'https://api.finam.ru/v1/assets/YOUR_UNDERLYING_SYMBOL/options?root=YOUR_ROOT&expiration_date.year=YOUR_EXPIRATION_DATE.YEAR&expiration_date.month=YOUR_EXPIRATION_DATE.MONTH&expiration_date.day=YOUR_EXPIRATION_DATE.DAY' \
  -H "Authorization: Bearer $TOKEN"
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "constituents": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "symbol": {
            "type": "string",
            "title": "Символьный код инструмента"
          },
          "name": {
            "type": "string",
            "title": "Полное наименование компании-эмитента"
          },
          "sector": {
            "type": "string",
            "title": "Глобальный сектор экономики, к которому относится компания (например, \"Technology\", \"Healthcare\")"
          },
          "sub_sector": {
            "type": "string",
            "title": "Отрасль (подотрасль) деятельности компании (например, \"Software - Application\")"
          },
          "cik": {
            "type": "string",
            "title": "Уникальный идентификатор компании в базе данных SEC США (Central Index Key)"
          }
        },
        "title": "Информация о компоненте (ценной бумаге), входящем в индекс"
      },
      "title": "Список компонентов (ценных бумаг), входящих в базу расчета запрошенного индекса"
    },
    "next_cursor": {
      "type": "string",
      "format": "int64",
      "title": "Курсор для получения следующей страницы. Содержит sec_id последнего инструмента в текущем списке.\nПередайте это значение в поле cursor следующего запроса, чтобы получить следующую часть данных.\nЕсли значение 0 или отсутствует — это последняя страница"
    }
  },
  "title": "Результат запроса состава индекса"
}
```

### Получение торговых параметров по инструменту

`GET /v1/assets/{symbol}/params`

Параметры:
- `symbol` — передается в URL пути
- `account_id` — передаётся как query-параметр

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| symbol | string | yes | Символ инструмента |

**Query Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| account_id | string | no | ID аккаунта для которого будут подбираться торговые параметры |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 400 | Неверно передан символ или счет. Символ должен быть в виде ticker@mic. Где ticker — это, например, SBER. А mic, например, MISX |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/assets/{symbol}/params?account_id=' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "symbol": {
      "type": "string",
      "title": "Символ инструмента"
    },
    "account_id": {
      "type": "string",
      "title": "ID аккаунта для которого подбираются торговые параметры"
    },
    "tradeable": {
      "type": "boolean",
      "description": "Доступны ли торговые операции\nСтарое поле, помечено как устаревшее.\nКлиентам следует перейти на is_tradeable."
    },
    "longable": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "enum": [
            "NOT_AVAILABLE",
            "AVAILABLE",
            "ACCOUNT_NOT_APPROVED"
          ],
          "default": "NOT_AVAILABLE",
          "description": "- NOT_AVAILABLE: Не доступен\n - AVAILABLE: Доступен\n - ACCOUNT_NOT_APPROVED: Запрещено на уровне счета",
          "title": "Статус"
        },
        "halted_days": {
          "type": "integer",
          "format": "int32",
          "title": "Сколько дней действует запрет на операции в Лонг (если есть)"
        }
      },
      "title": "Доступны ли операции в Лонг"
    },
    "shortable": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "enum": [
            "NOT_AVAILABLE",
            "AVAILABLE",
            "HTB",
            "ACCOUNT_NOT_APPROVED",
            "AVAILABLE_STRATEGY"
          ],
          "default": "NOT_AVAILABLE",
          "description": "- NOT_AVAILABLE: Не доступен\n - AVAILABLE: Доступен\n - HTB: Признак того, что бумага Hard To Borrow (если есть)\n - ACCOUNT_NOT_APPROVED: Запрещено на уровне счета\n - AVAILABLE_STRATEGY: Разрешено в составе стратегии",
          "title": "Статус"
        },
        "halted_days": {
          "type": "integer",
          "format": "int32",
          "title": "Сколько дней действует запрет на операции в Шорт (если есть)"
        }
      },
      "title": "Доступны ли операции в Шорт"
    },
    "long_risk_rate": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "long_collateral": {
      "type": "object",
      "properties": {
        "currency_code": {
          "type": "string",
          "description": "The three-letter currency code defined in ISO 4217."
        },
        "units": {
          "type": "string",
          "format": "int64",
          "description": "The whole units of the amount.\nFor example if `currencyCode` is `\"USD\"`, then 1 unit is one US dollar."
        },
        "nanos": {
          "type": "integer",
          "format": "int32",
          "description": "Number of nano (10^-9) units of the amount.\nThe value must be between -999,999,999 and +999,999,999 inclusive.\nIf `units` is positive, `nanos` must be positive or zero.\nIf `units` is zero, `nanos` can be positive, zero, or negative.\nIf `units` is negative, `nanos` must be negative or zero.\nFor example $-1.75 is represented as `units`=-1 and `nanos`=-750,000,000."
        }
      },
      "description": "Represents an amount of money with its currency type."
    },
    "short_risk_rate": {
      "type": "object",
      "properties": {
        "value": {
          "type": "string",
          "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
        }
      },
      "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
    },
    "short_collateral": {
      "type": "object",
      "properties": {
        "currency_code": {
          "type": "string",
          "description": "The three-letter currency code defined in ISO 4217."
        },
        "units": {
          "type": "string",
          "format": "int64",
          "description": "The whole units of the amount.\nFor example if `currencyCode` is `\"USD\"`, then 1 unit is one US dollar."
        },
        "nanos": {
          "type": "integer",
          "format": "int32",
          "description": "Number of nano (10^-9) units of the amount.\nThe value must be between -999,999,999 and +999,999,999 inclusive.\nIf `units` is positive, `nanos` must be positive or zero.\nIf `units` is zero, `nanos` can be positive, zero, or negative.\nIf `units` is negative, `nanos` must be negative or zero.\nFor example $-1.75 is represented as `units`=-1 and `nanos`=-750,000,000."
        }
      },
      "description": "Represents an amount of money with its currency type."
    },
    "long_initial_margin": {
      "type": "object",
      "properties": {
        "currency_code": {
          "type": "string",
          "description": "The three-letter currency code defined in ISO 4217."
        },
        "units": {
          "type": "string",
          "format": "int64",
          "description": "The whole units of the amount.\nFor example if `currencyCode` is `\"USD\"`, then 1 unit is one US dollar."
        },
        "nanos": {
          "type": "integer",
          "format": "int32",
          "description": "Number of nano (10^-9) units of the amount.\nThe value must be between -999,999,999 and +999,999,999 inclusive.\nIf `units` is positive, `nanos` must be positive or zero.\nIf `units` is zero, `nanos` can be positive, zero, or negative.\nIf `units` is negative, `nanos` must be negative or zero.\nFor example $-1.75 is represented as `units`=-1 and `nanos`=-750,000,000."
        }
      },
      "description": "Represents an amount of money with its currency type."
    },
    "short_initial_margin": {
      "type": "object",
      "properties": {
        "currency_code": {
          "type": "string",
          "description": "The three-letter currency code defined in ISO 4217."
        },
        "units": {
          "type": "string",
          "format": "int64",
          "description": "The whole units of the amount.\nFor example if `currencyCode` is `\"USD\"`, then 1 unit is one US dollar."
        },
        "nanos": {
          "type": "integer",
          "format": "int32",
          "description": "Number of nano (10^-9) units of the amount.\nThe value must be between -999,999,999 and +999,999,999 inclusive.\nIf `units` is positive, `nanos` must be positive or zero.\nIf `units` is zero, `nanos` can be positive, zero, or negative.\nIf `units` is negative, `nanos` must be negative or zero.\nFor example $-1.75 is represented as `units`=-1 and `nanos`=-750,000,000."
        }
      },
      "description": "Represents an amount of money with its currency type."
    },
    "is_tradable": {
      "type": "boolean",
      "description": "Доступны ли торговые операции\nНовое поле. Позволяет различать false и \"не установлено\"."
    },
    "price_type": {
      "type": "string",
      "enum": [
        "UNKNOWN",
        "POSITIVE",
        "NON_NEGATIVE",
        "ANY"
      ],
      "default": "UNKNOWN",
      "description": "- UNKNOWN: Неизвестно\n - POSITIVE: Положительная. Больше нуля\n - NON_NEGATIVE: Неотрицательная. Больше или равна нулю\n - ANY: Любая",
      "title": "Допустимая цена"
    }
  },
  "title": "Торговые параметры инструмента"
}
```

---

### Получение расписания торгов для инструмента

`GET /v1/assets/{symbol}/schedule`

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| symbol | string | yes | Символ инструмента |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 400 | Неверно передан символ. Символ должен быть в виде ticker@mic. Где ticker — это, например, SBER. А mic, например, MISX |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/assets/{symbol}/schedule' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "symbol": {
      "type": "string",
      "title": "Символ инструмента"
    },
    "sessions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "type": {
            "type": "string",
            "title": "Тип сессии"
          },
          "interval": {
            "type": "object",
            "properties": {
              "start_time": {
                "type": "string",
                "format": "date-time",
                "description": "Optional. Inclusive start of the interval.\n\nIf specified, a Timestamp matching this interval will have to be the same\nor after the start."
              },
              "end_time": {
                "type": "string",
                "format": "date-time",
                "description": "Optional. Exclusive end of the interval.\n\nIf specified, a Timestamp matching this interval will have to be before the\nend."
              }
            },
            "description": "Represents a time interval, encoded as a Timestamp start (inclusive) and a\nTimestamp end (exclusive).\n\nThe start must be less than or equal to the end.\nWhen the start equals the end, the interval is empty (matches no time).\nWhen both start and end are unspecified, the interval matches any time."
          }
        },
        "title": "Сессии"
      },
      "title": "Сессии инструмента"
    }
  },
  "title": "Расписание инструмента"
}
```

---

### Получение цепочки опционов для базового актива

`GET /v1/assets/{underlyingSymbol}/options`

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| underlyingSymbol | string | yes | Символ базового актива опциона |

**Query Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| root | string | no | Опциональный параметр. Актуален для опционов на фьючерсы, по типу (недельные, месячные). Если параметр не указан, будут возвращены опционы с ближайшей датой экспирации. |
| expirationDate.year | integer (int32) | no | Year of the date. Must be from 1 to 9999, or 0 to specify a date without a year. |
| expirationDate.month | integer (int32) | no | Month of a year. Must be from 1 to 12, or 0 to specify a year without a month and day. |
| expirationDate.day | integer (int32) | no | Day of a month. Must be from 1 to 31 and valid for the year and month, or 0 to specify a year by itself or a year and month where the day isn't significant. |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 400 | Неверно передан символ. Символ должен быть в виде ticker@mic. Где ticker — это, например, SBER. А mic, например, MISX |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/assets/{underlyingSymbol}/options?root=&expirationDate.year=1&expirationDate.month=1&expirationDate.day=1' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "symbol": {
      "type": "string",
      "title": "Символ базового актива опциона"
    },
    "options": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "symbol": {
            "type": "string",
            "title": "Символ инструмента"
          },
          "type": {
            "type": "string",
            "enum": [
              "TYPE_UNSPECIFIED",
              "TYPE_CALL",
              "TYPE_PUT"
            ],
            "default": "TYPE_UNSPECIFIED",
            "description": "- TYPE_UNSPECIFIED: Неопределенное значение\n - TYPE_CALL: Колл\n - TYPE_PUT: Пут",
            "title": "Тип опциона"
          },
          "contract_size": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "trade_first_day": {
            "type": "object",
            "properties": {
              "year": {
                "type": "integer",
                "format": "int32",
                "description": "Year of the date. Must be from 1 to 9999, or 0 to specify a date without\na year."
              },
              "month": {
                "type": "integer",
                "format": "int32",
                "description": "Month of a year. Must be from 1 to 12, or 0 to specify a year without a\nmonth and day."
              },
              "day": {
                "type": "integer",
                "format": "int32",
                "description": "Day of a month. Must be from 1 to 31 and valid for the year and month, or 0\nto specify a year by itself or a year and month where the day isn't\nsignificant."
              }
            },
            "description": "* A full date, with non-zero year, month, and day values\n* A month and day value, with a zero year, such as an anniversary\n* A year on its own, with zero month and day values\n* A year and month value, with a zero day, such as a credit card expiration\ndate\n\nRelated types are [google.type.TimeOfDay][google.type.TimeOfDay] and\n`google.protobuf.Timestamp`.",
            "title": "Represents a whole or partial calendar date, such as a birthday. The time of\nday and time zone are either specified elsewhere or are insignificant. The\ndate is relative to the Gregorian Calendar. This can represent one of the\nfollowing:"
          },
          "trade_last_day": {
            "type": "object",
            "properties": {
              "year": {
                "type": "integer",
                "format": "int32",
                "description": "Year of the date. Must be from 1 to 9999, or 0 to specify a date without\na year."
              },
              "month": {
                "type": "integer",
                "format": "int32",
                "description": "Month of a year. Must be from 1 to 12, or 0 to specify a year without a\nmonth and day."
              },
              "day": {
                "type": "integer",
                "format": "int32",
                "description": "Day of a month. Must be from 1 to 31 and valid for the year and month, or 0\nto specify a year by itself or a year and month where the day isn't\nsignificant."
              }
            },
            "description": "* A full date, with non-zero year, month, and day values\n* A month and day value, with a zero year, such as an anniversary\n* A year on its own, with zero month and day values\n* A year and month value, with a zero day, such as a credit card expiration\ndate\n\nRelated types are [google.type.TimeOfDay][google.type.TimeOfDay] and\n`google.protobuf.Timestamp`.",
            "title": "Represents a whole or partial calendar date, such as a birthday. The time of\nday and time zone are either specified elsewhere or are insignificant. The\ndate is relative to the Gregorian Calendar. This can represent one of the\nfollowing:"
          },
          "strike": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "multiplier": {
            "type": "object",
            "properties": {
              "value": {
                "type": "string",
                "description": "The decimal value, as a string.\n\nThe string representation consists of an optional sign, `+` (`U+002B`)\nor `-` (`U+002D`), followed by a sequence of zero or more decimal digits\n(\"the integer\"), optionally followed by a fraction, optionally followed\nby an exponent.\n\nThe fraction consists of a decimal point followed by zero or more decimal\ndigits. The string must contain at least one digit in either the integer\nor the fraction. The number formed by the sign, the integer and the\nfraction is referred to as the significand.\n\nThe exponent consists of the character `e` (`U+0065`) or `E` (`U+0045`)\nfollowed by one or more decimal digits.\n\nServices **should** normalize decimal values before storing them by:\n\n  - Removing an explicitly-provided `+` sign (`+2.5` -> `2.5`).\n  - Replacing a zero-length integer value with `0` (`.5` -> `0.5`).\n  - Coercing the exponent character to lower-case (`2.5E8` -> `2.5e8`).\n  - Removing an explicitly-provided zero exponent (`2.5e0` -> `2.5`).\n\nServices **may** perform additional normalization based on its own needs\nand the internal decimal implementation selected, such as shifting the\ndecimal point and exponent value together (example: `2.5e-1` <-> `0.25`).\nAdditionally, services **may** preserve trailing zeroes in the fraction\nto indicate increased precision, but are not required to do so.\n\nNote that only the `.` character is supported to divide the integer\nand the fraction; `,` **should not** be supported regardless of locale.\nAdditionally, thousand separators **should not** be supported. If a\nservice does support them, values **must** be normalized.\n\nThe ENBF grammar is:\n\n    DecimalString =\n      [Sign] Significand [Exponent];\n\n    Sign = '+' | '-';\n\n    Significand =\n      Digits ['.'] [Digits] | [Digits] '.' Digits;\n\n    Exponent = ('e' | 'E') [Sign] Digits;\n\n    Digits = { '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' };\n\nServices **should** clearly document the range of supported values, the\nmaximum supported precision (total number of digits), and, if applicable,\nthe scale (number of digits after the decimal point), as well as how it\nbehaves when receiving out-of-bounds values.\n\nServices **may** choose to accept values passed as input even when the\nvalue has a higher precision or scale than the service supports, and\n**should** round the value to fit the supported scale. Alternatively, the\nservice **may** error with `400 Bad Request` (`INVALID_ARGUMENT` in gRPC)\nif precision would be lost.\n\nServices **should** error with `400 Bad Request` (`INVALID_ARGUMENT` in\ngRPC) if the service receives a value outside of the supported range."
              }
            },
            "description": "A representation of a decimal value, such as 2.5. Clients may convert values\ninto language-native decimal formats, such as Java's [BigDecimal][] or\nPython's [decimal.Decimal][].\n\n[BigDecimal]:\nhttps://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html\n[decimal.Decimal]: https://docs.python.org/3/library/decimal.html"
          },
          "expiration_first_day": {
            "type": "object",
            "properties": {
              "year": {
                "type": "integer",
                "format": "int32",
                "description": "Year of the date. Must be from 1 to 9999, or 0 to specify a date without\na year."
              },
              "month": {
                "type": "integer",
                "format": "int32",
                "description": "Month of a year. Must be from 1 to 12, or 0 to specify a year without a\nmonth and day."
              },
              "day": {
                "type": "integer",
                "format": "int32",
                "description": "Day of a month. Must be from 1 to 31 and valid for the year and month, or 0\nto specify a year by itself or a year and month where the day isn't\nsignificant."
              }
            },
            "description": "* A full date, with non-zero year, month, and day values\n* A month and day value, with a zero year, such as an anniversary\n* A year on its own, with zero month and day values\n* A year and month value, with a zero day, such as a credit card expiration\ndate\n\nRelated types are [google.type.TimeOfDay][google.type.TimeOfDay] and\n`google.protobuf.Timestamp`.",
            "title": "Represents a whole or partial calendar date, such as a birthday. The time of\nday and time zone are either specified elsewhere or are insignificant. The\ndate is relative to the Gregorian Calendar. This can represent one of the\nfollowing:"
          },
          "expiration_last_day": {
            "type": "object",
            "properties": {
              "year": {
                "type": "integer",
                "format": "int32",
                "description": "Year of the date. Must be from 1 to 9999, or 0 to specify a date without\na year."
              },
              "month": {
                "type": "integer",
                "format": "int32",
                "description": "Month of a year. Must be from 1 to 12, or 0 to specify a year without a\nmonth and day."
              },
              "day": {
                "type": "integer",
                "format": "int32",
                "description": "Day of a month. Must be from 1 to 31 and valid for the year and month, or 0\nto specify a year by itself or a year and month where the day isn't\nsignificant."
              }
            },
            "description": "* A full date, with non-zero year, month, and day values\n* A month and day value, with a zero year, such as an anniversary\n* A year on its own, with zero month and day values\n* A year and month value, with a zero day, such as a credit card expiration\ndate\n\nRelated types are [google.type.TimeOfDay][google.type.TimeOfDay] and\n`google.protobuf.Timestamp`.",
            "title": "Represents a whole or partial calendar date, such as a birthday. The time of\nday and time zone are either specified elsewhere or are insignificant. The\ndate is relative to the Gregorian Calendar. This can represent one of the\nfollowing:"
          }
        },
        "title": "Информация об опционе"
      },
      "title": "Информация об опционе"
    }
  },
  "title": "Информация о цепочке опционов"
}
```

---

### Получение списка доступных бирж, названия и mic коды

`GET /v1/exchanges`

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl https://api.finam.ru/v1/exchanges \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "exchanges": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "mic": {
            "type": "string",
            "title": "Идентификатор биржи mic"
          },
          "name": {
            "type": "string",
            "title": "Наименование биржи"
          }
        },
        "title": "Информация о бирже"
      },
      "title": "Информация о бирже"
    }
  },
  "title": "Список доступных бирж"
}
```

---

## UsageMetricsService

- [GET /v1/usage](#получение-текущих-метрик-использования-для-пользователя)

---

### Получение текущих метрик использования для пользователя

`GET /v1/usage`

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl https://api.finam.ru/v1/usage \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "quotas": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": {
            "type": "string",
            "title": "Название метода"
          },
          "limit": {
            "type": "string",
            "format": "int64",
            "description": "Общий лимит по данной квоте."
          },
          "remaining": {
            "type": "string",
            "format": "int64",
            "description": "Сколько осталось доступных единиц в текущем окне."
          },
          "reset_time": {
            "type": "string",
            "format": "date-time",
            "description": "Время, когда счетчик квоты будет сброшен (начало нового окна)."
          }
        },
        "title": "Квота"
      },
      "description": "Список текущих квот и их использование."
    }
  },
  "title": "Информация о текущих метриках использования"
}
```

---

## ReportsService

- [POST /v1/report](#запустить-генерацию-отчета-по-счету-за-период)
- [GET /v1/report/{reportId}/info](#получение-информации-о-результате-генерации-отчета-по-счету)

---

### Запустить генерацию отчета по счету за период

`POST /v1/report`

**Body** (required, application/json)

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| accountId | string (int64) | yes | Идентификатор счета |
| dateRange | object | yes | Временной интервал |
| reportForm | string (enum) | no | Форма отчета: `REPORT_FORM_UNKNOWN` (не указана), `REPORT_FORM_SHORT` (краткая), `REPORT_FORM_LONG` (полная) |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl https://api.finam.ru/v1/report \
  --request POST \
  --header 'Content-Type: application/json' \
  --header 'Authorization: YOUR_SECRET_TOKEN' \
  --data '{
  "dateRange": {
    "dateBegin": "",
    "dateEnd": ""
  },
  "reportForm": "REPORT_FORM_UNKNOWN",
  "accountId": ""
}'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "report_id": {
      "type": "string",
      "title": "Идентификатор отчёта"
    }
  },
  "title": "Информация о генерируемом отчёте"
}
```

---

### Получение информации о результате генерации отчета по счету

`GET /v1/report/{reportId}/info`

**Path Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| reportId | string | yes | Идентификатор отчёта |

**Responses**

| Code | Description |
| --- | --- |
| 200 | Успешный ответ |
| 401 | Срок действия токена истек или токен недействителен |
| 404 | Счёт не был найден в токене |
| 429 | Слишком много запросов. Доступный лимит - 200 запросов в минуту |
| 500 | Внутренняя ошибка сервиса. Попробуйте позже |
| 503 | Сервис на данный момент не доступен. Попробуйте позже |
| 504 | Крайний срок истек до завершения операции |
| default | Непредвиденная ошибка |

**Request Example**

```curl
curl 'https://api.finam.ru/v1/report/{reportId}/info' \
  --header 'Authorization: YOUR_SECRET_TOKEN'
```

**Response Schema (200)**

```json
{
  "type": "object",
  "properties": {
    "info": {
      "type": "object",
      "properties": {
        "report_id": {
          "type": "string",
          "title": "Идентификатор отчёта"
        },
        "status": {
          "type": "string",
          "enum": [
            "NOT_FOUND",
            "PENDING",
            "IN_PROGRESS",
            "SUCCESS",
            "ERROR"
          ],
          "default": "NOT_FOUND",
          "description": "- NOT_FOUND: Не найден\n - PENDING: Ожидает генерации\n - IN_PROGRESS: Генерация запущена\n - SUCCESS: Генерация завершена успешно\n - ERROR: Генерация завершена с ошибкой",
          "title": "Статус генерации отчёта"
        },
        "date_range": {
          "type": "object",
          "properties": {
            "date_begin": {
              "type": "string",
              "format": "date-time",
              "title": "Дата начала временного интервала"
            },
            "date_end": {
              "type": "string",
              "format": "date-time",
              "title": "Дата конца временного интервала"
            }
          },
          "title": "Временной интервал"
        },
        "report_form": {
          "type": "string",
          "enum": [
            "REPORT_FORM_UNKNOWN",
            "REPORT_FORM_SHORT",
            "REPORT_FORM_LONG"
          ],
          "default": "REPORT_FORM_UNKNOWN",
          "description": "- REPORT_FORM_UNKNOWN: Не указана\n - REPORT_FORM_SHORT: Краткая\n - REPORT_FORM_LONG: Полная",
          "title": "Форма отчета"
        },
        "account_id": {
          "type": "string",
          "format": "int64",
          "title": "Идентификатор счета. Берётся из запроса на генерацию отчёта"
        },
        "url": {
          "type": "string",
          "title": "Ссылка на скачивание отчёта. Появляется только в случае успешной генерации отчёта (ReportCreationStatus = SUCCESS)"
        }
      },
      "title": "Информация о статусе генерации отчёта"
    }
  },
  "title": "Информация о статусе генерации отчёта"
}
```
