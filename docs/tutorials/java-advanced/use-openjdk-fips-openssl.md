---
myst:
  html_meta:
    description: "Running Java FIPS applications using OpenJDK OpenSSL FIPS"
---

(use-openjdk-openssl-fips)=
# Running Java FIPS applications using OpenJDK OpenSSL FIPS

This tutorial has instructions to run a simple Java FIPS application on Ubuntu Pro. We will also build the application on the same Pro instance. In a more practical setting, the application would ideally be built in a continuous integration (CI) step.

1. If you do not have access to an Ubuntu Pro subscription a good place to start would be the Ubuntu Pro [documentation](https://documentation.ubuntu.com/pro/start-here/#start-here). Once you have a subscription, follow these [instructions](https://documentation.ubuntu.com/pro/attach-tutorial/) to attach a machine to your subscription.

:::{important}
Getting an Ubuntu Pro subscription and attaching your machine to it is a strict prerequisite for this tutorial.
:::

2. Ensure the `fips-updates` service is enabled. Run `pro status --all` and check the `fips-updates` service:

```{terminal}
$ pro status

SERVICE          ENTITLED  STATUS       DESCRIPTION
anbox-cloud      yes       disabled     Scalable Android in the cloud
esm-apps         yes       enabled      Expanded Security Maintenance for Applications
esm-infra        yes       enabled      Expanded Security Maintenance for Infrastructure
fips-updates     yes       disabled     FIPS compliant crypto packages with stable security updates
landscape        yes       disabled     Management and administration tool for Ubuntu
livepatch        yes       enabled      Canonical Livepatch service
realtime-kernel  yes       disabled     Ubuntu kernel with PREEMPT_RT patches integrated
usg              yes       disabled     Security compliance and audit tools

Enable services with: pro enable <service>

```
In the sample output above, the `fips-updates` service is marked as disabled. Let's enable it:

```{terminal}
pro enable fips-updates
```

This will configure additional APT sources and install the fips packages. Restart the machine after the command completes successfully. On a restart, you must find `fips-updates` enabled:

```{terminal}
$ pro status 2>&1 | grep "fips-updates"
fips-updates     yes                enabled               FIPS compliant crypto packages with stable security updates
```

:::{important}
This step cannot be undone.
:::


3. Install the OpenJDK packages. 

```{terminal}
sudo apt update && \
  sudo apt install openjdk-17-jdk openjdk-21-fips-openssl-jre -y
```

4. Clone the sample FIPS application. This tutorial uses a very basic Java sample:

```{terminal}
git clone https://github.com/pushkarnk/fips-demo-app.git
cd fips-demo-app
```

A few notes about the sample application:
 - The algorithm names are not hard-coded. They are centralized into a properties file (`src/main/resources/fips.properties`) and loaded by a configuration class (`src/main/java/com/example/fipsdemo/FipsConfiguration.java`) at runtime. Consequently, the application does not need the provider classes at build-time.
 - There are no direct code references to any security provider. However, the application does register `OpenSSLFIPSProvider` using reflection in `src/main/java/com/example/fipsdemo/FipsCryptoService.java`


:::{important}
When running with OpenJDK FIPS installed through openjdk-{17,21}-fips-openssl-jre, the custom launcher pre-registers the OpenSSL FIPS provider. So, the reflection-based registering is optional.
:::

5. Compile the application using OpenJDK 17.

```{terminal}
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64/
mvn clean package
```

6. Run the application using OpenJDK OpenSSL FIPS v21.

```{terminal}
export $JAVA_HOME=/usr/lib/jvm/java-21-openjdk-fips-openssl-amd64/
$JAVA_HOME/bin/java -cp ./target/fips-crypto-demo-1.0.0-SNAPSHOT.jar com.example.fipsdemo.FipsDemoApplication
```

A successful run should print output like this:
```
=== FIPS Cryptography Demo ===
Provider : OpenSSLFIPSProvider
DRBG     : AES256CTR

-- Message Digests --
MDSHA256: 68aed7ce1c26cd5f4ff4615cb24492b9ff9468a692df508327332e18d7f71f87
MDSHA384: c0be3a368187599e8a255527875c7b0d62162cd3c31512a273c711a6ef4d26441f747f825ef950468f65eb3b7493ddc1

-- Symmetric Ciphers --
AES256/GCM/NONE
  ciphertext (b64): qG0NDXcCwcGWhI4LEL6VH1iu9+MuuVqXHOSNIdyCqt3quRwyUdCgmr21dEdq
  iv/nonce (b64)  : 3AI07wrrh7MNeTEm
  recovered       : FIPS-compliant secret payload
AES256/CBC/PKCS7
  ciphertext (b64): 33aHMxOFRTaBMqY1O8RuHru21ujln1t8WUzt5obA5bs=
  iv/nonce (b64)  : ppHtvUfc56KbRSpa6LRnCg==
  recovered       : FIPS-compliant secret payload

-- Message Authentication Code --
HMACwithSHA3_512: kIxHTeUnumAi44414xxbxbHBSlZ4OSmP7SRF74C9lmworKCw1vCN3Clu9/ug//noMn69nsmxtrZRjLXWXVNlgA==

-- Digital Signature --
RSAwithSHA256
  signature (b64): VaB8qCOhtpcsR3Csaclwf373AH+2Z61rT9+WwCwDZblJ0c/hNXfHiEAnrkNDc9zehxK9UwPvFTas7oNE8flP2I/iM9Ll7NDSVvIvn3se4bDDioSQsDcmYo9rUABKK7mQE+dehdxarRoZrzOrDGOMGtFgSmOXcLQY2I25GnF3lri1ZUSElBKzWGapvHookOfAM/aseeCbWznnWxqdItcy9rzOOtcF315Ea0OlnDf/LlWBEATNTsJTnIZxsCBfv4jyk3137hI4VdqKTuHDXbGnOLcVQmTHArhW3EtIzqPBBdMIXYDCKt9UfaAKQ41jATpJBfffLCrqRBkhyMZvTEdgKA==
  verified       : true

Provider registered at position: 12
```
:::{note}
We did not have to explicitly add the `OpenSSLFIPSProvider` classes (`openssl-fips-java.jar`) to the classpath. The OpenJDK OpenSSL FIPS launcher does that implicitly.
:::

## Further reading

Refer to {ref}`openjdk-openssl-fips` for an introduction to OpenSSL FIPS provider and the OpenJDK OpenSSL FIPS packages for Ubuntu Pro. 
