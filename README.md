# Complete Flow for ProfitPay VCC Functionality

The following describes the entire flow from the perspective of a registered ProfitPay user when creating and funding a VCC:

- Step Zero - User downloads app, succesfully registers, and passes KYC check
- [Step One](#step-one) - User receives a VCC with zero balance
- [Step Two](#step-two) - We create a test bank account -to simulate the user's own real bank account- for the user with test funds so the VCC can be funded
- [Step Three](#step-three) - The user moves funds via ACH from their test bank account to their previously empty VCC

- cash back balance total - daily accumulated for current month
- cash back balance daily - total accumualted daily
- cancel VCC - go balance go back to the user's original funding instrument

## Step One

The user receives a VCC with zero balance on card activation (after proper KYC flow).

**Endpoint required**

`POST api/prototype/vcards` - Please note this is the proposed endpoint for creating a VCC with funding in one single call

request body

```json
{
    "vcard": {
        "type": "string",
        "provider_id": "string",
        "first_name": "string",
        "last_name": "string",
        "street_address1": "string",
        "street_address2": "string",
        "zip": "string",
        "city": "string",
        "state": "string",
        "country": "string",
        "email": "string",
        "phone": "string",
        "amount": 0
    },
    "funding": [],
    "meta": [
        {
            "balance": "empty initial card"
        }
    ]
}
```

201 - response

```json
{
    "vcard": {
        "id": "rxg6dmu8-gwwv-3v2o-mw9x-ec1jfz1tt36b",
        "type": "vcc",
        "provider_id": "svvo9n2w-egic-t9hr-2m9g-xolwxbubxeq6",
        "merchant_id": "1xkavdlc-u4d8-95dc-rnhr-jq95n0q1kebu",
        "external_id": "yuupc1sm-gumq-qmsi-00gm-e3l834kcgzrr",
        "first_name": "string",
        "last_name": "string",
        "street_address1": "string",
        "street_address2": "string",
        "zip": "string",
        "city": "string",
        "state": "string",
        "country": "string",
        "email": "string",
        "phone": "string",
        "card_number": "448514******6955",
        "card_expiry_month": "03",
        "card_expiry_year": "2022",
        "card_brand": "visa",
        "token": "tok_mock_n4hzy1n79kej989iafij9b",
        "amount": 0,
        "status": "available",
        "available_amount": 0,
        "actions": [],
        "is_cancelable": false,
        "meta": [
            {
                "balance": "empty initial card"
            }
        ]
    },
    "created_at": "2020-08-06T22:34:36Z"
}
```

## Step Two

Now that the user has a VCC, they need to fund it with funds from their bank account. So the user will add a bank account to their app. This means we need the ability to create *test* bank accounts for our users, which are visible from the VCC API to determine balances and funds availability, meaning if a user tries to fund a VCC with a larger balance than their test bank account, VCC API should fail.

**Endpoint required**

- `POST api/prototype/bankAccount`

Body

```json
{
  "firstName": "Max",
  "lastName": "Planck",
  "email": "max@quantum.com",
  "account": 11111111111111,
  "routing": 111111111,
  "balance": 10000
}
```

201 - Response

```json
{
  "firstName": "Max",
  "lastName": "Planck",
  "email": "max@quantum.com",
  "account": 11111111111111,
  "routing": 111111111,
  "balance": 10000
}
```
Additionally we will require the following endpoints for testing:

Get a Bank Account by account number

- `GET api/prototype/bankAccount?account=11111111111111`

200 - Response

```json
{
  "firstName": "Max",
  "lastName": "Planck",
  "email": "max@quantum.com",
  "account": 11111111111111,
  "routing": 111111111,
  "balance": 10000
}
```

Add Funds to Bank Account

- `PATCH api/prototype/bankAccount?account=11111111111111`

Request body

```json
{
  "balance": 1000  
}
```

201 - Response

```json
{
  "firstName": "Max",
  "lastName": "Planck",
  "email": "max@quantum.com",
  "account": 11111111111111,
  "routing": 111111111,
  "balance": 11000
}
```

## Step Three

Now that the user has a VCC and a bank account, the user can now fund their VCC via ACH from their bank account.

**Endpoint required**

- `PATCH api/prototype/vcards/:cardId`

Request Body

```json
"funding": [
    {
        "method": "ACH",
        "token": "token_for_tokenized_test_bank_account",
        "amount": 10000
    }
 ]
```

201 - response

```json
{
    "vcard": {
        "id": "rxg6dmu8-gwwv-3v2o-mw9x-ec1jfz1tt36b",
        "type": "vcc",
        "provider_id": "svvo9n2w-egic-t9hr-2m9g-xolwxbubxeq6",
        "merchant_id": "1xkavdlc-u4d8-95dc-rnhr-jq95n0q1kebu",
        "external_id": "yuupc1sm-gumq-qmsi-00gm-e3l834kcgzrr",
        "first_name": "string",
        "last_name": "string",
        "street_address1": "string",
        "street_address2": "string",
        "zip": "string",
        "city": "string",
        "state": "string",
        "country": "string",
        "email": "string",
        "phone": "string",
        "card_number": "448514******6955",
        "card_expiry_month": "03",
        "card_expiry_year": "2022",
        "card_brand": "visa",
        "token": "tok_mock_n4hzy1n79kej989iafij9b",
        "amount": 10000,
        "status": "available",
        "available_amount": 10000,
        "actions": [],
        "is_cancelable": false,
        "meta": [
            {
                "balance": "empty initial card"
            }
        ]
    },
    "created_at": "2020-08-06T22:34:36Z"
}
```

- 404 when `cardId` is incorrect
- 402 for insufficient funds. Meaning the `amount` is greater than available balance in the `mock` bank account created previously

