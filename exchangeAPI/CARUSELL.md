## Instruction for *CARUSELL* payment provider

This is a payment provider for onramp/offramp in RUB.

### Content:
- **[OnRamp](#fiat-to-crypto-on-ramp)**
- **[OffRamp](#crypto-to-fiat-off-ramp)**

#### FIAT-TO-CRYPTO (On-Ramp)
1) Check whether the payment method is available for the client

POST api/v2/exchange/merchant/payment/method

Request Header: x-api-key
Request Body:
```
{
    "fiatAsset": "RUB",
    "orderType": "SELL",
    "direction": "SDK"
}
```
Response:
```
{
    "providerId": "CARUSELL",
    "providerType": "CARUSELL",
    "status": "ENABLED",
    "name": "CARUSELL"
}
```
2) Create a quote

POST api/v2/exchange/merchant/quote

Request Header: x-api-key
Request Body:
```
{
   "clientId": "76b758b0-5603-4419-9c35-c01e5f775269",
   "fromAsset": {
        "code": "RUB",
        "network": null,
        "amount": "2000"
   },
   "toAsset": {
      "code": "TRX",
      "network": "Tron"
   },
   "paymentMethod": "CARUSELL",
   "paymentMethodToken": "generateUUID()"
}
```

3) Create an order specifying returnUrl and failUrl to ensure the client is redirected back to the website after payment

POST
/api/v2/exchange/merchant/buy?quoteId=27690c04-e344-4d31-81ae-c58296ba6ddc&returnUrl=https://www.google.com/&failUrl=https://www.google.com/

Request Header: x-api-key

Response:
```
{
    "id": "53ab15af-31c9-4c8d-8ac8-55f9e353c09e",
    "type": "BUY",
    "status": "PROCESSING",
    "fiatPaymentLink": "https://sandbox.carusell.world/processing/api/v1/test/checkout/payment?payment-id=284949544680468480&status=CAPTURED&settlement-account-id=SA241575994854899712&signature=8f5a61dcdab280b21eb552e510f5c77a5b42f1da0e1c2d0896834328a5e32ad9",
    "createDate": "2026-02-26T11:45:38+0000",
    "modificationDate": "2026-02-26T11:45:38+0000"
}
```


#### CRYPTO-TO-FIAT (Off-Ramp)
1) Check whether the payment method is available for the client

POST api/v2/exchange/merchant/payment/method

Request Header: x-api-key
Request Body:
```
{
    "fiatAsset": "RUB",
    "orderType": "SELL",
    "direction": "SDK"
}
```
Response:
```
{
    "providerId": "CARUSELL",
    "providerType": "CARUSELL",
    "status": "ENABLED",
    "name": "CARUSELL"
}
```
2) Create a quote

POST api/v2/exchange/merchant/quote

Request Header: x-api-key
Request Body:
```
{
   "clientId": "76b758b0-5603-4419-9c35-c01e5f775269",
   "fromAsset": {
        "code": "TRX",
        "network": "Tron"
   },
   "toAsset": {
        "code": "RUB",
        "network": null,
        "amount": "2000"
   },
   "paymentMethod": "CARUSELL",
   "paymentMethodToken": "generateUUID()"
}
```

3) Get the list of available banks for transfers via **CARUSELL**

The client selects the bank to which the fiat funds will be credited.

GET
https://carusell-service.dev.wbdevel.net/api/v1/bank

Response:
```
{
    "id": "100000000118",
    "description": "AGROPROMKREDIT",
    "active": true
},
{
    "id": "200000000006",
    "description": "AK BARS BANK",
    "active": true
}
...
```
4) Create an order with the bankIdentifier selected by the client

POST
/api/v2/exchange/merchant/sell?quoteId=27690c04-e344-4d31-81ae-c58296ba6ddc&bankIdentifier=100000000118

Request Header: x-api-key

Response:
```
{
    "id": "53ab15af-31c9-4c8d-8ac8-55f9e353c09e",
    "type": "SELL",
    "status": "PROCESSING",
    "depositCryptoAddress": "TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
    "createDate": "2026-02-26T11:45:38+0000",
    "modificationDate": "2026-02-26T11:45:38+0000"
}
```
For the test environment, it does not matter which bank is selected (bankIdentifier) — the payment will be processed successfully in any case.

**CARUSELL** also has an anti-fraud check.
It is performed automatically:

At the moment of order creation for the crypto-to-fiat flow

At the moment of order execution for the fiat-to-crypto flow

If an order fails to be created or returns an error at any stage, there is a possibility that the anti-fraud system has declined the transaction.