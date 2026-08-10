---
myst:
  html_meta:
    description: "OpenJDK OpenSSL FIPS packages for Ubuntu Pro"
---

(openjdk-openssl-fips)=

# OpenJDK OpenSSL FIPS packages for Ubuntu Pro

Canonical provides FIPS (Federal Information Processing Standards) certified cryptographic modules, and enabled kernels, with the Ubuntu Pro offering. The OpenSSL FIPS modules, validated and [certified](https://csrc.nist.gov/projects/cryptographic-module-validation-program/validated-modules/search?SearchMode=Basic&Vendor=Canonical&ModuleName=openssl&CertificateStatus=Active&ValidationYear=0) for the FIPS 140-3 standard by NIST, are available with an Ubuntu Pro subscription on [Ubuntu 22.04](https://ubuntu.com/security/certifications/docs/2204/fips) and [Ubuntu 24.04](https://ubuntu.com/security/certifications/docs/2404).

Cryptography applications written in Java that are required to be FIPS-compliant may use a FIPS-certified, or FIPS-compliant, Java security provider. One such provider is the BouncyCastle FIPS (BCFIPS) provider which is FIPS 140-3 certified. Ubuntu Pro offers OpenJDK packages bundling this provider: openjdk-11-fips on Ubuntu 20.04, openjdk-17-fips and openjdk-21-fips on Ubuntu 24.04. Additionally, we now have OpenJDK packages which integrate with the system-installed OpenSSL libraries. These packages bundle a new Java security [provider](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/security/Provider.html) that interfaces with the OpenSSL FIPS module.

## The OpenSSL FIPS Java security provider

The [OpenSSL FIPS Java security provider](https://github.com/canonical/openssl-fips-java) creates Java bindings to the underlying OpenSSL library, including the OpenSSL FIPS module. Programmatically named `OpenSSLFIPSProvider`, it exposes a restricted set of FIPS-approved OpenSSL algorithms through the Java Crytpography Extensions ([JCE](https://en.wikipedia.org/wiki/Java_Cryptography_Extension)) interface. It lets Java applications invoke these algorithms through the Java Cryptography Architecture (JCA). Under the hood, it uses the Java Native Interface to interact with OpenSSL.

The FIPS-approved algorithms provided by OpenSSLFIPSProvider include the following cryptography functions:
 - Deterministic Random Bit Generators (DRBG)
 - Ciphers
 - Key Agreements
 - Key Derivations
 - Key Encapsulation
 - Message Digests (MD)
 - Message Authentication Codes (MAC)
 - Signatures

This is the [complete list](https://github.com/canonical/openssl-fips-java/blob/main/doc/algorithms.md) of algorithms offered by OpenSSLFIPSProvider along with their JCA/JCE names.


## OpenJDK OpenSSL FIPS packages on Ubuntu Pro

The Java Cryptography Architecture ([JCA](https://docs.oracle.com/en/java/javase/21/security/java-cryptography-architecture-jca-reference-guide.html)) lets applications invoke security algorithms through reflection, which implies a loose binding. The Java application may define provider configuration and invoke algorithms without having the security provider classes on the build-time classpath. Unless an application's unit-tests exercise a security provider, it suffices to have the latter's classes available on the runtime classpath alone.

The OpenJDK OpenSSL FIPS packages provide a ready-to-use OpenJDK FIPS runtime. They contain a launcher that auto-configures the security policy and initializes the classpath with a path to OpenSSLFIPSProvider's jar file. The application needs to ensure that the rest of the required classpath is configured. They also include a jlink'd Java runtime which will be based on the latest OpenJDK security updates, at any given time.  

The OpenJDK OpenSSL FIPS packages are available with a Ubuntu Pro subscription, on Ubuntu 22.04 and Ubuntu 24.04:
- openjdk-17-fips-openssl-jre and openjdk-17-fips-openssl-jre-headless provide an OpenJDK 17 launcher and runtime, bundled with the OpenSSL FIPS provider.
- openjdk-21-fips-openssl-jre and openjdk-21-fips-openssl-jre-headless provide an OpenJDK 21 launcher and runtime, bundled with the OpenSSL FIPS provider jar.


## Further reading

Refer to {ref}`use-openjdk-openssl-fips` OpenJDK OpenSSL FIPS guide for instructions to run a FIPS-compliant Java application on Ubuntu Pro.