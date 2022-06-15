
Hashing a Password in Java 
==========================

**1. Overview** 
----------------------------------------------------------------------------------------




In this tutorial, we\'ll be discussing the importance of password
hashing.

We\'ll take a quick look at what it is, why it\'s important, and some
secure and insecure ways of doing it in Java.

**2. What\'s Hashing?** 
---------------------------------------

`Hashing` is the process of
generating a string, or *hash*, from a given *message* using a
mathematical function known as a *cryptographic hash function*.

While there are several hash functions out there, those tailored to
hashing passwords need to have four main properties to be secure:

1.  It should be *deterministic*: the same message processed by the same
    hash function should *always *produce the same *hash*
2.  It\'s not *reversible*: it\'s impractical to generate a *message*
    from its *hash*
3.  It has high *entropy*: a
    small change to a *message* should produce a vastly different *hash*
4.  And it resists *collisions*: two different *messages* should not
    produce the same *hash*

A hash function that has all four properties is a strong candidate for
password hashing since together they dramatically increase the
difficulty in reverse-engineering the password from the hash.











Also, though, **password hashing functions should be slow**. A fast
algorithm would aid `*brute force *attacks`
in which a hacker will attempt to guess a password by hashing and
comparing billions ([or
trillions](https://www.wired.com/2014/10/snowdens-first-emails-to-poitras/))
of potential passwords per second.

Some great hash functions that meet all these criteria
are** ***PBKDF2, BCrypt, *and *SCrypt. *But first, let\'s take a look at
some older algorithms and why they are no longer recommended

**3. Not Recommended: MD5** 
-------------------------------------------------




Our first hash function is the MD5 message-digest algorithm, developed
way back in 1992.

Java\'s *MessageDigest* makes this easy to calculate and can still be
useful in other circumstances.

However, over the last several years, **MD5 was discovered to [fail the
fourth password hashing
property](https://blog.avira.com/md5-the-broken-algorithm/) **in that it
became computationally easy to generate collisions. To top it off, MD5
is a fast algorithm and therefore useless against brute-force attacks.











**Because of these, MD5 is not recommended.**

**4. Not Recommended: SHA-512** 
---------------------------------------------------------




Next, we\'ll look at SHA-512, which is part of the Secure Hash Algorithm
family, a family that began with SHA-0 back in 1993.

### **4.1. Why SHA-512?** 




As computers increase in power, and as we find new vulnerabilities, then
researchers derive new versions of SHA. Newer versions have a
progressively longer length, or sometimes researchers publish a new
version of the underlying algorithm.

SHA-512 represents the longest key in the third generation of the
algorithm.

While **there are now more secure versions of SHA**, [SHA-512 is the
strongest that is implemented in
Java](https://docs.oracle.com/en/java/javase/12/docs/specs/security/standard-names.html).











### **4.2. Implementing in Java** 




Now, let\'s have a look at implementing the SHA-512 hashing algorithm in
Java.

First, we have to understand the concept of *salt*. Simply put, **this
is a random sequence that is generated for each new hash**.

By introducing this randomness, we increase the hash\'s *entropy*, and
we protect our database against pre-compiled lists of hashes known as
*rainbow tables*.

Our new hash function then becomes roughly:

    salt <- generate-salt;
    hash <- salt + ':' + sha512(salt + password)

### **4.3. Generating a Salt** 




To introduce salt, we\'ll use
the *[SecureRandom](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/SecureRandom.html) *class
from *java.security*:

    SecureRandom random = new SecureRandom();
    byte[] salt = new byte[16];
    random.nextBytes(salt);

Then, we\'ll use
the *[MessageDigest](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/MessageDigest.html) *class
to configure the *SHA-512 *hash function with our salt:

    MessageDigest md = MessageDigest.getInstance("SHA-512");
    md.update(salt);

And with that added, we can now use the *digest* method to generate our
hashed password:

    byte[] hashedPassword = md.digest(passwordToHash.getBytes(StandardCharsets.UTF_8));

### **4.4. Why Is It Not Recommended?** 




When employed with salt, [SHA-512 is still a fair
option](https://en.wikipedia.org/wiki/Secure_Hash_Algorithms), **but
there are stronger and slower options out there**.











Also, the remaining options we\'ll cover have an important feature:
configurable strength.

**5. PBKDF2, BCrypt, and SCrypt** 
------------------------------------------------------------




PBKDF2, BCrypt, and SCrypt are three recommended algorithms.

### **5.1. Why Are Those Recommended?** 




Each of these is slow, and each has the brilliant feature of having a
configurable strength.

This means that as computers increase in strength, **we can slow down
the algorithm by changing the inputs.**

### **5.2. Implementing PBKDF2 in Java** 




Now, **salts are a fundamental principle of password hashing**, and so
we need one for PBKDF2, too:

    SecureRandom random = new SecureRandom();
    byte[] salt = new byte[16];
    random.nextBytes(salt);

Next, we\'ll create
a [*PBEKeySpec*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/spec/PBEKeySpec.html) and
a
*[SecretKeyFactory](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/SecretKeyFactory.html)* which
we\'ll instantiate using the *PBKDF2WithHmacSHA1 *algorithm:

    KeySpec spec = new PBEKeySpec(password.toCharArray(), salt, 65536, 128);
    SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");

The third parameter (*65536*) is effectively the strength parameter. It
indicates how many iterations that this algorithm run for, increasing
the time it takes to produce the hash.

Finally, we can use our *SecretKeyFactory *to generate the hash:











    byte[] hash = factory.generateSecret(spec).getEncoded();

### **5.3. Implementing BCrypt and SCrypt in Java** 




So, it turns out that **BCrypt and SCrypt support don\'t yet ship with
Java**, though some Java libraries support them.

One of those libraries is Spring Security.

**6. Password Hashing With Spring Security** 
------------------------------------------------------------------------------------




Although Java natively supports both the PBKDF2 and SHA hashing
algorithms, it doesn\'t support BCrypt and SCrypt algorithms.

Luckily for us, Spring Security ships with support for all these
recommended algorithms via
the [*PasswordEncoder*](https://docs.spring.io/spring-security/site/docs/4.2.4.RELEASE/apidocs/org/springframework/security/crypto/password/PasswordEncoder.html)
interface:

-   *Pbkdf2PasswordEncoder* gives us PBKDF2
-   *BCryptPasswordEncoder *gives us BCrypt, and
-   *SCryptPasswordEncoder *gives us SCrypt

**The password encoders for PBKDF2, BCrypt, and SCrypt all come with
support for configuring the desired strength of the password hash.**

We can use these encoders directly, even without having a Spring
Security-based application. Or, if we are protecting our site with
Spring Security, then we can configure our desired password encoder
through its DSL or via dependency injection.

And, unlike our examples above, **these encryption algorithms will
generate the salt for us internally**. The algorithm stores the salt
within the output hash for later use in validating a password.

**7. Conclusion** 
--------------------------------------------------------------------------------------------




So, we\'ve taken a deep dive into password hashing; exploring the
concept and its uses.











And we\'ve taken a look at some historical hash functions as well as
some currently implemented ones before coding them in Java.

Finally, we saw that Spring Security ships with its password encrypting
classes, implementing an array of different hash functions.

As always, the code is [available on
GitHub.](https://github.com/fenago/java-tutorials/tree/master/core-java-modules/core-java-security-2)

