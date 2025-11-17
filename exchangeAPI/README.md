# WhiteBird API guide

## Process description

![ExchangeFlow](exchangeApiFlow.drawio.svg)

## Content:
- **[Assets](#assets-request)**
- **[Providers](#providers-request)**
- **[Limits](#Limit-request)**
- **[Quote](#Quote-request)**
- **[Buy](#buy-request---onramp)**
- **[Sell](#sell-request---offramp)**
- **[Order Status](#order-status-request)**

### Assets request

#### POST /api/v2/exchange/merchant/assets
> Доступные для обмена валюты.
#### params:
- **clientId** - *optional*, string(255), registered client id

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
- **clientId** - *optional*, string(255), registered client id
- **paymentMethod** - *optional*, PaymentProviderId, какой paymentProvider используем
- **paymentMethodToken** - *optional*, string(255), токен карты или любой GUID(для кастомных интеграций)
- **fromAsset** - QuoteAsset, из какой валюты обмен
- **toAsset** - QuoteAsset, в какую валюту обмен
- **destinationCryptoAddress** - string, **опциональный** параметр для **OnRamp**, позволяет точнее рассчитать комиссию сети, криптоадрес на который мы должны перевести крипту(клиент подтверждает, что кошелёк принадлежит ему)
- **comment** - string, опциональный **параметр** для **OnRamp** операции на TON.

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
    },
    "destinationCryptoAddress": "0QD1LvlKFvAT2fHxwm2uCxZ9X5EmIWbGLhGMwDDwcQXauCkU",
    "comment": "12331"
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

#### GET /api/v2/exchange/merchant/buy
> Запрос на создание заявки для покупки криптовалюты за фиат. **OnRamp** операция.
#### params:
- **quoteId** - string(255), ID условий сделки

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

### Order status request

#### GET /api/v2/exchange/merchant/order
> Запрос для получения статуса заявки **OnRamp**/**OffRamp**.
#### params:
- **orderId** - string(255), order ID

#### response
- **id** - string(255), ID заявки
- **number** - int, номер заявки
- **status** - OrderStatus, статус текущего ордера
- **exchangeOperation** - ExchangeOperation, условия текущей заявки
- **cryptoTransaction** - CryptoTransaction, статус крипто транзакции
- **fiatTransaction** - FiatTransaction, статус фиатной транзакции
- **client** - OrderClient, client для текущего ордера
- **creationDate** - string, UTC date string
- **modificationDate** - string, UTC date string
- **expiresAtDate** - string, UTC date string
- **completionDate** - string, UTC date string
- **serverDate** - string, UTC date string, внутренняя информация
- **exchangeType** - "BUY" | "SELL", тип обменной операции
- **operationType** - "CRYPTO_TO_FIAT" | "FIAT_TO_CRYPTO", тип операции
- **orderType** - "DEFAULT", ???
- **resultMessage** - string?, сервисная информация по ордеру
- **submitByResident** - bool, операция произведена резидентом РБ
- **promoCodeDetails** - string, информация об использовании промокода для транзакции

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
    USDC_ERC = "USDC_ERC",
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
    USDC = "USDC",
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
    NEW = "NEW",                // заявка создана
    PROCESSING = "PROCESSING",  // в обработке
    EXPIRED = "EXPIRED",        // когда ордер не выполнился за отведенное ему время(например клиент не отправил на крипту)
    COMPLETED = "COMPLETED",    // Ордер завершен успешно
    ERROR = "ERROR"             // ошибка выполнения ордера (для получения дополнительной информации нужно обратиться к поддержке WhiteBird)
}

interface ExchangeOperation {
    inputAsset: number;     // количество продаваемого актива
    outputAsset: number;    // количество приобритаемого актива
    exchangeFeeAssetInFiat: number; // сумма комиссий сервиса в фиате
    bonusOutputAsset: number;   // ???
    plainRatio: number;     // курс обмена до учёта комиссий
    ratio: number;          // курс обмена после учёта комиссий
    currencyPair: {
        fromCurrency: CurrencyCode; // продаваемая валюта
        toCurrency: CurrencyCode;   // приобретаемая валюта
    }
}

interface СryptoTransaction {
    transactionHash?: string;   // хэш транзакции в blockchain
    externalCryptoAddress?: string, // криптоадрес клиента на/с которого ушла/пришла крипта
    internalCryptoAddress: string;  // криптоадрес WhiteBird который участвовал в транзакции
    fromAddress?: string,       // криптоадрес с которого списывалась крипта
    toAddress?: string,         // криптоадрес на который зачислялась крипта
    status: СryptoTransactionStatus,
    currency: CurrencyCode;     // валюта транзакции
    comment?: string;           // для сети TON
    fee?: number;               // комиссия сети блокчейн
    feeNative?: null;           // ???
    feePaymentEnabledByClient: boolean; // платит ли комиссию крипты клиент 
    type: СryptoTransactionType;    // Тип проведения крипто операции
}

interface FiatTransaction {
    status: FiatTransactionStatus,    // статус фиатной транзакции
    paymentToken: string;             // пользовательский карт-токен/счёт/уникальный идентификатор счета проведения транзакции
    internalToken?: string;           // служебная информация
    orderIdentity: string;            // внутренний id ордера
    link?: string;                    // ссылка на 3ds страницу
    providerType: PaymentProviderId;  // текущий платёжный провайдер
    paymentType: string,              // ??? P2P
    processingBank?: string;          // банк проведения платежа(если такой есть)
    resultMessage?: string;           // причина ошибки платежа
    currency: CurrencyCode;           // валюта транзакции
    post?: string,                    // маска карты
    paymentSystem?: PaymentSystemType,// для карточных интеграций
}

interface OrderClient {
    clientId: string;
}

enum СryptoTransactionStatus {
    New = "NEW",                      // крипто тразакция создана
    NotFound = "NOT_FOUND",           // транзакции в сети блокчейн нет
    Rejected = "REJECTED",            // клиент отменил ордер
    Timeout = "TIMEOUT",              // за отведенное время не поступила крипта
    InvaldAmount = "INVALID_AMOUNT",  // поученное количество крипты отличается от количества указанного в ордере
    Error = "ERROR",                  // ошибка при выполнении ордера
    AmlError = "AML_ERROR",           // ошибка при проверке крипто адреса
    AmlBlocked = "AML_BLOCKED",       // амл заблокировал ордер 
    Arrest = "ARREST",                // крпто адрес не прошел проверку амл 
    Submitting = "SUBMITTING",        // транзакция создается в сети блокчейн
    Submitted = "SUBMITTED",          // транзакция создана в сети блокчейн
    Pending = "PENDING",              // транзакция запущена в сети блокчейн
    Selected = "SELECTED",            // крипто транзакция найдена в сети блокчейн
    Confirmed = "CONFIRMED",          // крипто транзакция подтверждена
}

enum FiatTransactionStatus {
    New = "NEW",                      // фиатная тразакция создана
    Rejected = "REJECTED",            // клиент отменил ордер
    Timeout = "TIMEOUT",              // за отведенное время не поступил фиат
    InvaldAmount = "INVALID_AMOUNT",  // поученное количество фиата отличается от количества указанного в ордере
    Error = "ERROR",                  // ошибка при выполнении ордера
    Declined = "DECLINED",            // ошибка при выполнении фиатной транзакции 
    AmlBlocked = "AML_BLOCKED",       // амл заблокировал ордер 
    Pending = "PENDING",              // фиатная транзакция запущена
    Processing = "PROCESSING",        // фиатная транзакция выполняется 
    Approved = "APPROVED",            // фиатная транзакция подтверждена
}

enum СryptoTransactionType {
    Auto = "AUTO",     // Транзакция прошла в автоматическом режиме
    Manual = "MANUAL"  // Транзакция прошла в ручном режиме
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
    VTB = "VTB",
    // или кастомные
}

enum PaymentSystemType {
    MIR = "MIR",
    VISA = "VISA",
    BelCard = "BelCard",
    MasterCard = "MasterCard",
}

```
