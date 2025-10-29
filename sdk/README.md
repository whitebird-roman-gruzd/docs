# SDK Documentation

## Введение
Добро пожаловать в документацию WhiteBird SDK!

Этот SDK предоставляет возможность встроить функционал **атомарного обмена WhiteBird** прямо к себе в мобильное приложение(Swift/Kotlin SDK) или на сайт(JS SDK).
Авторизация в SDK работает по упрощенному механизму, используя OTP для входа на платформу

SDK предоставляет возможность взаимодействия с WhiteBird в трёх режимах. Режимы работы:
- LoginMode
- AuthMode
- TokensMode

#### Презентация
- Ознакомиться c визуальным флоу можно в [Figma](https://www.figma.com/design/QhTl1W0BEncjvGXRu03UiW/SDK-flow?node-id=0-1&p=f)
- Демо доступно по ссылке: [SDK Demo](https://sdk.dev.wbdevel.net/v2.0/assets/sdk-demo/index.html)
(```MerchantId: 4f19017b-0793-4591-94ff-610bb3c4665b```)

Описание каждого режима доступно [ниже](#краткое-описание-режимов) в документации.

## Верхнеуровневое описание процессов

| Общий flow WhiteBird SDK | Процесс Регистрации |
|-|-|
| ![UserFlow](UserFlow.drawio.svg) | ![Registration](Registration.drawio.svg) |

| Процесс Верификации | Процесс Авторизации |
|-|-|
| ![Verification](verification.drawio.svg) | ![Authz](Authorization.drawio.svg) |

## Краткое описание режимов

Сразу добавим лексику:
- Клиент - компания, которая интегрирует SDK
- Пользователь - конечный пользователь продукта

### LoginMode
Этот режим предназначен для аутентификации *(авторизации/регистрации/верификации)* пользователей через WhiteBird. Он позволяет пользователям войти в систему с использованием учетных данных WhiteBird и получить возможность совершать операции обмена, просматривать историю личных операций, обратиться в support. Подходит для сценариев, когда нужно добавить возможность пополнения/списания средств на/со своей платформы или криптовалюты в фиат и обратно.

### AuthMode
SDK в данном режиме используется для аутентификации *(авторизации/регистрации/верификации)* пользователей через WhiteBird и позволяет получить авторизационные токены пользователя, которые можно использовать для взаимодействия с API WhiteBird. В этом случае клиент для нас не является *агентом по идентификации пользователей*, и добавляет свой custom UI атомарного обмена, который позволяет совершать операции через наш API, используя авторизационные токены клиента.

### TokensMode
Режим предназначен для запуска нашего SDK с токенами доступа. Полностью включает в себя все возможности как в LoginMode, но позволяет убрать авторизацию WhiteBird.
Данный режим подразумевает бесшовный переход из приложения клиента на платформу WhiteBird и обратно. Также в данном режиме подразумевается, что клиент будет для нас *агентом по идентификации пользователей*.

## Агент по идентификации пользователей

Возможность, которую это открывает - использование SDK в TokensMode. Пользователю не понадобится ещё раз авторизовываться на платформе WhiteBird самостоятельно, это за него сделает клиент, получив Auth tokens через **backend to backend** взаимодействие по REST API, c помощью API Key мерчанта.

- Запрос на регистрацию пользователей: [register](../onboardingAPI/README.md#register-post-request)
- Запрос на получение токенов пользователя: [generate](../onboardingAPI/README.md#generate-tokens-request)

## Добавление на сайт

### Подключение через CDN
```html
<script src="https://sdk.dev.wbdevel.net/v2.0/integration/wbExchangeSdk-v001.js"></script>
```

### Контейнер
- Контент SDK расчитан на мобильную версию - контейнер не рекомендуется делать менее 360 пикселей
- Контейнер для SDK может находиться в любом месте сайта
- Далее в конфиг передаем html-element этого контейнера
- !! SDK не изменяет стилей контейнера, все стили контейнера настраиваются приложением
- iframe с SDK занимает все доступное место в контейнере

### API Reference
Пример инициализации SDK в ```LoginMode```:
```javascript
wbExchangeSdk.setup({
  // контейнер для встраивания SDK
  el: document.getElementById("wbExchangeSdkWrapper"),
  // id клиента
  merchantId: 'xxxx',
  mode: wbExchangeSdk.mode.LoginMode,
})
```

Пример инициализации SDK в ```TokensMode```:
```javascript
wbExchangeSdk.setup({
  // контейнер для встраивания SDK
  el: document.getElementById("wbExchangeSdkWrapper"),
  // id клиента
  merchantId: 'xxxx',
  mode: wbExchangeSdk.mode.TokensMode,

  accessToken: '*****',
  refreshToken: '*****',
})
```

Пример инициализации SDK в ```AuthMode```:
```javascript
wbExchangeSdk.setup({
  // контейнер для встраивания SDK
  el: document.getElementById("wbExchangeSdkWrapper"),
  // id клиента
  merchantId: 'xxxx',
  mode: wbExchangeSdk.mode.AuthMode,

  // callback возвращает токены пользователей для взаимодействия с whitebird через API
  // не вернет токены, в случае если пользователь ещё не верифицирован.
  onLogin: ({accessToken, refreshToken, isUserVerified}) => {},
})
```

### Опциональные парамертры конфигураци SDK.

**externalClientId** - string, должен быть передан мерчантом для связки пользователей WhiteBird с пользователями мерчантов.

**email** - string, позволяет предзаполнить поле email, чтобы сократить количество действий пользователя при авторизации на WhiteBird.

**merchantPass** - string, необходимый параметр, чтобы сессия пользователя не завершалась через 15 минут, а токен мог обновляться.

**currencyAmount** - int, позволяет предзаполнить количество валюты, из которой совершается обмен

**currencyFrom** - строки из _Currency enum_. позволяет предзаполнить поле валюты, из которой будет совершаться обмен.
**disableCurrencyFrom** - bool, позволяет отключить возможность редактирования валюты From, а также блокирует поле количества если оно было передано.

**currencyTo** - строки из _Currency enum_. позволяет предзаполнить поле валюты, на которую будет совершаться обмен.
**disableCurrencyTo** - bool, позволяет отключить возможность редактирования валюты To, а также блокирует поле криптокошелька если оно было передано.
```typescript
let currencyFrom: Currency;
let currencyTo: Currency;

enum Currency {
    BYN,
    RUB,
    USD,
    EUR,
    BTC,
    ETH,
    USDT, // ERC-20
    USDC, // ERC-20
    TRX,
    USDT_TRC, // TRC-20
    TON,
    USDT_TON, // TON
    WBP, // TRC-20
}
```
**cryptoWallet** - string, позволяет предзаполнить поле кошелёк для пользователя.

**showBackButtonOnHomePage** - bool, позволяет показать кнопку "back" на нашем UI и реагировать, с помощью callback onExit, на нажатие кнопки пользователем.

**onExit** - () -> void, callback для исполнения действий по нажатию на "back"
**onOrderCreated** - ({orderId, internalCryptoAddress}) -> void, callback для уведомления мерчанта о создании ордера.


Желательно после окончания работы с SDK вызвать ```wbExchangeSdk.cleanup()``` 


