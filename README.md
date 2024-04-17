# Account Recovery README
This API implements the [SEP-30](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0030.md) protocol. Enables an individual (e.g., a user or wallet) to regain access to a Stellar account that it owns after the individual has lost its private key without providing any third-party control of the account.

## Table of Contents
* [Implementation](https://github.com/bigger-tech/account-recovery-readme/main/README.md#how-to-implement-account-recovery-api)
   * [External authentication endpoints](https://github.com/bigger-tech/account-recovery-readme/main/README.md#external-authentication-endpoints)
   * [Errors](https://github.com/bigger-tech/account-recovery-readme/main/README.md#errors)
   * [SEP-30 endpoints](https://github.com/bigger-tech/account-recovery-readme/main/README.md#sep-30-endpoints)
   * [SEP-10 endpoints](https://github.com/bigger-tech/account-recovery-readme/main/README.md#sep-10-endpoints)

## How to implement Account Recovery API
Account Recovery API provides two custom endpoints to authenticate externally using the authentication methods defined by the server, one to request a verification code, and another to send it and receive a JWT in return.

It also provides the required endpoints in the [SEP-30](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0030.md) and the [SEP-10](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0010.md) documentation.

### External Authentication methods
Defined by the implementer, for now, the supported methods are:

Type | Description
-----|------------
`phone_number` | A phone number. When encoding a phone number the ITU-T recommendations [E.123](https://www.itu.int/rec/T-REC-E.123) and [E.164](https://www.itu.int/rec/T-REC-E.164) should be followed. The phone number should be formatted in international notation with a leading `+` as demonstrated in E.123 Section 2.5 to ensure phone numbers are consistently encoded and are globally unique. Spaces should be omitted.
`email` | An email address.


### External authentication Endpoints   

* [`POST /api/external-auth/verification/<address>`](https://github.com/bigger-tech/account-recovery-readme/edit/main/README.md#post-apiexternal-authverificationaddress)
* [`POST /api/external-auth/authentication/<address>`](https://github.com/bigger-tech/account-recovery-readme/edit/main/README.md#post-apiexternal-authauthenticationaddress)

####  `POST /api/external-auth/verification/<address>`

This endpoint requests a verification code for a registered account.

The user that wants to recover an account, should send the authorization method and value for the account previously registered in the [SEP-30](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0030.md#post-accountsaddress) endpoint [`POST /accounts/<address>`](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0030.md#post-accountsaddress). It is necessary to send only one of the `auth_methods` that has been registered.

###### Request

Request Parameters:

Name | Type  | Description
-----|------ |------------
`type` | string | Auth method type defined in [External authentication methods](https://github.com/bigger-tech/account-recovery-readme/edit/main/README.md#external-authentication-methods), one of the `auth_methods` previously registered in the [POST /accounts/<address>](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0030.md#post-accountsaddress) endpoint.
`value`| string | One of the auth values previously registered in the [POST /accounts/<address>](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0030.md#post-accountsaddress) endpoint, with which the user wants to receive the verification code.

Example:
```
POST https://server-url/api/external-auth/verification/GCQKGUIQ56DQYFY7S7HYBRX6JLOS56FXSDEWRGAEPDHF7BJR3HTXC24G
```
```json
{
    "type": "email",
    "value": "john-titor@test.com"
}
```

###### Response 
```200```

####  `POST /api/external-auth/authentication/<address>`

This endpoint sends to the server the verification code received when the user made a request in the previous endpoint. In response, if the request is successful, the user will get a JWT that he could use to access the required endpoints, authenticated as one of the identities that can control the registered account.

###### Request

Request Parameters:

Name | Type  | Description
-----|------ |------------
`type` | string | Auth method type defined in [External authentication methods](https://github.com/ScaleMote/account-recovery-readme/edit/main/README.md#external-authentication-methods), one of the auth values previously registered in the [POST /accounts/<address>](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0030.md#post-accountsaddress) endpoint.
`value`| string | One of the auth values previously registered in the [POST /accounts/<address>](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0030.md#post-accountsaddress) endpoint, with which the user wants to receive the verification code.
`verification_code` | string | Verification code received in the auth_method requested in the [POST api/external-auth/verification/<address>](https://github.com/bigger-tech/account-recovery-readme/edit/main/README.md#post-apiexternal-authverificationaddress)

Example:
```
POST https://server-url/api/external-auth/authentication/GCQKGUIQ56DQYFY7S7HYBRX6JLOS56FXSDEWRGAEPDHF7BJR3HTXC24G
```
```json
{
    "type": "email",
    "value": "john-titor@test.com",
    "verification_code": "98400"
}
```
###### Response 
```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVC..."
}
```

### Errors
If an error occurs when calling any endpoint, an appropriate HTTP status code will be returned along with an error response.

##### Status Code
Common HTTP status codes may be returned for a server. In particular the following are expected:

Status Code | Name | Reason
-----|------|------------
`400` | Bad Request | The request is invalid in any way.
`401` | Unauthorized | No `Authorization` header has been provided or the contents of the header are not accepted as valid.
`404` | Not Found | The validation code or the validation method is invalid.
`429` | Too many requests | There were too many attempts to validate the verification code.


### SEP-30 Endpoints
For further information, please refer to the [SEP-30](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0030) documentation.

* `POST /accounts/<address>` required `SEP-10` JWT
* `PUT /accounts/<address>` required `external-auth` JWT
* `POST /accounts/<address>/sign/<signing-address>` required `external-auth` JWT *
* `GET /accounts/<address>` required `SEP-10` JWT
* `DELETE /accounts/<address>` required `external-auth` JWT
* `GET /accounts` required `SEP-10` JWT


In the case of `POST /accounts/<address>/sign/<signing-address>`, we use the simple approach as it is detailed in the [SEP-30](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0030.md#post-accountsaddresssignsigning-address) documentation:

> The simplest approach is to require the transaction to only contain operations for the authenticated account. The server can verify that the source account of the transaction and the source account of all operations in the transaction match the account in the request [ ... ]   
> The transaction can contain any operations, but it is anticipated for the use cases discussed earlier in this protocol that the transaction will contain an operation to add a new signer to the account and possibly to remove an old signer or to merge an account. The signature will be generated using the signing key that the server generated during registration for the account.


### SEP-10 Endpoints
For further information, please refer to the [SEP-10](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0010.md) documentation.

* `GET <WEB_AUTH_ENDPOINT>`
* `POST <WEB_AUTH_ENDPOINT>`
