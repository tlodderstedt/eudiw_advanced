The idea of this design is to leverage the protocol mechanics of OpenID4VP and the trust model of the EUDIW to allow RPs to request a EUDIW to create a Qualified Electronic Signature.

There are two flavors:

1. The RP uses the EUDIW as raw signing device (RP is Signature Creation Application / SCA): The RP shows the document to be signed and also incorporates the signature data into this document (hash)
1. The RP uses the EUDIW to handle the signature completely (EUDIW is the SCA): The EUDIW shows the document to the user, adds the signature and sends the signed document back to the RP. 