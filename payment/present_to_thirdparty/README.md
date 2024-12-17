
This document describes a design to use the wallet to authorize a payment and to be able to initate this payment through an API call between the RP and a bank (a third party service). 

The basic idea is to use a VC issued by the bank into the holder's wallet along with the transaction data feature of OpenID4VP to authorize the payment. On top of that, the result of the process, the credential presentation, shall be used with the bank to authorize the payment initation API call. To make this a secure process, the credential presentation is wrapped into an access token that is bound to a DPoP key provided by the RP and audience and origin restricted. 

The new option is called "present to 3rd party".

OpenID4VP "present to 3rd party" is an alterative proposal to using OAuth (../oauth/README.md). Although more of less equivalent from a conceptual standpoint, it is an extension of the OpenID4VP flow. That has the following advantages: 

* It is an extension of the "normal" transation data flow not requiring implementation of an additional protocol.
* It allows the request and presentation of other credentials in the same flow. 
* It can be used over HTTPS and Digital Credential Profile API.

This flow can be used for any use case where a RP wants to use a credential from a wallet to securely invoke an API on behalf of the user, examples are:

* Authorization of a (Qualified) Electronic Signature, where the signature is created with a remote signature creation service
* Authentication of an app acting on behalf of the user in a messaging system (e.g. Matrix)

![Alt text](https://github.com/tlodderstedt/eudiw_advanced/blob/main/out/payment/present_to_thirdparty/present_to_thirdparty_sca/present_to_thirdparty_sca.png "Payment Initiation with present to 3rd party")

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
  "nonce": "n-0S6_WzA2Mj",
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
          {"path": ["id"]},
          {"path": ["payment-product"]},
          {"path": ["payment-uri"]},
          {"path": ["currency"]},
          {"path": ["name"]}
        ]
      }
    ]
  },
  "transaction_data": [
    eyAidHlwZSI6IC...cGluZyBhdCBNZXJjaGFudCBBIiB9
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

The wallet will ask the user for consent for the transaction and produce the response. The wallet will wrap the respective credential presentation into an JWT-based access token along with a `cnf` for the DPoP key and the `client_id` of the authenticated wallet client. Here is an example access token: 

```json
{
    "iss": "https://issuer.eudiw.com",
    "aud": "https://bank.example.com",
    "client_id": "x509_san_dns:client.psp.org",
    "exp": 1883000000,
    "nbf": 1718198433,
    "iat": 1718198433,
    "credential_presentation": "eyJhbGciOiAiRVMy..V0~Wy..ZnBAan5g",
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

Note: Even though the access token uses the JWT-format, this design is intended to work with _any_ credential format. 

This access token is signed with the same key that is used to sign the credential presentation. The access token is encrypted with a public key of the bank. This key would need to be determined either from data in the A2Pay credential or through metadata the wallet requests from the bank. 

The access token is returned in the `vp_token` instead of the credential presentation in a structure containing the following claims:

* `access_token`: the access token
* `nonce`: the `nonce` as send in the request.
* `metadata`: Additional metadata the wallet shares with the RP. In the case of the A2Pay credential, this is the payment endpoint URI. 

Here is an example:

```json
{
    "vp_token": {
        "A2Pay": {
            "access_token": eyJhbGciOi...00ifQ.OKOaw...zg.48...3b.5eym8T...QD_A.XFBo...SkQ,
            "nonce": "n-0S6_WzA2Mj",
            "metadata": {
                "payment-uri": "https://bank.example.com/pay/7dfe5484g78"
            }
        }
    }
}
```

Note: depending on the use case, the RP might know the API  to send the access token to in advance. In such cases, the RP would add them to the `present_to_thirdparty`structure. If not, the wallet determines those and add it to the response. 

The TPP will send this access token along with the DPoP proof in step 11.
