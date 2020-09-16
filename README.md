# OpenLDAP Docker Image for testing

![Docker Build Status](https://img.shields.io/docker/build/rroemhild/test-openldap.svg) ![Docker Stars](https://img.shields.io/docker/stars/rroemhild/test-openldap.svg) ![Docker Pulls](https://img.shields.io/docker/pulls/rroemhild/test-openldap.svg)

This image provides an OpenLDAP Server for testing LDAP applications, i.e. unit tests. The server is initialized with the example domain `planetexpress.com` with data from the [Futurama Wiki][futuramawikia].

Parts of the image are based on the work from Nick Stenning [docker-slapd][slapd] and Bertrand Gouny [docker-openldap][openldap].

The Flask extension [flask-ldapconn][flaskldapconn] use this image for unit tests.

[slapd]: https://github.com/nickstenning/docker-slapd
[openldap]: https://github.com/osixia/docker-openldap
[flaskldapconn]: https://github.com/rroemhild/flask-ldapconn
[futuramawikia]: http://futurama.wikia.com


## Features

* Initialized with data from Futurama
* Support for TLS (snake oil cert on build)
* memberOf overlay support
* MS-AD Style Groups support
* ~124MB images size (~40MB compressed)


## Usage

```
docker pull rroemhild/test-openldap
docker run --privileged -d -p 389:389 rroemhild/test-openldap
```

## Exposed ports

* 389
* 636

## Exposed volumes

* /etc/ldap/slapd.d
* /etc/ldap/ssl
* /var/lib/ldap
* /run/slapd


## LDAP structure

### dc=example,dc=com

| Admin            | Secret           |
| ---------------- | ---------------- |
| cn=admin,dc=example,dc=com | GoodNewsEveryone |

### ou=openid,dc=example,dc=com

#### cn=Homer Simpson,ou=openid,dc=example,dc=com

| Attribute        | Value            |
| ---------------- | ---------------- |
| objectClass      | inetOrgPerson |
| cn               | Homer Simpson |
| sn               | Simpson |
| givenName        | Homer |
| mail             | hsimpson@example.com |
| uid              | 90344.ASDFJWFC |
| userPassword     | 007 |


### cn=Test User 1,ou=openid,dc=planetexpress,dc=com

| Attribute        | Value            |
| ---------------- | ---------------- |
| objectClass      | inetOrgPerson |
| cn               | Test User 1 |
| sn               | User |
| givenName        | Test |
| mail             | testuser1@example.com |
| uid              | 90342.ASDFJWFA |
| userPassword     | 007 |


### cn=Test User 2,ou=openid,dc=example,dc=com

| Attribute        | Value            |
| ---------------- | ---------------- |
| objectClass      | inetOrgPerson |
| cn               | Test User 2 |
| sn               | User |
| givenName        | Test |
| mail             | testuser2@example.com |
| uid              | 923478.WESRJKSL |
| userPassword     | 007 |


## JAAS configuration

In case you want to use this OpenLDAP server for testing with a Java-based
application using JAAS and the `LdapLoginModule`, here's a working configuration
file you can use to connect.

```
other {
  com.sun.security.auth.module.LdapLoginModule REQUIRED
    userProvider="ldap://localhost/ou=people,dc=planetexpress,dc=com"
    userFilter="(&(uid={USERNAME})(objectClass=inetOrgPerson))"
    useSSL=false
    java.naming.security.principal="cn=admin,dc=planetexpress,dc=com"
    java.naming.security.credentials="GoodNewsEveryone"
    debug=true
    ;
};
```

This config uses the admin credentials to connect to the OpenLDAP server and to
submit the search query for the user that enters their credentials. As username
the `uid` attribute of each entry is used.
