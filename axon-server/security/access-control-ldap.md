# Axon Server EE - LDAP Extension

This extension will allow you to use LDAP for authentication and authorization in Axon Server. User accounts and roles
from LDAP are not synchronized to the Axon Server cluster, so they won't show up on the "Users" tab.

## Installing the LDAP Extension

To install the LDAP Extension, you need to unpack the distribution ZIP-file, so the JAR files are in the "`exts`"
subdirectory of the working directory of Axon Server:

```text
$ mkdir exts
$ unzip -j axon-server-extension-ldap-4.5.1-bin.zip -d exts
Archive:  axon-server-extension-ldap-4.5-bin.zip
  inflating: exts/axon-server-extension-ldap-4.5.1-sources.jar
  inflating: exts/axon-server-extension-ldap-4.5.1.jar
  inflating: exts/javax.inject-1.jar
  inflating: exts/spring-ldap-core-2.3.4.RELEASE.jar
  inflating: exts/spring-security-ldap-5.4.7.jar
$
```

Note that version 4.5.1 is the current version at the time of Axon Server 4.5.13.

## Configuring the LDAP Extension

The options used are:

* `axoniq.axonserver.accesscontrol.enabled`

    This must be set to "`true`" to enable access control.

* `axoniq.axonserver.enterprise.ldap.enabled`

    Set this to "`true`" to enable the plugin.
  
* `axoniq.axonserver.enterprise.ldap.server-name`

    This sets the LDAP server's hostname, which is defaulted to "`localhost`".  

* `axoniq.axonserver.enterprise.ldap.server-port`

    This sets the LDAP server's port, which is defaulted to `389`.

* `axoniq.axonserver.enterprise.ldap.server-url`

    As an alternative, for example when you want to use a TLS-secured connection, you can provide the URL to the LDAP
    server, such as "`ldaps://ldap-server.local`".

* `axoniq.axonserver.enterprise.ldap.initialBindUserDN`

    If the LDAP server does not accept unauthenticated initial binds, set the DN of the user for that, for example
    "`cn=admin,dc=demo,dc=io`". Only if both this property and the corresponding password are set, will they be used.

* `axoniq.axonserver.enterprise.ldap.initialBindPassword`

    If the LDAP server does not accept unauthenticated initial binds, set the password for that. Only if both
    this property and the corresponding User DN are set, will they be used.

* `axoniq.axonserver.enterprise.ldap.search-base`

    This setting provides the base context for searching users, for example "`ou=people,dc=planetexpress,dc=com`".

* `axoniq.axonserver.enterprise.ldap.search-filter`

    This is the filter to be used for searching, so you typically add object types, and the attribute to match on. An
    example would be "`(&(objectClass=inetOrgPerson)(uid={0}))`". The "`{0}`" notation is used to place the username.

  * `axoniq.axonserver.enterprise.ldap.group-base`

      Similarly to the "`search-base`" setting, you can add a "`group-base`". This setting is optional and normally not
      needed.

* `axoniq.axonserver.enterprise.ldap.group-filter`

    The "`group-filter`" is the search pattern for groups, which will be translated to roles, for example
    "`(&(objectclass=Group)(member={0}))`"

### Active Directory specific settings

When using ActiveDirectory, the following properties are needed:

* `axoniq.axonserver.enterprise.ldap.activeDirectory`

    This must be set to "`true`".
* `axoniq.axonserver.enterprise.ldap.adDomain`

    This must be set to the AD Domain serviced by the controller.

An example of an Active Directory configuration is:

```properties
axoniq.axonserver.accesscontrol.enabled=true
axoniq.axonserver.enterprise.ldap.enabled=true
axoniq.axonserver.enterprise.ldap.activeDirectory=true
axoniq.axonserver.enterprise.ldap.serverName=my-ad
axoniq.axonserver.enterprise.ldap.adDomain=demo.io
axoniq.axonserver.enterprise.ldap.searchFilter=(&(objectClass=user)(sAMAccountName={1}))
axoniq.axonserver.enterprise.ldap.roles.AxonAdmin=ADMIN@_admin
```

## Linking LDAP groups to roles in Axon Server

In order to translate LDAP Groups, you must provide properties as follows:

```properties
axoniq.axonserver.enterprise.ldap.roles.<group-name>=<role>
```

For example, if we have a group "ADMIN_STAFF" that we want to make administrators, and a group "SHIP_CREW" that should
be normal users of the "default" context, then we would use:

```properties
axoniq.axonserver.enterprise.ldap.roles.ADMIN_STAFF=ADMIN@_admin
axoniq.axonserver.enterprise.ldap.roles.SHIP_CREW=USE_CONTEXT@default
```
## Tuning the LDAP Extension

Two further options exist that may be used to tune the connection with the LDAP server. Both have a default value of "`true`".

* `axoniq.axonserver.enterprise.ldap.using-pooled-queries`
* `axoniq.axonserver.enterprise.ldap.allowing-referrals`