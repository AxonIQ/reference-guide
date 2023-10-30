# SSL

Axon Server supports TLS/SSL \(Transport Layer Security/Secure Sockets Layer\) to encrypt all of Axon Server's network traffic - From Axon Framework client applications to Axon Server \(SE/EE\) as well as between Axon Server nodes within a cluster \(EE only\).

Axon Server \(SE/EE\) has two ports \(HTTP/gRPC\) that need to be enabled for SSL and hence there are two different groups of settings to use, one for each port. The HTTP port uses the generic Spring Boot configuration settings, and requires a Java compatible keystore. For the gRPC port we use standard PEM files.

The following properties need to be setup in `axonserver.properties` for both SE and EE:

<table>
  <thead>
    <tr>
      <th style="text-align:left"><em><b>Port Type</b></em>
      </th>
      <th style="text-align:left">Property Name</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><em><b>SSL (Axon Server - HTTP Port)</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">security.require-ssl</td>
      <td style="text-align:left">Determines whether the server has ssl enabled on the HTTP port.</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">server.ssl.key-store-type</td>
      <td style="text-align:left">Keystore type. (should be PKCS12)</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">server.ssl.key-store</td>
      <td style="text-align:left">Location of the keystore.</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">server.ssl.key-store-password</td>
      <td style="text-align:left">Password to access the keystore.</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">server.ssl.key-alias</td>
      <td style="text-align:left">Alias to be used to access the keystore.</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>SSL (Axon Server - gRPC Port)</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">axoniq.axonserver.ssl.enabled</td>
      <td style="text-align:left">Determines whether the server has ssl enabled on the gRPC port.</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">axoniq.axonserver.ssl.cert-chain-file</td>
      <td style="text-align:left">Location of the public certificate file.</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">axoniq.axonserver.ssl.private-key-file</td>
      <td style="text-align:left">Location of the private key file.</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.internal-cert-chain-file</td>
      <td style="text-align:left">
        <p>File containing the full certificate chain to be used in internal communication
          between Axon Server nodes. If not specified, Axon Server will use the primary key file from <em>ssl.cert-chain-file</em>.</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.internal-trust-manager-file</td>
      <td style="text-align:left">
        <p>Trusted certificates for verifying the other AxonServer&apos;s certificate.</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.internal-private-key-file</td>
      <td style="text-align:left">
        <p>File containing the private key to be used in internal communication
          between Axon Server nodes. If not specified, Axon Server will use the primary key file from <em>ssl.private-key-file</em>.</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
    </tr>
  </tbody>
</table>

With Axon Server EE SSL is also used for the communication between the Axon Server nodes. If the internal host names of the Axon Server
nodes differ from the host names as they are used by clients, it is required to use another certificate (bound to the internal hostname).
If this is the case, you can specify the certificate used for cluster-internal traffic using the “...ssl.internal-cert-chain-file” property.

The certificate used by internal traffic may be generated using its own private key. If this is the case, you must specify the location
of this key file in the property "...ssl.internal-private-key-file".

If the certificates used for internal traffic are self-signed certificates, you must ensure that these are trusted by the other nodes.
In this case you add the certificates (or the root certificate) in the (PEM) keystore specified by 
the “...ssl.internal-trust-manager-file” property.

As of Axon Server version 2023.2.0, you can update the certificates without restarting Axon Server. 

When a certificate is about to expire you can replace it with a new certificate, using the same file name. 
To load the new certificate for the gRPC communication you must ensure that the modified timestamp for both the key file and the certificate file are updated. 

The optional trust manager file used for the internal communication in an Axon Server cluster is also monitored. If this file is updated, Axon Server will use the new version.

When you update the certificate in the keystore used for the Tomcat HTTP server, it is also reloaded automatically.

Axon Server checks the TLS artifacts for updates once a minute.

## Client configuration

The following properties are available for Axon client applications to use TLS/SSL for the traffic to Axon Server:
<table>
  <thead>
    <tr>
      <th style="text-align:left">Property Name</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td style="text-align:left">axon.axonserver.ssl-enabled</td>
        <td style="text-align:left">Is SSL enabled for the traffic with Axon Server</td>
    </tr>
    <tr>
        <td style="text-align:left">axon.axonserver.cert-file</td>
        <td style="text-align:left">(PEM) keystore containing trusted certificates, in case that the certificate 
that is used by Axon Server is not issued by a trusted certificate authority.</td>
    </tr>
  </tbody>
</table>

## Downtime Considerations

A thing to remember is that enabling SSL on an Axon Server cluster will require downtime, as the “...ssl.enabled” setting controls both server and client side code. This is intentional, as it is unreasonable to expect all nodes to have individual settings per node showing which ones communicate using SSL and which do not, so it is recommended to get this done in the beginning during the installation phase of Axon Server.

