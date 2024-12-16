The idea of this design is to leverage the protocol mechanics of OpenID4VP and the trust model of the EUDIW to allow RPs to request a payment through a EUDIW.

There are two options 

* ./oauth: The EUDIW acts as an OAuth Authorization Server and issues a sender-constrained access token, the RP will use to initate a payment with a financial institution. 
* ./present_to_thirdparty: The normal OpenID4VP flow could be extended to support the presentation of a VC to a third party (which basically is an access token). 