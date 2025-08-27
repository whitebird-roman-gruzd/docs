# WhiteBird Card API guide

## Назначение
Запросы для партнёров на список доступных платежных средств клиента, а также возможность добавления новых платёжных средств

### Описание
Для совершения операций партнёр может выступать в роли [paymentProvider](../paymentProviderAPI) - это глубокая интеграция платежных методов с целью избежания комиссии со стороны PSP.

Также партнёр может использовать текущие платёжные системы, с которыми уже есть интеграции у WhiteBird. Для этого пользователь в приложении партнёра добавляет платёжные карты через наше API ниже

Авторизация запросов происходит через **x-api-key** header. все запросы Backend to Backend

### Разделы:
- **[Получение списка доступных провайдеров](#providers-request)**
- **[Добавление карты](#card-bind-request)**
- **[Получение списка карт](#payment-methods-request)**

### Providers request

> Список доступных способов оплаты с их характеристиками

[Документация](../exchangeAPI#providers-request)

---

### Card bind request

#### POST /api/v2/exchange/merchant/payment/card/bind
> Добавление новой карты через выбранного провайдера, на странице PSP (максимально можно добавить 5 карт)
#### params:
- **clientId** - string(255), registered client id
- **providerType** - [PaymentProviderId](../exchangeAPI#PaymentProvider-interfaces.), провайдер для добавления карты
- **returnUrl** - string, страница на которую должна перевести PSP после добавления карты пользователем

#### response:
- **url** - String, ссылка на страницу PSP для добавления карты
>Есть список карт для тестовой среды.
---

### Payment methods request

#### POST /api/v2/exchange/merchant/payment/method
> получение доступных платёжных методов пользователя. 
#### params:
- **clientId** - string(255), registered client id
- **fiatAsset** - [CurrencyCode](../exchangeAPI#common-interfaces.), валюта карты
- **orderType** - "BUY" | "SELL", тип операции, так как платежные средства могут иметь ограничения направлений

#### response:
- **[PaymentMethod](#interfaces)[]**
---

#### Interfaces
```typescript
interface PaymentMethod {
    providerType: PaymentProviderId,
    id: "string",                   // платёжный токен карты
    number: "string",               // маска карты, последние 4 цифры
    brand: PaymentSystemType,       // тип платёжной системы, со страницы exchangeAPI
    status: PaymentMethodStatus     // статус карты
}

enum PaymentMethodStatus {
    ACTIVE = "ACTIVE",
    // ... ???
}

```