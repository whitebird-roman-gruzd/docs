# Whitebird API guide

## Process description

![ExchangeFlow](exchangeApiFlow.drawio.svg)

## Content:
- **[Assets](#assets-request)**
- **[Providers](#providers-request)**
- **[Limits](#Limit-request)**
- **[Quote](#Quote-request)**
- **[Buy](#buy-request---onramp)**
- **[Sell](#sell-request---offramp)**

### Assets request

#### POST /api/v2/exchange/merchant/assets
> Доступные для обмена валюты.
#### params:
- **clientId** - string(255), registered client id

#### response:
- **cryptoAssets** - [`CryptoAsset`](#common-interfaces), доступные криптовалюты
- **fiatAssets** - [`FiatAsset`](#common-interfaces), доступные фиатные валюты
---
### Providers request

#### POST /api/v2/exchange/merchant/payment/provider
> Список доступных способов оплаты с их характеристиками
#### params:
- **clientId** - string(255), registered client id

#### response:
- **Array** - [PaymentProvider[]](#paymentprovider-interfaces)
---

### Limit request

#### POST /api/v2/exchange/merchant/limit
> Текущие лимиты клиента по выбранной паре 
#### params:
- **clientId** - string(255), registered client id
- **paymentMethod** - PaymentProviderId, какой paymentProvider используем
- **fromAsset** - LimitAsset, из какой валюты обмен
- **toAsset** - LimitAsset, в какую валюту обмен

#### response:
- **asset** - LimitAsset, копия  fromAsset
- **min** - int, минимальное количество входящей валюты для обмена
- **max** - int, максимальное количество входящей валюты для обмена

#### Пример запросов:
``` json
// OnRamp example
{
    "clientId": "a03a693e-d6b4-4692-b23d-ccb246392431",
    "paymentMethod": "ASSIST",
    "fromAsset": {
        "code": "BYN"
    },
    "toAsset": {
        "code": "USDT",
        "network": "Ton"
    }
}

// OffRamp example
{
  "clientId": "a03a693e-d6b4-4692-b23d-ccb246392431",
  "paymentMethod": "ASSIST",
  "fromAsset": {
    "code": "USDT",
    "network": "Ton"
  },
  "toAsset": {
    "code": "BYN"
  }
}
```
---

### Quote request

#### POST /api/v2/exchange/merchant/quote
> Запрос на условия сделки. время жизни условий сделки 30сек, нужно запрашивать постоянно, нельзя создать ордер на обмен с истёкшей квотой. В ответе - условия текущего обмена с детализацией и quoteId  
#### params:
- **clientId** - string(255), registered client id
- **paymentMethod** - PaymentProviderId, какой paymentProvider используем
- **paymentMethodToken** - string(255), токен карты или любой GUID(для кастомных интеграций)
- **fromAsset** - QuoteAsset, из какой валюты обмен
- **toAsset** - QuoteAsset, в какую валюту обмен

#### response:
- **quoteId** - string(255), **Id условий сделки**, с этим ID создавать запрос на покупку/продажу
- **fromAsset** - QuoteAsset, рассчитанное количество валюты для обмена
- **toAsset** - QuoteAsset, рассчитанное количество валюты для обмена
- **rate** - double, итоговый курс с учётом всех комиссий(сервиса и сети)
- **plainRate** - double, курс без учёта комиссий
- **fee** - QuoteFee, комиссии в текущей сделке
- **expirationDate** - string(255), дата истечения условий сделки (now + 30sec)

#### Пример запросов:
``` json
// OnRamp example
{
    "clientId": "a03a693e-d6b4-4692-b23d-ccb246392431"
    "paymentMethod": "TEST"
    "paymentMethodToken": "a13a693e-c6b4-4692-b23d-cdb246392453"
    "fromAsset": {
        "amount": 100,
        "code": "BYN"
    },
    "toAsset": {
        "code": "USDT",
        "network": "Ton"
    }
}

// OffRamp example
{
    "clientId": "a03a693e-d6b4-4692-b23d-ccb246392431"
    "paymentMethod": "TEST"
    "paymentMethodToken": "a13a693e-c6b4-4692-b23d-cdb246392453"
    "fromAsset": {
        "amount": 100,
        "code": "USDT",
        "network": "Ton"
    },
    "toAsset": {
        "code": "USD",
    }
}
```
---

### Buy request - OnRamp

#### POST /api/v2/exchange/merchant/buy
> Запрос на создание заявки для покупки криптовалюты за фиат. **OnRamp** операция.
#### params:
- **clientId** - string(255), registered client id
- **quoteId** - string(255), ID условий сделки
- **destinationCryptoAddress** - string(255), крипто-адрес, на который мы отправим криптовалюту(клиент подтверждает, что кошелёк принадлежит ему) 

#### response:
- **id** - string(255), ID заявки
- **type** - "BUY", тип ордера
- **status** - OrderStatus, Статус ордера
- **fiatPaymentLink** - string, ссылка на 3ds в случае интеграции с картами
- **creationDate** - string(255), дата создания заявки
- **modificationDate** - string(255), дата изменения заявки
- **expiresAtDate** - string(255), дата истечения заявки
---

### Sell request - OffRamp

#### POST /api/v2/exchange/merchant/sell
> Запрос на создание заявки для продажи криптовалюты в фиат. **OffRamp** операция.
#### params:
- **clientId** - string(255), registered client id
- **quoteId** - string(255), ID условий сделки
- **sourceAddress** - **Optional** string(255), крипто-адрес, с которого придёт криптовалюта, чтобы мы заранее могли проверить чистоту кошелька
 
#### response:
- **id** - string(255), ID заявки
- **type** - "SELL", тип ордера
- **status** - OrderStatus, Статус ордера
- **depositCryptoAddress** - string, крипто-адрес на который клиент должен прислать криптовалюту для обмена
- **creationDate** - string(255), дата создания заявки
- **modificationDate** - string(255), дата изменения заявки
- **expiresAtDate** - string(255), дата истечения заявки
---

### Common interfaces

```typescript
interface FiatAsset {
    id: CurrencyId;
    code: CurrencyCode;
}

interface CryptoAsset {
    id: CurrencyId;
    code: CurrencyCode;
    network: CurrencyNetwork;
    protocol?: CryptoProtocol;  // присутствует для USDT для опеределения сети
}

interface LimitAsset {
    code: CurrencyCode;
    network?: CurrencyNetwork;  // должно присутствовать для криптовалюты
}

interface QuoteAsset {
    // в запросе передавать в той части из которой надо рассчитать, в ответе придет рассчитанная в обе стороны
    amount?: double;
    code: CurrencyCode;
    network?: CurrencyNetwork;  // должно присутствовать для криптовалюты
}

interface QuoteFee {
    total: double;      // комиссия WhiteBird
    network: double;    // комиссия Crypto за проведение транзакции
    service: null;      // ???
    asset: CurrencyCode;// Какая фиатная валюта комиссии
}

enum CurrencyId {
    BYN = "BYN",
    RUB = "RUB",
    USD = "USD",
    EUR = "EUR",
    BTC = "BTC",
    ETH = "ETH",
    USDT_ERC = "USDT_ERC",
    TON = "TON",
    USDT_TON = "USDT_TON",
    TRX = "TRX",
    USDT_TRC = "USDT_TRC",
}

enum CurrencyCode {
    BYN = "BYN",
    RUB = "RUB",
    USD = "USD",
    EUR = "EUR",
    BTC = "BTC",
    ETH = "ETH",
    USDT = "USDT",
    TON = "TON",
    TRX = "TRX",
}

enum CurrencyNetwork {
    Bitcoin = "Bitcoin",
    Ethereum = "Ethereum",
    Ton = "Ton",
    Tron = "Tron",
}

enum CryptoProtocol {
    Erc = "ERC-20",
    Ton = "TON",
    Trc = "TRC-20",
}

enum OrderStatus {
    NEW = "NEW",    // заявка создана
}
```

#### PaymentProvider interfaces.
```typescript
interface PaymentProvider { // способ фиатного платежа
    id: PaymentProviderId;  // Id                      
    name: string;
    commissions: PaymentProviderCommission[];   // описание комиссий
    config: PaymentProviderConfig;              // описание доступных направлений/стран/тип карт
}

interface PaymentProviderConfig {
    paymentSystems: PaymentSystem[];
}

interface PaymentProviderCommission {
    buyCommission: string;  // percentage
    sellCommision: string;  // percentage
    bank?: string;  // опциональный, если комисси зависят от банка эмитента
}

interface PaymentSystem {
    paymentSystem: PaymentSystemType;   // тип платежного средства
    type: "PSP" | "CI" | "CA";          // PSP для карт, остальное для сторонних интеграций
    directions: PaymentDirection[];     // доступные направления
}

interface PaymentDirection {
    direction: "BUY" | "SELL";
    currencies: PaymentCurrency[];      // доступные валюты + банки + страны
}

interface PaymentCurrency {
    currency: CurrencyCode;
    countries?: string[];
    banks?: string[];
}

enum PaymentProviderId {
    ASSIST = "ASSIST",
    ALFA = "ALFA",
    MTS = "MTS",
    CA = "CA",
    STATUSBANK = "STATUSBANK",
    // или кастомные
}

enum PaymentSystemType {
    MIR = "MIR",
    VISA = "VISA",
    BelCard = "BelCard",
    MasterCard = "MasterCard",
}

```