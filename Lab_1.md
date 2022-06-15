
Encrypting and Decrypting Files in Java 
=======================================

1. Overview 
--------------------------------




In this tutorial, we\'ll take a look on how to encrypt and decrypt a
file using existing JDK APIs.

2. Writing a Test First 
------------------------------------------




We\'ll start by writing our test, TDD style. Since we\'re going to work
with files here, an integration test seems to be appropriate.

As we\'re just using existing JDK functionality, no external
dependencies are necessary.

First, **we\'ll encrypt the content using a newly generated secret key**
(we\'re using AES, [Advanced Encryption
Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), as
the symmetric encryption algorithm in this example).

Also note, that we\'re defining the complete transformation string in
the constructor (*AES/CBC/PKCS5Padding*), which is a concatenation of
used encryption, block cipher mode, and padding
(*algorithm/mode/padding*). JDK implementations support a number of
different transformations by default, but please note, that not every
combination can still be considered cryptographically secure by today\'s
standards.



We\'ll assume **our *FileEncrypterDecrypter* class will write the output
to a file called *baz.enc***. Afterward, **we decrypt this file using
the same secret key** and check that the decrypted content is equal to
the original content:

    @Test
    public void whenEncryptingIntoFile_andDecryptingFileAgain_thenOriginalStringIsReturned() {
        String originalContent = "foobar";
        SecretKey secretKey = KeyGenerator.getInstance("AES").generateKey();

        FileEncrypterDecrypter fileEncrypterDecrypter
          = new FileEncrypterDecrypter(secretKey, "AES/CBC/PKCS5Padding");
        fileEncrypterDecrypter.encrypt(originalContent, "baz.enc");

        String decryptedContent = fileEncrypterDecrypter.decrypt("baz.enc");
        assertThat(decryptedContent, is(originalContent));

        new File("baz.enc").delete(); // cleanup
    }

3. Encryption 
------------------------------------




We\'ll initialize the cipher in the constructor of
our *FileEncrypterDecrypter* class using the specified transformation
*String.*

This allows us to fail early in case a wrong transformation was
specified:

    FileEncrypterDecrypter(SecretKey secretKey, String transformation) {
        this.secretKey = secretKey;
        this.cipher = Cipher.getInstance(transformation);
    }

We can then **use the instantiated cipher and the provided secret key to
perform the encryption:**

    void encrypt(String content, String fileName) {
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        byte[] iv = cipher.getIV();

        try (FileOutputStream fileOut = new FileOutputStream(fileName);
          CipherOutputStream cipherOut = new CipherOutputStream(fileOut, cipher)) {
            fileOut.write(iv);
            cipherOut.write(content.getBytes());
        }
    }

Java allows us to **leverage the convenient *CipherOutputStream* class
for writing the encrypted content into another *OutputStream***.





Please note that we\'re writing the IV ([Initialization
Vector](https://en.wikipedia.org/wiki/Initialization_vector)) to the
beginning of the output file. In this example, the IV is automatically
generated when initializing the *Cipher*.

Using an IV is mandatory when using CBC mode, in order to randomize the
encrypted output. The IV is however not considered a secret, so it\'s
okay to write it at the beginning of the file.

4. Decryption 
------------------------------------




For decrypting we likewise have to read the IV first. Afterward, we can
initialize our cipher and decrypt the content.

Again we can make use of a special Java class, ***CipherInputStream*,
which transparently takes care of the actual decryption**:

    String decrypt(String fileName) {
        String content;

        try (FileInputStream fileIn = new FileInputStream(fileName)) {
            byte[] fileIv = new byte[16];
            fileIn.read(fileIv);
            cipher.init(Cipher.DECRYPT_MODE, secretKey, new IvParameterSpec(fileIv));

            try (
                    CipherInputStream cipherIn = new CipherInputStream(fileIn, cipher);
                    InputStreamReader inputReader = new InputStreamReader(cipherIn);
                    BufferedReader reader = new BufferedReader(inputReader)
                ) {

                StringBuilder sb = new StringBuilder();
                String line;
                while ((line = reader.readLine()) != null) {
                    sb.append(line);
                }
                content = sb.toString();
            }

        }
        return content;
    }

5. Conclusion 
------------------------------------




We\'ve seen we can perform basic encryption and decryption using
standard JDK classes, such as *Cipher*, *CipherOutputStream* and
*CipherInputStream*.











As usual, the complete code for this article is available in our [GitHub
repository](https://github.com/fenago/java-tutorials/tree/master/core-java-modules/core-java-security).

In addition, you can find a list of the Ciphers available in the JDK
[here](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/Cipher.html).

Finally, do note that the code examples here aren\'t meant as
production-grade code and the specifics of your system need to be
considered thoroughly when using them.
