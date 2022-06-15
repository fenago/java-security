
MD5 Hashing in Java 
===================


**1. Overview** 
---------------------------------------------------------------------------




MD5 is a widely used cryptographic hash function, which produces a hash
of 128 bit.

In this article, we will see different approaches to **create MD5 hashes
using various Java libraries**.

**2. MD5 Using *MessageDigest* Class** 
---------------------------------------------------------


There is a `hashing` functionality
in *java.security.MessageDigest* class. The idea is to first instantiate
*MessageDigest* with the kind of algorithm you want to use as an
argument:

    MessageDigest.getInstance(String Algorithm)

And then keep on updating the message digest using *update()* function:

    public void update(byte [] input)

The above function can be called multiple times when say you are reading
a long file. Then finally we need to use *digest()* function to generate
a hash code:








    public byte[] digest()

Below is an example which generates a hash for a password and then
verifies it:

    @Test
    public void givenPassword_whenHashing_thenVerifying() 
      throws NoSuchAlgorithmException {
        String hash = "35454B055CC325EA1AF2126E27707052";
        String password = "ILoveJava";
            
        MessageDigest md = MessageDigest.getInstance("MD5");
        md.update(password.getBytes());
        byte[] digest = md.digest();
        String myHash = DatatypeConverter
          .printHexBinary(digest).toUpperCase();
            
        assertThat(myHash.equals(hash)).isTrue();
    }

Similarly, we can also verify checksum of a file:

    @Test
    public void givenFile_generatingChecksum_thenVerifying() 
      throws NoSuchAlgorithmException, IOException {
        String filename = "src/test/resources/test_md5.txt";
        String checksum = "5EB63BBBE01EEED093CB22BB8F5ACDC3";
            
        MessageDigest md = MessageDigest.getInstance("MD5");
        md.update(Files.readAllBytes(Paths.get(filename)));
        byte[] digest = md.digest();
        String myChecksum = DatatypeConverter
          .printHexBinary(digest).toUpperCase();
            
        assertThat(myChecksum.equals(checksum)).isTrue();
    }

We need to be aware, that the **MessageDigest is not thread-safe**.
Consequently, we should use a new instance for every thread.

**3. MD5 Using Apache Commons** 
---------------------------------------------




The *org.apache.commons.codec.digest.DigestUtils* class makes things
much simpler.

Let\'s see an example for hashing and verifying password:








    @Test
    public void givenPassword_whenHashingUsingCommons_thenVerifying()  {
        String hash = "35454B055CC325EA1AF2126E27707052";
        String password = "ILoveJava";

        String md5Hex = DigestUtils
          .md5Hex(password).toUpperCase();
            
        assertThat(md5Hex.equals(hash)).isTrue();
    }

**4. MD5 Using Guava** 
-----------------------------------------------------------------------------------------




Below is another approach we can follow to generate MD5 checksums using
*com.google.common.io.Files.hash* :

    @Test
    public void givenFile_whenChecksumUsingGuava_thenVerifying() 
      throws IOException {
        String filename = "src/test/resources/test_md5.txt";
        String checksum = "5EB63BBBE01EEED093CB22BB8F5ACDC3";
            
        HashCode hash = com.google.common.io.Files
          .hash(new File(filename), Hashing.md5());
        String myChecksum = hash.toString()
          .toUpperCase();
            
        assertThat(myChecksum.equals(checksum)).isTrue();
    }

Note, that *Hashing.md5* is deprecated. However, as the [official
documentation](https://guava.dev/releases/23.0/api/docs/com/google/common/hash/Hashing.html#md5--)
indicates, the reason is rather to advise not to use MD5 in general for
security concerns. This means we can still use this method if we, for
example, need to integrate with the legacy system that requires MD5.
Otherwise, we\'re better off considering safer options, like
`SHA-256`.

**5. Conclusion** 
-------------------------------------------------------------------------------




There are different ways in Java API and other third-party APIs like
Apache commons and Guava to generate the MD5 hash. Choose wisely based
on the requirements of the project and dependencies your project needs
to follow.

As always, the code is available [over on
Github](https://github.com/fenago/java-tutorials/tree/master/core-java-modules/core-java-security-2).

