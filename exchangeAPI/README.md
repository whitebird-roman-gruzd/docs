# Whitebird API guide

## Process description

![ExchangeFlow](exchangeApiFlow.drawio.svg)

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
- **paymentMethod** - LimitAsset, в какую валюту обмен
- **fromAsset** - LimitAsset, из какой валюты обмен
- **toAsset** - LimitAsset, в какую валюту обмен

#### response:
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