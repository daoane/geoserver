# Passwords {: #security_passwd }

Passwords are a central aspect of any security system. This section describes how GeoServer handles passwords.

## Password encryption {: #security_passwd_encryption }

A GeoServer configuration stores two types of passwords:

-   Passwords for **user accounts** to access GeoServer resources
-   Passwords used internally for **accessing external services** such as databases and cascading OGC services

As these passwords are typically stored on disk it is strongly recommended that they be encrypted and not stored as human-readable text. GeoServer security provides four schemes for encrypting passwords: **empty**, **plain text**, **Digest**, and **Password-based encryption (PBE)**.

The password encryption scheme is specified as a global setting that affects the encryption of passwords used for external resources, and as an encryption scheme for each [user/group service](usergrouprole/usergroupservices.md). The encryption scheme for external resources has to be [reversible](passwd.md#security_passwd_reversible), while the user/group services can use any scheme.

### Empty

The scheme is not reversible. Any password is encoded as an empty string, and as a consequence it is not possible to recalculate the plain text password. This scheme is used for user/group services in combination with an authentication mechanism using a back end system. Examples are user name/password authentication against a LDAP server or a JDBC database. In these scenarios, storing passwords locally to GeoServer does not make sense.

### Plain text

Plain text passwords provide no encryption at all. In this case, passwords are human-readable by anyone who has access to the file system. For obvious reasons, this is not recommended for any but the most basic test server. A password `mypassword` is encoded as `plain:mypassword`, the prefix uniquely describing the algorithm used for encoding/decoding.

### Digest

Digest encryption is not reversible. It applies, 100,000 times through an iterative process, a SHA-256 [cryptographic hash function](http://en.wikipedia.org/wiki/Cryptographic_hash_function) to passwords. This scheme is "one-way" in that it is virtually impossible to reverse and obtain the original password from its hashed representation. Please see the section on [Reversible encryption](passwd.md#security_passwd_reversible) for more information on reversibility.

To protect from well known attacks, a random value called a [salt](http://en.wikipedia.org/wiki/Salt_%28cryptography%29) is added to the password when generating the key. For each digesting, a separate random salt is used. Digesting the same password twice results in different hashed representations.

As an example, the password `geoserver` is digested to `digest1:YgaweuS60t+mJNobGlf9hzUC6g7gGTtPEu0TlnUxFlv0fYtBuTsQDzZcBM4AfZHd`. `digest1` indicates the usage of digesting. The hashed representation and the salt are base 64 encoded.

### Password-based encryption

[Password-based encryption](http://www.javamex.com/tutorials/cryptography/password_based_encryption.shtml) (PBE) normally employs a user-supplied password to generate an encryption key. This scheme is reversible. A random salt described in the previous section is used.

!!! note

    The system never uses passwords specified by users because these passwords tend to be weak. Passwords used for encryption are generated using a secure random generator and stored in the GeoServer key store. The number of possible passwords is 2\^260 .

GeoServer supports two forms of PBE. **Weak PBE** (the GeoServer default) uses a basic encryption method that is relatively easy to crack. The encryption key is derived from the password using [MD5](http://en.wikipedia.org/wiki/Message_Digest_Algorithm_5) 1000 times iteratively. The encryption algorithm itself is [DES](http://en.wikipedia.org/wiki/Data_Encryption_Standard) (Data Encryption Standard). DES has an effective key length of 56 bits, which is not really a challenge for computer systems in these days.

**Strong PBE** uses a much stronger encryption method based on an [AES](http://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 256-bit algorithm with [CBC](http://en.wikipedia.org/wiki/Block_cipher_modes_of_operation). The key length is 256 bit and is derived using [SHA-256](http://en.wikipedia.org/wiki/SHA-2) instead of MD5. Using Strong PBE is highly recommended.

As an example, the password `geoserver` is encrypted to `crypt1:KWhO7jrTz/Gi0oTQRKsVeCmWIZY5VZaD`. `crypt1` indicates the usage of Weak PBE. The prefix for Strong PBE is `crypt2`. The ciphertext and the salt are base 64 encoded.

### Reversible encryption {: #security_passwd_reversible }

Password encryption methods can be **reversible**, meaning that it is possible (and desirable) to obtain the plain-text password from its encrypted version. Reversible passwords are necessary for database connections or external OGC services such as [cascading WMS](../data/cascaded/wms.md) and [cascading WFS](../data/cascaded/wfs.md), since GeoServer must be able to decode the encrypted password and pass it to the external service. Plain text and PBE passwords are reversible.

Non-reversible passwords provide the highest level of security, and therefore should be used for user accounts and wherever else possible. Using password digesting is highly recommended, the installation of the unrestricted policy files is not required.

## Secret keys and the keystore {: #security_passwd_keystore }

For a reversible password to provide a meaningful level of security, access to the password must be restricted in some way. In GeoServer, encrypting and decrypting passwords involves the generation of secret shared keys, stored in a typical Java *keystore*. GeoServer uses its own keystore for this purpose named `geoserver.jceks` which is located in the `security` directory in the GeoServer data directory. This file is stored in the [JCEKS format rather than the default JKS](http://www.itworld.com/nl/java_sec/07202001). JKS does not support storing shared keys.

The GeoServer keystore is password protected with a [Keystore password](passwd.md#security_master_passwd). It is possible to access the contents of the keystore with external tools such as [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html). For example, this following command would prompt for the keystore password and list the contents of the keystore:

``` bash
$ keytool -list -keystore geoserver.jceks -storetype "JCEKS"
```

## Keystore password {: #security_master_passwd }

It is also possible to set a **keystore password** for GeoServer. This password serves two purposes:

-   Protect access to the [keystore](passwd.md#security_passwd_keystore)
-   Protect access to the GeoServer [Root account](root.md)

By default, the keystore password is generated and stored in a file named `security/masterpw.info` using plain text. When upgrading from an existing GeoServer data directory (versions 2.1.x and lower), the algorithm attempts to figure out the password of a user with the role `ROLE_ADMINISTRATOR`. If such a password is found and the password length is 8 characters at minimum, GeoServer uses this password as keystore password. Again, the name of the chosen user is found in `security/masterpw.info`.

!!! warning

    The file `security/masterpw.info` is a security risk. The administrator should read this file and verify the keystore password by logging on GeoServer as the `root` user. On success, this file should be removed.

!!! warning

    The first thing an Administrator of the System should do is dump the keystore Password generated by GeoServer, store it somewhere not accessible by anyone, and delete `security/masterpw.info` or whatever file you dumped the password to.

Refer to [Active keystore password provider](webadmin/passwords.md#security_webadmin_masterpasswordprovider) for information on how to change the keystore password.

!!! note

    By default login to the Admin GUI and REST APIs using the Keystore Password is disabled. In order to enable it you will need to manually change the Keystore Password Provider config.xml, usually located in `security/masterpw/default/config.xml`, by adding the following statement:
    
        ``<loginEnabled>true</loginEnabled>``

## Password policies {: #security_passwd_policy }

A password policy defines constraints on passwords such as password length, case, and required mix of character classes. Password policies are specified when adding [User/group services](usergrouprole/usergroupservices.md) and are used to constrain passwords when creating new users and when changing passwords of existing users.

Each user/group service uses a password policy to enforce these rules. The default GeoServer password policy implementation supports the following optional constraints:

-   Passwords must contain at least one number
-   Passwords must contain at least one upper case letter
-   Passwords must contain at least one lower case letter
-   Password minimum length
-   Password maximum length

## Parametrized Passwords

It is possible to parametrize users' passwords in a similar way to the catalog settings (see [Parameterize catalog settings](../datadirectory/configtemplate.md)). Parametrization is supported when the encryption method used to store the place holder in the password field is plain text or is reversible (pbe, strong pbe). Non reversible encoding for the placeholder (e.g. digest) is not supported. On the contrary the actual value can be defined in the `geoserver-environment.properties` with any password encoding method supported by GeoServer. Examples are provided below:

``` properties
pwd.one=plain:clear_text_password
pwd.two=digest1:D9miJH/hVgfxZJscMafEtbtliG0ROxhLfsznyWfG38X2pda2JOSV4POi55PQI4tw
pwd.three=crypt1:xZJscMafEtbtliG0ROxhLfsznyWfG38X2pda2JOSV4POi55PQI4tw
```