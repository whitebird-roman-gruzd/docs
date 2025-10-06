# WhiteBird Registration API guide

## Назначение

Для интеграции с партнёрами, у которых большая пользовательская база с полным объёмом информации о документах удостоверяющих личность(ДУЛ) пользователей.

##### Описание
Для совершения транзакций на нашей платформе, нам необходимо собирать информацию о ДУЛ пользователей. Если у партнёра есть необходимая нам информация о пользователях, то мы можем переложить задачу сбора данных с пользователей на партнёра. Закрепим данное взаимодействие договором и сделаем партнёра "агентом по идентицикации пользователей" для нас.

В таком случае запрос на регистрацию пользователей в нашей системе является протоколом получения данных о пользователе от "агента по идентификации".

Авторизация запросов происходит через **x-api-key** header. все запросы Backend to Backend

### Разделы:
- **[Регистрация](#Register-POST-request)**
- **[Статус клиента](#Client-status)**
- **[Крипто-тест(для РБ пользователей)](#Crypto-test-requests)**
- **[Получение пользовательских токенов(для SDK)](#Generate-tokens-request)**

### Register POST request

Протокол получения полного набора данных о пользователе, создание аккаунта.

Если в документе удостоверяющем личность нет информации о месте регистрации эту информацию нужно получить из других документах подтверждающих регистрацию пользователя(оплата коммунальных, выписка из банка итп), также допустимо получить эту информацию со слов пользователя.

#### POST /api/v2/kyc/merchant/client/register

#### params:
- **email** - string(255)
- **phone** - string(100), телефон - опциональный параметр
- **gender** - Gender, пол
- **firstNameRu** - string(255), Имя на русском, если имеется в ДУЛ
- **lastNameRu** - string(255), Фамилия на русском, если имеется в ДУЛ
- **patronymicRu** - string(255), Отчество на русском, если имеется в ДУЛ
- **firstName** - string(255), Имя латиницей, если имеется в ДУЛ
- **lastName** - string(255), Фамилия латиницей, если имеется в ДУЛ


- **placeOfBirth** - string(unlimit), место рождения
- **birthDate** - "YYYY-MM-DD", дата рождения
- **nationality** - CountryCode, национальность по рождению


- **residence** - CountryCode, какой страны ДУЛ предоставлен
- **identityDocType** - DocType, тип ДУЛ из списка
- **identityDocIssueDate** - "YYYY-MM-DD", дата выдачи ДУЛ
- **identityDocExpireDate** - "YYYY-MM-DD", срок действия ДУЛ
- **identityDocNumber** - string(50), серия и номер ДУЛ
- **identityDocIssuer** - string(unlimit), орган выдавший ДУЛ
- **personalNumber** - string(50), идентификационный номер из ДУЛ

>данные о месте регистрации могут отличаться в зависимости от страны, заполнить нужно те поля, которые соответствуют формату адреса в стране регистрации пользователя
- **registrationCountry** - CountryCode, страна регистрации
- **registrationRegion** - string(unlimit), область регистрации
- **residenceDistrict** - string(unlimit), район регистрации
- **registrationCity** - string(unlimit), город регистрации, обязательно с наименованием, "город Минск" или "п. Колядичи" пробелы обязательны!
- **registrationStreet** - string(unlimit), улица регистрации - в формате "улица Радужная" или "ул. Радужная" пробелы обязательны!
- **registrationHouseAndFlat** - string(100), дом, корпус и квартира регистрации - "д. 25, к. 1, кв. 22", допустимо "д. 25/1, кв. 22", пробелы обязательны!
- **postCode** - string(20), почтовый код места регистрации


- **notUSTaxPayer** - bool, не является налогоплательщиком США
- **agreedWithOffer** - bool, согласие пользователя с офертой WhiteBird
- **exchangeInPersonalInterests** - bool, подтверждение обмена в личных интересах

### !! Все данные обязательны !!

#### Response:
- **clientId** - string(255)

Request examples:

``` json
// BY user example
{
   "email": "test.user.testov+15112024@ya.ru",
   "phone": "+375297778899",
   "gender": "муж",
   "firstNameRu": "Джон",
   "lastNameRu": "До",
   "patronymicRu": "Иванович",
   "firstName": "John",
   "lastName": "Doe",
   "placeOfBirth": "Республика Беларусь, г. Минск",
   "birthDate": "1994-01-05",
   "nationality": "112",
   "residence": "112",
   "identityDocType": "3",
   "identityDocIssueDate": "2020-01-02",
   "identityDocExpireDate": "2030-01-02",
   "identityDocNumber": "HB2129425",
   "identityDocIssuer": "Центравльный РУВД г. Минска",
   "personalNumber": "3029120H059PB9",
   "registrationCountry": "112",
   "registrationRegion": "Минская область",
   "residenceDistrict": "-",
   "registrationCity": "г. Минск",
   "registrationStreet": "ул. Криптоманов",
   "registrationHouseAndFlat": "д.30/1 кв.3",
   "postCode": "220000",
   "notUSTaxPayer": true,
   "agreedWithOffer": true,
   "exchangeInPersonalInterests": true
}

// RU user example
{
   "email":"test.user.testov+15112024@ya.ru",
   "phone": "-",
   "gender":"жен",
   "firstNameRu":"Джон",
   "lastNameRu":"До",
   "firstName": "-",
   "lastName": "-",
   "patronymicRu":"Иванович",
   "placeOfBirth":"РФ, Еврейская автономная область, г. Биробиджан",
   "birthDate":"1994-05-09",
   "nationality":"643",
   "residence":"643",
   "identityDocType":"9",
   "identityDocIssueDate":"2020-01-02",
   "identityDocExpireDate":"2030-01-02",
   "identityDocNumber":"9992129425",
   "identityDocIssuer":"УМВД России по Еврейской автономной области",
   "registrationCountry":"643",
   "registrationRegion":"Еврейская автономная область",
   "residenceDistrict":"Смидовичский район",
   "registrationCity":"п. Николаевка",
   "registrationStreet":"ул. Комсомольская",
   "registrationHouseAndFlat":"д. 23, кв. 30",
   "postCode": "-",
   "notUSTaxPayer": true,
   "agreedWithOffer": true,
   "exchangeInPersonalInterests": true
}
```

### Client status

Запрос для получения текущего статуса клиента, единственный возможный статус для совершения транзакций - VERIFIED

#### POST /api/v2/kyc/merchant/client/status

#### params:
- **clientId** - string(255)
#### response:
- **String** - ClientStatus

### Crypto test requests

Крипто-тест должны пройти все пользователи зарегистрированные с документами РБ. Требование регулятора, не можем тут подвинуться. При использовании SDK - тест встроен в SDK.
Тест простейший на 5 вопросов, получаем с помощью GET crypto-test, затем отправляем ответы пользователя в запрос POST crypto-test.

#### GET /api/v2/kyc/merchant/client/crypto-test?clientId=

#### response:
- **cryptoTestRequired** - bool, нужно ли пользователю пройти тест
- **questions** - TestQuestion[], сами вопросы с вариантами ответов

#### POST /api/v2/kyc/merchant/client/crypto-test

#### request:
- **clientId** - string(255), нужно ли пользователю пройти тест
- **answers** - object, “questionId”(string): answerId(int).

### Generate tokens request

Запрос для получения клиентских токенов, для передачи в SDK.
в On/Off ramp API не используется

#### POST /api/v2/auth/merchant/client/token/generate

#### params:
- **clientId** - string(255)
#### response:
- **accessToken** - string(unlimit)
- **refreshToken** - string(unlimit)

#### Типы данных

```typescript
enum Gender {
    Men = "муж",    // мужской пол
    Woman = "жен"   // женский пола
}

CountryCode = string // ISO код страны

enum DocType {
    PassportBY = "3",           // пасспорт РБ
    ResidencePermitBY = "6",    // вид на жительство РБ
    RefugeeCertificateBY = "7", // удостоверение беженца РБ
    ForeignPassport = "9",      // Пасспорт иностранца
    IDCardBY = "15",            // ID-карта РБ
    ForeignBiometricResidencePermitBY = "16", // биометрический вид на жительство РБ иностранного гражданина
    RefugeeBiometricResidencePermitBY = "17", // биометрический вид на жительство РБ лица без гражданства
    Other = "99"                // Иное
}

enum ClientStatus {
    CREATED = "CREATED",    // документы для AML не предоставлены
    PENDING = "PENDING",    // ожидает проверки AML
    VERIFIED = "VERIFIED",  // может совершать транзакции
    FROZEN = "FROZEN",      // окончательный статус (дубликат аккаунта)
    ARREST = "ARREST"       // окончательный статус после использования грязной крипты
}

interface TestQuestion {
    id: string;
    title: string;
    answers: TestAnswer[]
}

interface TestAnswer {
    id: number;
    title: string;
    correct: boolean;
}
```
