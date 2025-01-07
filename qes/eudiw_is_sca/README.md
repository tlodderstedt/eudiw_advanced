
![Alt text](https://github.com/tlodderstedt/eudiw_advanced/blob/main/out/qes/eudiw_is_sca/qes_eudiw_is_sca/qes_eudiw_is_sca.png "Signing in the EUDIW")

In this flow, the EUDIW fetches the document(s) to be signed from the RP and responds with the signed documents to the RP using a similar message flow as for OpenID4VP. 

From the RP's perspective, the EUDIW takes care of the whole signing process, including showing the document to the user, gathering consent, determining the certificate, create the signature, add the signature to the document.

In case of a remote QES, the EUDIW will utilize a CSC compliant QSCD to create the signature. The signature authorization is conducted using OAuth. This way, the QSCD can use pre-provisioned EAAs, a PID presentation, or a pseudonymous authentication from the EUDIW to obtain the data and confirmation from the user.  

There are two options to design the interface between RP and EUDIW. 

1. ./oid4vp_alike: A new protocol with a OID4VP alike message flow, where the messages are based on the CSC standard. 

1. ./oid4vp: Use of OID4VP with transaction data, where the document to be signed is a transaction data elememt and the signed document is treated as credential presentation. 