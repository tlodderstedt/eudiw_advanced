The idea of this design is to leverage the protocol mechanics of OpenID4VP and the trust model of the EUDIW to allow RPs to request a payment through a EUDIW.

The EUDIW acts as an OAuth Authorization Server and issues a sender-constrained access token, the RP will use to initate a payment with a financial institution.  

The new messages (or rather the message parameters) can be combined with OpenID4VP message parameter, i.e. signature creation could be combined with credential presentation.

![Alt text](https://github.com/tlodderstedt/eudiw_advanced/blob/main/out/payment/payment/payment.png "Payment Initiation with the EUDIW")

1. (steps 1-4) The TPP (acting as an OAuth client) sends an authorization request using RAR requesting authorization to request a payment initiation. Technically, this means a redirect from TPP to wallet and the wallet fetching the request object from the request URI. This also allows to do a proof of possession for a DPoP key. 
2. (step 8) The wallet asks for consents and pushes the access token (containing the EAA presentation) to the TPP using the TPP's response  endpoint.
3. (steps 9-10) Redirect from wallet back to TPP
4. (step 11) The TPP sends a payment initiation request to the ASPSP (including a DPoP PoP). 
5. ... the rest is payment initiation processing

The request (step 4) would contain an authorization_details parameter with the following content: 

```json
{
   "type": "payment_initiation",
   "instructedAmount": {
      "currency": "EUR",
      "amount": "123.50"
   },
   "creditorName": "Merchant A",
   "creditorAccount": {
      "bic":"ABCIDEFFXXX",
      "iban": "DE02100100109307118603"
   },
   "remittanceInformationUnstructured": "Ref Number Merchant"
}
```

It would also need to contain a proof of possession of a DPoP key, like this: 

```json
{
  "typ":"dpop+jwt",
  "alg":"ES256",
  "jwk": {
    "kty":"EC",
    "x":"l8tFrhx-34tV3hRICRDY9zCkDlpBhF42UQUfWVAWBFs",
    "y":"9VE4jf_Ok_o64zbTTlcuNJajHmt6v9TDVrU0CdvGRDA",
    "crv":"P-256"
  }
}
.
{
  "iat":1562262616,
  "nonce": "eyJ7S_zG.eyJH0-Z.HX4w-7v"
}
```

The wallet would need to know that such a request needs to be answered with an access token containing the presentation of an EAA of a certain type, lets assume a `vct` of "https://credentials.example.com/a2pay". The respective presentation would be added to the access token (along with the DPoP key in the `cnf` claim). Here is an example: 

```json
{
    "iss": "https://issuer.eudiw.com",
    "aud": "https://bank.example.com/api",
    "exp": 1883000000,
    "nbf": 1718198433,
    "iat": 1718198433,
    "a2pay_presentation": "eyJhbGciOiAiRVMy..V0~Wy..ZnBAan5g",
    "cnf": {
      "jwk": {
        "kty":"EC",
        "x":"l8tFrhx-34tV3hRICRDY9zCkDlpBhF42UQUfWVAWBFs",
        "y":"9VE4jf_Ok_o64zbTTlcuNJajHmt6v9TDVrU0CdvGRDA",
        "crv":"P-256"
      }
    }
}
```

The TPP would need to send this access token along with the DPoP proof in step 11.
