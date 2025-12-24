# Session flow

## Create Session

### **POST** `/api/v3/exchange/merchant/session`

Creates an  session for fiat ↔ crypto or crypto ↔ fiat operations and returns an SDK URL to complete the order.

---

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API key |

---

## Request Body

```json
{
"fromAsset":"BYN",
"fromAmount":"100",
"toAsset":"USDT_TRC",
"destinationCryptoAddress":"TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
"comment":"",
"externalClientId":"externalClientId",
"redirectUrl":""
}
```

### Request Parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `fromAsset` | string | ✅ | Source asset |
| `fromAmount` | string | ✅ | Amount to exchange |
| `toAsset` | string | ✅ | Target asset |
| `destinationCryptoAddress` | string | ✅ | Destination crypto wallet address |
| `comment` | string | ❌ | Optional comment for TON |
| `externalClientId` | string | ✅ | External client identifier |
| `redirectUrl` | string | ❌ | Redirect URL after completion |

---

## Supported Assets

### Fiat

- `BYN`
- `RUB`
- `USD`
- `EUR`

### Crypto

- `BTC`
- `ETH`
- `TRX`
- `TON`
- `USDT_ERC`
- `USDC_ERC`
- `USDT_TRC`
- `USDT_TON`

---

## Errors & Exceptions

### `/merchant/session`

| Error Code | Description |
| --- | --- |
| `INVALID_AMOUNTS` | Amount not specified or two asset amounts provided |
| `INVALID_DESTINATION_CRYPTO_ADDRESS` | Invalid destination crypto address |
| `DESTINATION_ADDRESS_IS_INTERNAL` | Destination address is internal |
| `NoSuchCurrencyException` | Requested currency is not supported |
| `DIRECTION_BLOCKED` | Exchange direction is blocked |
| `Unauthorized` | Invalid api key |

---

## Response

```json
{
    "sessionId": "14500600-0631-46c4-9ae1-fab9e4c798f8",
    "externalClientId": "external-client-id-2",
    "destinationCryptoAddress": "TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
    "comment": null,
    "exchange": {
        "fromAsset": "USD",
        "fromGrossAmount": "20",
        "fromNetAmount": "19.5",
        "toAsset": "TRX",
        "toGrossAmount": "68.783069",
        "toNetAmount": "68.520069",
        "actualRate": "0.2919",
        "exchangeRate": "0.2835",
        "fees": [
            {
                "type": "PERCENT",
                "amount": "-0.5",
                "asset": "USD"
            },
            {
                "type": "CRYPTO_FEE",
                "amount": "-0.263",
                "asset": "TRX"
            }
        ]
    },
    "limit": {
        "min": "11.67",
        "max": "13333.33"
    },
    "sdk": {
        "url": "https://sdk.dev.wbdevel.net/v2.0/?mode=LoginMode&merchantId=...&externalClientId=...&sessionId=..."
    }
}
```

---

## Webhooks

---

### **order.processing**

Triggered when a new order is created.

```json
{
	"id":"webhook-id",
	"type":"order.processing",
	"createdAt":"2024-05-23T08:44:58+0000",
	"sessionId":"3c0130a4-06f2-4d18-bf39-27153caff6f5",
	"orderId":"e57e77fc-802c-4c0d-8c7c-08806c159725"
}

```

---

### **order.completed**

Triggered when the order is successfully completed.

```json
{
	"id":"webhook-id",
	"type":"order.completed",
	"createdAt":"2024-05-23T08:44:58+0000",
	"sessionId":"3c0130a4-06f2-4d18-bf39-27153caff6f5",
	"orderId":"e57e77fc-802c-4c0d-8c7c-08806c159725"
}
```

---

### **order.expired**

Triggered when the order or session expires.

```json
{
	"id":"webhook-id",
	"type":"order.expired",
	"createdAt":"2024-05-23T08:44:58+0000",
	"sessionId":"3c0130a4-06f2-4d18-bf39-27153caff6f5",
	"orderId":"e57e77fc-802c-4c0d-8c7c-08806c159725"
}
```

---

### **order.error**

Triggered when the order fails.

```json
{
	"id":"webhook-id",
	"type":"order.error",
	"createdAt":"2024-05-23T08:44:58+0000",
	"sessionId":"3c0130a4-06f2-4d18-bf39-27153caff6f5",
	"orderId":"e57e77fc-802c-4c0d-8c7c-08806c159725"
}
```

---

## Get Current Session Order

### GET `/api/v3/exchange/merchant/oreder/current`

---

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API key |

---

## RequestParam

| Param | Required | Description |
| --- | --- | --- |
| `externalClientId` | ✅ Yes |  External client identifier  |

---

## Response

```json
        {
            "id": "5289b6f0-4945-4b74-b243-4fad013eed50",
            "number": 9210086,
            "rate": "TRX/BYN",
            "systemRateValue": "1.2",
            "exchangeRateValue": "1.2",
            "actualRateValue": "1.24409",
            "promoCode": null,
            "recalculationReason": "NONE",
            "clientId": "ad372b6f-9eb8-48fc-9278-51c7a3639aef",
            "status": "COMPLETED",
            "failureMessage": null,
            "completionDate": "2025-12-22T18:02:35+0000",
            "creationDate": "2025-12-22T17:59:18+0000",
            "input": {
                "type": "FIAT_PROVIDER",
                "asset": "BYN",
                "amount": "50",
                "transactionAmount": "50",
                "feeAmount": "1.45",
                "status": "COMPLETED",
                "failureMessage": null,
                "expirationDate": null,
                "provider": "CA",
                "paymentType": null,
                "processingBank": null,
                "fromToken": "only internal token should be set",
                "toToken": "3fe6f272-2608-4800-9859-56897a4261da",
                "link": "191e0716-5e84-4e92-8e62-23b04cec63cb"
            },
            "output": {
                "type": "CRYPTO_TRANSFER",
                "asset": "TRX",
                "amount": "40.190074",
                "transactionAmount": "40.190074",
                "feeAmount": "0",
                "status": "COMPLETED",
                "failureMessage": null,
                "expirationDate": null,
                "fromAddress": "TTUhGjn4LZKZC35Sk6T7e731dEE8HqBWwQ",
                "toAddress": "TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
                "comment": null,
                "hash": "b733486d285253130a61b2b2cf07dbd6d76528f56195e6937cdfbb68d42e4759"
            }
        }
```

---

## Get Current Session Orders By ExternalClientId

### POST `/api/v3/exchange/merchant/oreder/history`

---

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API keyRequest |

---

```json
{
    "externalClientId": "5289b6f0-4945-4b74-b243-4fad013eed50",
    "sessionIds": ["5289b6f0-4945-4b74-b243-4fad013eed50"]
}
```

---

## Response

```json
{
    "content": [
        {
            "id": "5289b6f0-4945-4b74-b243-4fad013eed50",
            "number": 9210086,
            "rate": "TRX/BYN",
            "systemRateValue": "1.2",
            "exchangeRateValue": "1.2",
            "actualRateValue": "1.24409",
            "promoCode": null,
            "recalculationReason": "NONE",
            "clientId": "ad372b6f-9eb8-48fc-9278-51c7a3639aef",
            "status": "COMPLETED",
            "failureMessage": null,
            "completionDate": "2025-12-22T18:02:35+0000",
            "creationDate": "2025-12-22T17:59:18+0000",
            "input": {
                "type": "FIAT_PROVIDER",
                "asset": "BYN",
                "amount": "50",
                "transactionAmount": "50",
                "feeAmount": "1.45",
                "status": "COMPLETED",
                "failureMessage": null,
                "expirationDate": null,
                "provider": "CA",
                "paymentType": null,
                "processingBank": null,
                "fromToken": "only internal token should be set",
                "toToken": "3fe6f272-2608-4800-9859-56897a4261da",
                "link": "191e0716-5e84-4e92-8e62-23b04cec63cb"
            },
            "output": {
                "type": "CRYPTO_TRANSFER",
                "asset": "TRX",
                "amount": "40.190074",
                "transactionAmount": "40.190074",
                "feeAmount": "0",
                "status": "COMPLETED",
                "failureMessage": null,
                "expirationDate": null,
                "fromAddress": "TTUhGjn4LZKZC35Sk6T7e731dEE8HqBWwQ",
                "toAddress": "TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
                "comment": null,
                "hash": "b733486d285253130a61b2b2cf07dbd6d76528f56195e6937cdfbb68d42e4759"
            }
        }
    ],
    "pageable": {
        "sort": {
            "unsorted": false,
            "sorted": true,
            "empty": false
        },
        "pageNumber": 0,
        "pageSize": 10,
        "offset": 0,
        "paged": true,
        "unpaged": false
    },
    "totalElements": 1,
    "totalPages": 1,
    "last": true,
    "numberOfElements": 1,
    "size": 10,
    "number": 0,
    "sort": {
        "unsorted": false,
        "sorted": true,
        "empty": false
    },
    "first": true,
    "empty": false
}
```

---