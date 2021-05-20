# Axon Server EE - OAuth Extension

This extension will allow you to use OAuth2 integration (currently only with Google) for authentication in Axon Server.
User accounts and roles from the OAuth provider are not synchronized to the Axon Server cluster, because no roles are
associated with them. This means an account needs to be made in the cluster with the username from the provider, and
roles assigned.

## Installing the OAuth Extension

To install the OAuth Extension, you need to unpack the distribution ZIP-file, so the JAR files are in the "`exts`"
subdirectory of the working directory of Axon Server:

```text
$ mkdir exts
$ unzip -j axon-server-extension-oauth-4.5-SNAPSHOT-bin.zip -d exts
Archive:  axon-server-extension-oauth-4.5-SNAPSHOT-bin.zip
  inflating: exts/axon-server-extension-oauth-4.5-SNAPSHOT.jar
  inflating: exts/commons-compress-1.9.jar
  inflating: exts/commons-lang3-3.8.1.jar
  inflating: exts/content-type-2.1.jar
  inflating: exts/javax.inject-1.jar
  inflating: exts/javax.persistence-api-2.2.jar
  inflating: exts/javax.transaction-api-1.3.jar
  inflating: exts/jcip-annotations-1.0-1.jar
  inflating: exts/lang-tag-1.4.4.jar
  inflating: exts/nimbus-jose-jwt-9.1.3.jar
  inflating: exts/oauth2-oidc-sdk-8.23.1.jar
  inflating: exts/spring-boot-starter-oauth2-client-2.1.6.RELEASE.jar
  inflating: exts/spring-security-oauth2-client-5.1.5.RELEASE.jar
  inflating: exts/spring-security-oauth2-core-5.1.5.RELEASE.jar
  inflating: exts/spring-security-oauth2-jose-5.1.5.RELEASE.jar
  inflating: exts/tomcat-embed-el-9.0.21.jar
  inflating: exts/validation-api-2.0.1.Final.jar
$
```

Note that the actual version numbers may differ in your case.

## Configuring the OAuth Extension

The options used are:

* `axoniq.axonserver.accesscontrol.enabled`

  This must be set to "`true`" to enable access control.

* `axoniq.axonserver.enterprise.oauth2.enabled`

  This must be set to "`true`" to enable the OAuth extension.

* `axoniq.axonserver.enterprise.oauth2.authorization-uri`

  This optional value can be used to configure the URI that will trigger the authentication using OAuth2. The default
  value is "`/oauth2/authorization`" and should work fine.

* `spring.security.oauth2.client.registration.google.client-id`

  This should be set to the client-id provided by the Google Developer Console where you registered the cluster.

* `spring.security.oauth2.client.registration.google.client-secret`

  This should be set to the secret provided by the Google Developer Console where you registered the cluster.

* `spring.security.oauth2.client.registration.google.scope`

  This setting is used to configure what information Google should share with the Axon Server cluster. A good value to
  use is "`email`", which will allow you to use the email address as username, as is common with Google accounts.

* `axoniq.axonserver.enterprise.oauth2.username-map.google`

  This setting tells the _extension_ what value to use as username and requires that this value is provided by Google
  using the "`scope`" setting described above. If the email address is to be used, as suggested above, the value should
  be "`email`".

* `axoniq.axonserver.enterprise.oauth2.request-params`

  This setting defines a map of parameters to add to the redirect URL, to customize the behavior of the provider's
  integration. For Google, if the users use the same browser with multiple Google accounts, a good setting to add is
  "`prompt`", with value "`select_account`":

  ```properties
  axoniq.axonserver.enterprise.oauth2.request-params.prompt=select_account
  ```

  This will force Google to always ask which account must be used to continue, even if there is only a single account
  in use, and that account is currently active.

## Configuring the User's Access and Roles

If a username is unknown in the Axon Server cluster, even when authentication succeeds, the user will not be allowed to
log in. To allow this, a user with "`ADMIN`" level access needs to create a user, optionally without a password, and
assign the roles for this user. The Axon Server CLI has a special options ("`--no-password`") to allow the creation of
accounts without a password. Note that if you create an account _with_ a password, this will allow the user to choose
to use that rather than the OAuth integration.