The idea of this design is to leverage the protocol mechanics of OpenID4VP and the trust model of the EUDIW to allow RPs to request a payment through a EUDIW.

The EUDIW acts as an OAuth Authorization Server and issues a sender-constrained access token, the RP will use to initate a payment with a financial institution. See OAuth sub folder. 

The new messages (or rather the message parameters) can be combined with OpenID4VP message parameter, i.e. signature creation could be combined with credential presentation.

Alternative idea: the normal OpenID4VP flow could be extended to support the presentation of a VC to a third party (which basically is an access token). See delegated_sca sub folder.