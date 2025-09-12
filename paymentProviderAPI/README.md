# WhiteBird Payment Provider API guide

## Назначение

Фиатный PaymentProvider - система осуществляющая фиатный рассчет с пользователями по поручению WhiteBird.

Партнер может выступать в роли фиатного PaymentProvider для WhiteBird.
Это означает, что фиатные рассчеты с клиентом выполняет партнёр по поручению криптоплатформы

Для обеспечения работы таких платежей партнер должен иметь возможность принимать платежные поручения на списание средств с пользователя в пользу WhiteBird, а также зачислять средства в пользу пользователя с WhiteBird.

Для того чтобы поддерживать данный способ платежей партнёр должен интегрировать 1-2 API запроса.

### PaymentProvider POST запрос
> запрос на списание/зачислений средств на баланс пользователя

#### Любой https URL с API_KEY для авторизации.

#### Request params:
- **externalUserId** - string, client id в системе WhiteBird.
- **externalOperationId** - string, id операции в рамках, которой даём поручение
- **paymentMethodToken** - string, optional id платежного средства, необходим если партнер предоставляет несколько платежных средств
- **currency** - int, код валюты по ISO 4217
- **amount** - double, количество средств для движения в рамках поручения
- **type** - PaymentType, тип операции
- **status** - "EXCHANGED" - единтсвенное возможное значение, описывающее характер операции

Request example:
``` json
{
  "externalUserId": "9ee9e9f7-a7ff-4dc4-9804-5d7dd2cf2df5",
  "externalOperationId": "c2cd82d4-7653-4a0f-a861-f8b1aa45816c",
  "paymentMethodToken": "4b13260e-e642-438f-ab40-adc6a1edb039",
  "currency": 933,
  "amount": 100,
  "type": "PAYMENT",
  "status": "EXCHANGED"
}
```

#### Response params:
- **status** - PaymentStatus,
- **errorMessage** - string, описание ошибки при возникновении

Response example:
``` json
{
  "status": "FAILED",
  "errorMessage": "Недостаточно средств на балансе"
}
```

### PaymentProvider Timeout POST запрос
> уведомления о нашей отмене операции(по истечении времени или пользователь отменил)

#### Любой https URL с API_KEY для авторизации.

#### Request params:
- **externalOperationId** - string, id операции в рамках, которой даём поручение (order.fiatTransaction.orderIdentity)
- **status** - "СANCELLED" - единственное возможное значение, описывающее характер операции

Request example:
``` json
{
  "externalOperationId": "c2cd82d4-7653-4a0f-a861-f8b1aa45816c",
  "status": "СANCELLED"
}
```

Response - 200 OK

#### Типы данных

```typescript
enum PaymentType {
    PAYMENT = "PAYMENT",            // пополнение баланса пользователя
    DISBURSEMENT = "DISBURSEMENT"   // списание с баланса пользователя
}

enum PaymentStatus {
    PROCESSING = "PROCESSING",  // операция в обработке, мы со своей стороны будем пробовать снова до получения финального статуса
    SUCCESS = "SUCCESS",        // операция прошла успешно
    FAILED = "FAILED"           // не удалось совершить операцию, а данном случае ожидаем errorMessage.
}
```
