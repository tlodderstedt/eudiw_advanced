
![Alt text](https://github.com/tlodderstedt/eudiw_advanced/blob/main/out/payment/delegated_sca/delegated_sca/delegated_sca.png "Payment Initiation with the EUDIW")

1. (steps 1-4) The TPP (acting as OpenID4VP RP) sends a presentation request with transaction data for payment initiation. In addition to the normal flow, it also indicates that the resulting VC presentation shall be used with a third party (the API of the ASPSP/the bank). It also passes a DPoP key, to be used to proof possession when using the VC presentation (as access token) with the payment initation API of the ASPSP. 
2. (step 8) The wallet asks for consents and pushes the access token (containing the EAA presentation) to the TPP using the TPP's response  endpoint.
3. (steps 9-10) Redirect from wallet back to TPP
4. (step 11) The TPP sends a payment initiation request to the ASPSP (including a DPoP PoP). 
5. ... the rest is payment initiation processing

This is an example request for step 4: 

```json
{
  "client_id": "x509_san_dns:client.psp.org",
  "response_type": "vp_token",
  "redirect_uri": "https://client.psp.org/callback",
  "dcql_query": {
    "credentials": [
      {
        "id": "A2Pay",
        "format": "dc+sd-jwt",
        "meta": {
          "vct_values": [
            "https://credentials.example.com/a2pay"
          ]
        },
        "claims": [
          {
            "path": [
              "id"
            ]
          },
          {
            "path": [
              "payment-product"
            ]
          },
          {
            "path": [
              "payment-uri"
            ]
          },
          {
            "path": [
              "currency"
            ]
          },
          {
            "path": [
              "name"
            ]
          }
        ]
      }
    ]
  },
  "transaction_data": [
    {
      "type": "PaymentRequest",
      "credential_ids": [
        "A2Pay"
      ],
      "transaction_data_hashes_alg": "sha-256",
      "payment-id": "7D8AC610-566D-3EF0-9C22-186B2A5ED793",
      "creditor-account": {
        "iban": "DE75512108001245126199"
      },
      "instructed-amount": "15.49",
      "currency": "EUR",
      "creditor-name": "Merchant A",
      "purpose": "Shopping at Merchant A"
    }
  ],
  "present_to_thirdparty": {
    "credential_ids": [
      "A2Pay"
    ],
    "dpop_key": {
      "jwk": {
        "crv": "P-256",
        "kty": "EC",
        "x": "NASJ2ADuagOvraLf7O4VxcBMbantzL9dd0jpvMLnBfs",
        "y": "OJY6pqCqRIzpEt78OXasWHGgqV5ZGre_3cHtpNH82gg"
      }
    }
  }
}
```

The parameter `present_to_thirdparty` contains the additional data and indicates to the wallet a modified flow. 

* `credential_id` identifies the requested credentials, the RP wants to present to a third party. 
* `dpop_key` is the public key the credential presentation (used as access token) shall be bound to. 

Note: the `present_to_thirdparty` function MAY be combined with transaction data. 

The wallet would need to know that the respective credential presentation must be wrapped into an access token along with a `cnf` for the DPoP key. Here is an example access token: 

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
