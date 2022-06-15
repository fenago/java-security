
Get a List of Trusted Certificates in Java 
==========================================


1. Overview 
----------------------------




In this quick tutorial, we\'ll learn how to read a list of trusted
certificates in Java through quick and practical examples.

2. Loading the *KeyStore* 
---------------------------------------------------------




Java stores the trusted certificates in a special file named *cacerts*
that lives inside our Java installation folder.

Let\'s start by reading this file and loading it into the
*KeyStore*:

    private KeyStore loadKeyStore() {
        String relativeCacertsPath = "/lib/security/cacerts".replace("/", File.separator);
        String filename = System.getProperty("java.home") + relativeCacertsPath;
        FileInputStream is = new FileInputStream(filename);

        KeyStore keystore = KeyStore.getInstance(KeyStore.getDefaultType());
        String password = "changeit";
        keystore.load(is, password.toCharArray());

        return keystore;
    }

**The default password for this *KeyStore* is *"changeit"*, but it could
be different if it was previously changed in our system.**

Once loaded, the *KeyStore* will hold our trusted certificates, and
next, we\'ll see how to read them.











3. Reading Certificates From a Specified *KeyStore* 
--------------------------------------------




We\'re going to use the
*[PKIXParameters](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/cert/PKIXParameters.html)* class,
which takes a *KeyStore* as a constructor parameter:

    @Test
    public void whenLoadingCacertsKeyStore_thenCertificatesArePresent() {
        KeyStore keyStore = loadKeyStore();
        PKIXParameters params = new PKIXParameters(keyStore);

        Set<TrustAnchor> trustAnchors = params.getTrustAnchors();
        List<Certificate> certificates = trustAnchors.stream()
          .map(TrustAnchor::getTrustedCert)
          .collect(Collectors.toList());

        assertFalse(certificates.isEmpty());
    }

The *PKIXParameters* class is usually used for validating a certificate,
but in our example, we simply used it to exact the certificates from our
*KeyStore*.

When creating an instance of *PKIXParametrs*, it builds a list of
*[TrustAnchor](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/cert/TrustAnchor.html)* that
will contain the trusted certificates present in our *KeyStore*.

A *TrustAnchor* instance simply represents a trusted certificate.

4. Reading Certificates From Default *KeyStore* 
------------------------------------




We can also get a list of the trusted certificates present in our system
by **using the *TrustManagerFactory* class and initializing it without a
*KeyStore***, which will use the default *KeyStore*.











If we don\'t provide a *KeyStore* explicitly, the same one from the
previous chapter will be used by default:

    @Test
    public void whenLoadingDefaultKeyStore_thenCertificatesArePresent() {
        TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        trustManagerFactory.init((KeyStore) null);

        List<TrustManager> trustManagers = Arrays.asList(trustManagerFactory.getTrustManagers());
        List<X509Certificate> certificates = trustManagers.stream()
          .filter(X509TrustManager.class::isInstance)
          .map(X509TrustManager.class::cast)
          .map(trustManager -> Arrays.asList(trustManager.getAcceptedIssuers()))
          .flatMap(Collection::stream)
          .collect(Collectors.toList());

        assertFalse(certificates.isEmpty());
    }

In the above example, we've used X509TrustManager, which is a specialized TrustManager used to authenticate the remote part of an SSL connection.

Note that this behavior may depend on the specific JDK implementation,
as [the
specification](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/net/ssl/TrustManagerFactory.html#init(java.security.KeyStore))
doesn\'t define what should happen in case the *init()* *KeyStore*
parameter is *null*.

5. Certificate Aliases 
-----------------------------------------------------




A certificate alias is simply a *String* that uniquely identifies a
certificate.

**Among the default certificates imported by Java, there\'s also a
well-known certificate issued by GoDaddy, a public Internet domain
registrar, which we\'ll use in our tests:**











    String GODADDY_CA_ALIAS = "godaddyrootg2ca [jdk]";

Let\'s see how we can read all certificate aliases present in our
*KeyStore*:

    @Test
    public void whenLoadingKeyStore_thenGoDaddyCALabelIsPresent() {
        KeyStore keyStore = loadKeyStore();

        Enumeration<String> aliasEnumeration = keyStore.aliases();
        List<String> aliases = Collections.list(aliasEnumeration);
        assertTrue(aliases.contains(GODADDY_CA_ALIAS));
    }

In the next example, we\'ll see how we can retrieve a certificate by its
alias:

    @Test
    public void whenLoadingKeyStore_thenGoDaddyCertificateIsPresent() {
        KeyStore keyStore = loadKeyStore();

        Certificate goDaddyCertificate = keyStore.getCertificate(GODADDY_CA_ALIAS);
        assertNotNull(goDaddyCertificate);
    }

6. Conclusion 
-----------------------------------




**In this quick article, we\'ve looked at different ways of listing
trusted certificates in Java through quick and practical examples.**

As always, code snippets can be found [over on
GitHub](https://github.com/fenago/java-tutorials/tree/master/core-java-modules/core-java-security-2).

