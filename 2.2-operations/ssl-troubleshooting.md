# 2.2.3 SSL troubleshooting

*Symptom*: Client receives exception _INTERNAL: Connection closed with unknown cause_, no logging on the server.

*Cause*: Server is running in SSL mode, client is not.

*Solution*: Add certificate to client configuration.

*Symptom*: Client receives exception _UNAVAILABLE_. Server gives error: _HTTP/2 client preface string missing or corrupt_.

*Cause*: Client is using SSL, server is not.

*Solution*: Add SSL configuration to the server.

*Symptom*: Client receives exception _UNAVAILABLE_. Server gives no error.

*Cause*: Possibly the hostname used by the client does not match the hostname pattern in the certificate.

*Solution*: Check certificate values from client by starting the client with java option -Djavax.net.debug=all. This will output something like:

 > adding as trusted cert:
 >   Subject: CN=*.axoniq.io, O=AxonIQ
 >   Issuer:  CN=*.axoniq.io, O=AxonIQ
 >   Algorithm: RSA; Serial number: 0x92047296751fe115
 >   Valid from Mon Jul 31 15:18:29 CEST 2017 until Sat Jul 30 15:18:29 CEST 2022

Verify that the name for the server matches the common name (CN) for the subject.

*Symptom*: Client receives error _No name matching [hostname] found_

*Cause*: Server is not returning a fully qualified name. During the initial connection from client to server the server returns the address of the master node.

*Solution*: add property `axoniq.axonserver.domain` to the server configuration.
