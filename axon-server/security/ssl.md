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
      <td style="text-align:left">axoniq.axonserver.ssl.internal-cert-chain-file</td>
      <td style="text-align:left">
        <p>File containing the full certificate chain to be used in internal communication
          between Axon Server nodes.</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">axoniq.axonserver.ssl.internal-trust-manager-file</td>
      <td style="text-align:left">
        <p>Trusted certificates for verifying the other AxonServer&apos;s certificate.</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
    </tr>
  </tbody>
</table>

With Axon Server EE we have two extra settings for the internal gRPC port; “...ssl.internal-cert-chain-file” and “...ssl.internal-trust-manager-file”.

The first is for the PEM certificate to be used for cluster-internal traffic, if it is different from the one used for client connections. The most common reason is when the nodes use a different DNS name for internal \(cluster node to cluster node\) communication than for external connections.

The second is for a \(PEM\) keystore that certifies the other certificates, which may be needed when they are signed using an authority that is not available from the Java JDK’s CA keystore.

## Downtime Considerations

A thing to remember is that enabling SSL on an Axon Server cluster will require downtime, as the “...ssl.enabled” setting controls both server and client side code. This is intentional, as it is unreasonable to expect all nodes to have individual settings per node showing which ones communicate using SSL and which do not, so it is recommended to get this done in the beginning during the installation phase of Axon Server.

