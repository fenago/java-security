
SHA-256 and SHA3-256 Hashing in Java 
====================================


**1. Overview** 
---------------------------------------------------------------------------------------




The SHA (Secure Hash Algorithm) is one of the popular cryptographic hash
functions. A cryptographic hash can be used to make a signature for a
text or a data file.

In this tutorial, let\'s have a look at how we can perform SHA-256 and
SHA3-256 hashing operations using various Java libraries.

The [SHA-256](https://en.wikipedia.org/wiki/SHA-2) algorithm generates
an almost unique, fixed-size 256-bit (32-byte) hash. This is a one-way
function, so the result cannot be decrypted back to the original value.

Currently, SHA-2 hashing is widely used, as it is considered the most
secure hashing algorithm in the cryptographic arena.

[SHA-3](https://en.wikipedia.org/wiki/SHA-3) is the latest secure
hashing standard after SHA-2. Compared to SHA-2, SHA-3 provides a
different approach to generate a unique one-way hash, and it can be much
faster on some hardware implementations. Similar to SHA-256, SHA3-256 is
the 256-bit fixed-length algorithm in SHA-3.











[NIST](https://csrc.nist.gov/projects/hash-functions) released SHA-3 in
2015, so there are not quite as many SHA-3 libraries as SHA-2 for the
time being. It\'s not until JDK 9 that SHA-3 algorithms were available
in the built-in default providers.

Now let\'s start with SHA-256.




**2. *MessageDigest* Class in Java**  
-----------------------------------------------------




Java provides inbuilt *MessageDigest* class for SHA-256 hashing:

    MessageDigest digest = MessageDigest.getInstance("SHA-256");
    byte[] encodedhash = digest.digest(
      originalString.getBytes(StandardCharsets.UTF_8));

However, here we have to use a custom byte to hex converter to get the
hashed value in hexadecimal:

    private static String bytesToHex(byte[] hash) {
        StringBuilder hexString = new StringBuilder(2 * hash.length);
        for (int i = 0; i < hash.length; i++) {
            String hex = Integer.toHexString(0xff & hash[i]);
            if(hex.length() == 1) {
                hexString.append('0');
            }
            hexString.append(hex);
        }
        return hexString.toString();
    }

We need to be aware that the **MessageDigest is not thread-safe.**
Consequently, we should use a new instance for every thread.











**3. Guava Library** 
-----------------------------------




The Google Guava library also provides a utility class for hashing.

First, let\'s define the dependency:

    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>31.0.1-jre</version>
    </dependency>

Next, here\'s how we can use Guava to hash a String:

    String sha256hex = Hashing.sha256()
      .hashString(originalString, StandardCharsets.UTF_8)
      .toString();

**4. Apache Commons Codecs** 
--------------------------------------------




Similarly, we can also use Apache Commons Codecs:

    <dependency>
        <groupId>commons-codec</groupId>
        <artifactId>commons-codec</artifactId>
        <version>1.11</version>
    </dependency>

Here\'s the utility class --- called *DigestUtils* --- that supports
SHA-256 hashing:











    String sha256hex = DigestUtils.sha256Hex(originalString);

**5. Bouncy Castle Library** 
-------------------------------------------




### **5.1. Maven Dependency** 




    <dependency>
        <groupId>org.bouncycastle</groupId>
        <artifactId>bcprov-jdk15on</artifactId>
        <version>1.60</version>
    </dependency>

### **5.2. Hashing Using the Bouncy Castle Library** 




The Bouncy Castle API provides a utility class for converting hex data
to bytes and back again.

However, we need to populate a digest using the built-in Java API first:

    MessageDigest digest = MessageDigest.getInstance("SHA-256");
    byte[] hash = digest.digest(
      originalString.getBytes(StandardCharsets.UTF_8));
    String sha256hex = new String(Hex.encode(hash));

**6. SHA3-256** 
---------------------------------------------------------------------------------------




Now let\'s continue with SHA3-256. SHA3-256 hashing in Java isn\'t that
different from SHA-256.

### **6.1. *MessageDigest* Class in Java** 




[Starting from JDK
9](https://docs.oracle.com/javase/9/security/oracleproviders.htm#JSSEC-GUID-3A80CC46-91E1-4E47-AC51-CB7B782CEA7D),
we can simply use the built-in SHA3-256 algorithm:

    final MessageDigest digest = MessageDigest.getInstance("SHA3-256");
    final byte[] hashbytes = digest.digest(
      originalString.getBytes(StandardCharsets.UTF_8));
    String sha3Hex = bytesToHex(hashbytes);

### **6.2. Apache Commons Codecs** 




Apache Commons Codecs provides a convenient *DigestUtils* wrapper for
the *MessageDigest* class.

This library began to support SHA3-256 since version
[1.11](https://search.maven.org/artifact/commons-codec/commons-codec/1.11/jar),
and it [requires JDK
9+](https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/digest/MessageDigestAlgorithms.html#SHA3_256)
as well:

    String sha3Hex = new DigestUtils("SHA3-256").digestAsHex(originalString);

### **6.3. Keccak-256** 




Keccak-256 is another popular SHA3-256 hashing algorithm. Currently, it
serves as an alternative to the standard SHA3-256. Keccak-256 delivers
the same security level as the standard SHA3-256, and it differs from
SHA3-256 only on the padding rule. It has been used in several
blockchain projects, such as
[Monero](https://monerodocs.org/cryptography/keccak-256/).

Again, we need to import the Bouncy Castle Library to use Keccak-256
hashing:











    Security.addProvider(new BouncyCastleProvider());
    final MessageDigest digest = MessageDigest.getInstance("Keccak-256");
    final byte[] encodedhash = digest.digest(
      originalString.getBytes(StandardCharsets.UTF_8));
    String sha3Hex = bytesToHex(encodedhash);

We can also make use of the Bouncy Castle API to do the hashing:

    Keccak.Digest256 digest256 = new Keccak.Digest256();
    byte[] hashbytes = digest256.digest(
      originalString.getBytes(StandardCharsets.UTF_8));
    String sha3Hex = new String(Hex.encode(hashbytes));

**7. Conclusion** 
-------------------------------------------------------------------------------------------




In this quick article, we had a look at a few ways of implementing
SHA-256 and SHA3-256 hashing in Java, using both built-in and
third-party libraries.

The source code of the examples can be found in the [GitHub
project](https://github.com/fenago/java-tutorials/tree/master/core-java-modules/core-java-security-2).

