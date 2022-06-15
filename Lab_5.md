
Checksums in Java 
=================


1. Overview 
-----------------------------------------------------------------------------




In this mini-article, we\'ll provide a brief explanation of what
checksums are and show how to use some of **Java\'s built-in features
for calculating checksums**.

2. Checksums and Common Algorithms 
---------------------------------------




Essentially, **a checksum is a minified representation of a binary
stream of data.**

Checksums are commonly used for network programming in order to check
that a complete message has been received. Upon receiving a new message,
the checksum can be recomputed and compared to the received checksum to
ensure that no bits have been lost. Additionally, they may also be
useful for file management, for instance, to compare files or to detect
changes.

**There are several common algorithms for creating checksums, such as
Adler32 and CRC32**. These algorithms work by converting a sequence of
data or bytes into a much smaller sequence of letters and numbers.  They
are designed such that any small change in the input will result in a
vastly different calculated checksum.

Let\'s take a look at Java\'s support for CRC32. Note that while CRC32
may be useful for checksums, it\'s not recommended for secure
operations, like hashing a password.





3. Checksum From a String or Byte Array 
-----------------------------------------------------------




The first thing we need to do is to obtain the input to the checksum
algorithm.

If we\'re starting with a *String*, we can use the *getBytes()* method
to get a byte array from a *String*:

    String test = "test";
    byte[] bytes = test.getBytes();

Next, we can calculate the checksum using the byte array:

    public static long getCRC32Checksum(byte[] bytes) {
        Checksum crc32 = new CRC32();
        crc32.update(bytes, 0, bytes.length);
        return crc32.getValue();
    }

Here, we are using Java\'s built-in
[*CRC32*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/zip/CRC32.html)
class. Once the class is instantiated, we use the *update *method to
update the *Checksum* instance with the bytes from the input.

Simply put, the *update *method replaces the bytes held by the *CRC32*
*Object* -- this helps with code re-use and negates the need to create
new instances of *Checksum. *The *CRC32* class provides a few overridden
methods to replace either the whole byte array or a few bytes within it.








Finally, after setting the bytes*,* we export the checksum with the
*getValue *method.

4. Checksum From an *InputStream* 
-------------------------------------------------------




When dealing **with larger data sets of binary data, the above approach
would not be very memory-efficient as every byte is loaded into
memory**.

When we have an *InputStream*, we may opt to use *CheckedInputStream* to
create our checksum**.** By using this approach, we can define how many
bytes are processed at any one time.

In this example, we process a given amount of bytes at the time until we
reach the end of the stream.

The checksum value is then available from the *CheckedInputStream*:








    public static long getChecksumCRC32(InputStream stream, int bufferSize) 
      throws IOException {
        CheckedInputStream checkedInputStream = new CheckedInputStream(stream, new CRC32());
        byte[] buffer = new byte[bufferSize];
        while (checkedInputStream.read(buffer, 0, buffer.length) >= 0) {}
        return checkedInputStream.getChecksum().getValue();
    }

5. Conclusion 
---------------------------------------------------------------------------------




In this tutorial, we look at how to generate checksums from byte arrays
and *InputStream*s using Java\'s CRC32 support.

As always, the code is available [over on
GitHub](https://github.com/fenago/java-tutorials/tree/master/core-java-modules/core-java-security-2).

