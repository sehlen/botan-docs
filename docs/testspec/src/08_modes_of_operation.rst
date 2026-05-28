Modes of Operation
==================

Block cipher modes of operation are tested using known answer tests that
(1) encrypt a message and (2) decrypt a message. All the tests are
implemented in :srcref:`src/tests/test_modes.cpp`. The test cases are described
in the following.

.. table::
   :class: longtable
   :widths: 20 80

   +---------------------+----------------------------------------------------------------------------+
   | **Test Case No.:**  | MODE-1                                                                     |
   +=====================+============================================================================+
   | **Type:**           | Positive Test                                                              |
   +---------------------+----------------------------------------------------------------------------+
   | **Description:**    | Known Answer Test that verifies the correctness of encryption under the    |
   |                     | mode of operation                                                          |
   +---------------------+----------------------------------------------------------------------------+
   | **Preconditions:**  | None                                                                       |
   +---------------------+----------------------------------------------------------------------------+
   | **Input Values:**   | -  Block Cipher: The underlying block cipher, e.g., AES-128 or AES-256     |
   |                     |                                                                            |
   |                     | -  Key: The encryption/decryption key used for the block cipher (varying   |
   |                     |    length depending on the block cipher)                                   |
   |                     |                                                                            |
   |                     | -  Nonce: The nonce used to initialize the mode of operation (varying      |
   |                     |    length)                                                                 |
   |                     |                                                                            |
   |                     | -  In: The test message to be encrypted (varying length)                   |
   +---------------------+----------------------------------------------------------------------------+
   | **Expected          | -  Out: Ciphertext (varying length depending on the block cipher)          |
   | Output:**           |                                                                            |
   +---------------------+----------------------------------------------------------------------------+
   | **Steps:**          | #. Create a Cipher_Mode encryption object                                  |
   |                     |                                                                            |
   |                     | #. Test the name of the mode                                               |
   |                     |                                                                            |
   |                     | #. Test that the mode is not an authenticated mode                         |
   |                     |                                                                            |
   |                     | #. Test that calculating the ciphertext before setting a key throws an     |
   |                     |    exception                                                               |
   |                     |                                                                            |
   |                     | #. Test that large nonce sizes are rejected by throwing an exception       |
   |                     |                                                                            |
   |                     | #. Set the key *Key* on the Cipher_Mode encryption object                  |
   |                     |                                                                            |
   |                     | #. Set the nonce *Nonce* on the Cipher_Mode encryption object              |
   |                     |                                                                            |
   |                     | #. Calculate the ciphertext of input value *In* and compare the result     |
   |                     |    with the expected output value *Out*                                    |
   |                     |                                                                            |
   |                     | #. If *In* is longer than the block size of the mode, calculate the        |
   |                     |    ciphertext of input value *In* by encrypting *In* in block size blocks  |
   |                     |    and comparing the result with the expected output value *Out*           |
   |                     |                                                                            |
   |                     | #. If *In* is longer than the block size of the mode, calculate the        |
   |                     |    ciphertext of input value *In* by encrypting *In* in multiples of block |
   |                     |    size blocks and comparing the result with the expected output value     |
   |                     |    *Out*                                                                   |
   |                     |                                                                            |
   |                     | #. Clear the Cipher_Mode encryption object                                 |
   |                     |                                                                            |
   |                     | #. Test that calculating the ciphertext after clearing throws an exception |
   +---------------------+----------------------------------------------------------------------------+

.. table::
   :class: longtable
   :widths: 20 80

   +---------------------+----------------------------------------------------------------------------+
   | **Test Case No.:**  | MODE-2                                                                     |
   +=====================+============================================================================+
   | **Type:**           | Positive Test                                                              |
   +---------------------+----------------------------------------------------------------------------+
   | **Description:**    | Known Answer Test that verifies the correctness of decryption under the    |
   |                     | mode of operation                                                          |
   +---------------------+----------------------------------------------------------------------------+
   | **Preconditions:**  | None                                                                       |
   +---------------------+----------------------------------------------------------------------------+
   | **Input Values:**   | -  Block Cipher: The underlying block cipher, e.g., AES-128 or AES-256     |
   |                     |                                                                            |
   |                     | -  Key: The encryption/decryption key used for the block cipher (varying   |
   |                     |    length depending on the block cipher)                                   |
   |                     |                                                                            |
   |                     | -  Nonce: The nonce used to initialize the mode of operation (varying      |
   |                     |    length)                                                                 |
   |                     |                                                                            |
   |                     | -  Out: Ciphertext (varying length depending on the block cipher)          |
   +---------------------+----------------------------------------------------------------------------+
   | **Expected          | -  In: The original plaintext (varying length)                             |
   | Output:**           |                                                                            |
   +---------------------+----------------------------------------------------------------------------+
   | **Steps:**          | #. Create a Cipher_Mode decryption object                                  |
   |                     |                                                                            |
   |                     | #. Test the name of the mode                                               |
   |                     |                                                                            |
   |                     | #. Test that the mode is not an authenticated mode                         |
   |                     |                                                                            |
   |                     | #. Test that calculating the ciphertext before setting a key throws an     |
   |                     |    exception                                                               |
   |                     |                                                                            |
   |                     | #. Test that large nonce sizes are rejected by throwing an exception       |
   |                     |                                                                            |
   |                     | #. Set the key *Key* on the Cipher_Mode encryption object                  |
   |                     |                                                                            |
   |                     | #. Set the nonce *Nonce* on the Cipher_Mode decryption object              |
   |                     |                                                                            |
   |                     | #. Calculate the plaintext of output value *In* and compare the result     |
   |                     |    with the output value *In*                                              |
   |                     |                                                                            |
   |                     | #. If *Out* is longer than the block size of the mode, calculate the       |
   |                     |    plaintext of input value *Out* by decrypting *Out* in block size blocks |
   |                     |    and comparing the result with the expected output value *In*            |
   |                     |                                                                            |
   |                     | #. If *Out* is longer than the block size of the mode, calculate the       |
   |                     |    plaintext of input value *Out* by decrypting *Out* in multiples of      |
   |                     |    block size blocks and comparing the result with the expected output     |
   |                     |    value *In*                                                              |
   |                     |                                                                            |
   |                     | #. Clear the Cipher_Mode decryption object                                 |
   |                     |                                                                            |
   |                     | #. Test that calculating the plaintext after clearing throws an exception  |
   +---------------------+----------------------------------------------------------------------------+

CBC
---

CBC is tested with the following constraints:

-  Number of test cases: 3

-  Block Cipher: AES-128, AES-192 and AES-256
-  Key: 128 bits, 192 and 256 bits
-  Nonce: 128 bits
-  In: 512 bits
-  Out: 512 bits

The following table shows an example test case with one test vector. All
test vectors are listed in :srcref:`src/tests/data/modes/cbc.vec`.

.. table::
   :class: longtable
   :widths: 20 80

   +---------------------+----------------------------------------------------------------------------+
   | **Test Case No.:**  | MODE-CBC-1                                                                 |
   +=====================+============================================================================+
   | **Type:**           | Positive Test                                                              |
   +---------------------+----------------------------------------------------------------------------+
   | **Description:**    | Known Answer Test that verifies the correctness of encryption under CBC    |
   +---------------------+----------------------------------------------------------------------------+
   | **Preconditions:**  | None                                                                       |
   +---------------------+----------------------------------------------------------------------------+
   | **Input Values:**   | .. code-block:: none                                                       |
   |                     |                                                                            |
   |                     |    Block Cipher = AES-128                                                  |
   |                     |    Key = 0x2B7E151628AED2A6ABF7158809CF4F3C (128 bits)                     |
   |                     |    Nonce = 0x000102030405060708090A0B0C0D0E0F (128 bits)                   |
   |                     |    In = 0x6BC1BEE22E409F96E93D7E117393172AAE2D8A571E03AC9C9EB76FAC45AF8E51 |
   |                     |    30C81C46A35CE411E5FBC1191A0A52EFF69F2445DF4F9B17AD2B417BE66C3710        |
   |                     |    (512 bits)                                                              |
   +---------------------+----------------------------------------------------------------------------+
   | **Expected          | .. code-block:: none                                                       |
   | Output:**           |                                                                            |
   |                     |    Out = 0x7649ABAC8119B246CEE98E9B12E9197D5086CB9B507219EE95DB113A917678B |
   |                     |    273BED6B8E3C1743B7116E69E222295163FF1CAA1681FAC09120ECA307586E1A7       |
   |                     |    (512 bits)                                                              |
   +---------------------+----------------------------------------------------------------------------+
   | **Steps:**          | #. Create a CBC_Encryption object                                          |
   |                     |                                                                            |
   |                     | #. Test the name of the mode                                               |
   |                     |                                                                            |
   |                     | #. Test that the mode is not an authenticated mode                         |
   |                     |                                                                            |
   |                     | #. Test that calculating the ciphertext before setting a key throws an     |
   |                     |    exception                                                               |
   |                     |                                                                            |
   |                     | #. Test that large nonce sizes are rejected by throwing an exception       |
   |                     |                                                                            |
   |                     | #. Set the key *Key* on the CBC_Encryption object                          |
   |                     |                                                                            |
   |                     | #. Set the nonce *Nonce* on the CBC_Encryption object                      |
   |                     |                                                                            |
   |                     | #. Calculate the ciphertext of input value *In* and compare the result     |
   |                     |    with the expected output value *Out*                                    |
   |                     |                                                                            |
   |                     | #. Clear the CBC_Encryption object                                         |
   |                     |                                                                            |
   |                     | #. Test that calculating the ciphertext after clearing throws an exception |
   +---------------------+----------------------------------------------------------------------------+

CBC-CTS (CBC-CS3)
-----------------

CBC-CTS is tested with the following constraints:

-  Number of test cases: 6
-  Source: RFC 3962

-  Block Cipher: AES-128
-  Key: 128 bits
-  Nonce: 128 bits
-  In: 136 bits, 248 bits, 256 bits, 376 bits, 384 bits, 512 bits
-  Out: 136 bits, 248 bits, 256 bits, 376 bits, 384 bits, 512 bits

The following table shows an example test case with one test vector. All
test vectors are listed in :srcref:`src/tests/data/modes/cbc.vec`.

.. table::
   :class: longtable
   :widths: 20 80

   +---------------------+----------------------------------------------------------------------------+
   | **Test Case No.:**  | MODE-CTS-1                                                                 |
   +=====================+============================================================================+
   | **Type:**           | Positive Test                                                              |
   +---------------------+----------------------------------------------------------------------------+
   | **Description:**    | Known Answer Test that verifies the correctness of encryption under CTS    |
   +---------------------+----------------------------------------------------------------------------+
   | **Preconditions:**  | None                                                                       |
   +---------------------+----------------------------------------------------------------------------+
   | **Input Values:**   | Block Cipher = AES-128                                                     |
   |                     |                                                                            |
   |                     | Key = 0x636869636b656e207465726979616b69 (128 bits)                        |
   |                     |                                                                            |
   |                     | Nonce = 0x00000000000000000000000000000000 (128 bits)                      |
   |                     |                                                                            |
   |                     | In = 0x4920776f756c64206c696b652074686520 (136 bits)                       |
   +---------------------+----------------------------------------------------------------------------+
   | **Expected          | Out = 0xc6353568f2bf8cb4d8a580362da7ff7f97 (136 bits)                      |
   | Output:**           |                                                                            |
   +---------------------+----------------------------------------------------------------------------+
   | **Steps:**          | #. Create a CTS_Encryption object                                          |
   |                     |                                                                            |
   |                     | #. Test the name of the mode                                               |
   |                     |                                                                            |
   |                     | #. Test that the mode is not an authenticated mode                         |
   |                     |                                                                            |
   |                     | #. Test that calculating the ciphertext before setting a key throws an     |
   |                     |    exception                                                               |
   |                     |                                                                            |
   |                     | #. Test that large nonce sizes are rejected by throwing an exception       |
   |                     |                                                                            |
   |                     | #. Set the key *Key* on the CTS_Encryption object                          |
   |                     |                                                                            |
   |                     | #. Set the nonce *Nonce* on the CTS_Encryption object                      |
   |                     |                                                                            |
   |                     | #. Calculate the ciphertext of input value *In* and compare the result     |
   |                     |    with the expected output value *Out*                                    |
   |                     |                                                                            |
   |                     | #. Clear the CTS_Encryption object                                         |
   |                     |                                                                            |
   |                     | #. Test that calculating the ciphertext after clearing throws an exception |
   +---------------------+----------------------------------------------------------------------------+

CTR
---

CTR mode is a stream cipher mode of operation in the library and thus is
tested differently than other block cipher modes of operation. All the
stream cipher modes of operation tests are implemented in
:srcref:`src/tests/test_stream.cpp`. CTR mode is tested with the following
constraints:

-  Number of test cases: 6

-  Block Cipher: AES-128, AES-192, AES-256
-  Key: 128 bits, 192 bits, 256 bits
-  Nonce: 128 bits
-  In: 384 bits, 512 bits, 5720 bits, 65536 bits
-  Out: 384 bits, 512 bits, 5720 bits, 65536 bits

The following table shows an example test case with one test vector. All
test vectors are listed in :srcref:`src/tests/data/stream/ctr.vec`.

.. table::
   :class: longtable
   :widths: 20 80

   +----------------------+--------------------------------------------------------------------------+
   | **Test Case No.:**   | MODE-CTR-1                                                               |
   +======================+==========================================================================+
   | **Type:**            | Positive Test                                                            |
   +----------------------+--------------------------------------------------------------------------+
   | **Description:**     | Known Answer Test that verifies the correctness of encryption under CTR  |
   +----------------------+--------------------------------------------------------------------------+
   | **Preconditions:**   | None                                                                     |
   +----------------------+--------------------------------------------------------------------------+
   | **Input Values:**    | .. code-block:: none                                                     |
   |                      |                                                                          |
   |                      |    Block Cipher = AES-128                                                |
   |                      |    Key = 0x2B7E151628AED2A6ABF7158809CF4F3C (128 bits)                   |
   |                      |    Nonce = 0xF0F1F2F3F4F5F6F7F8F9FAFBFCFDFEFF (128 bits)                 |
   |                      |    In = 0x6BC1BEE22E409F96E93D7E117393172AAE2D8A571E03AC9C9EB76FAC45AF8E |
   |                      |    5130C81C46A35CE411E5FBC1191A0A52EFF69F2445DF4F9B17AD2B417BE66C3710    |
   |                      |    (384 bits)                                                            |
   +----------------------+--------------------------------------------------------------------------+
   | **Expected Output:** | .. code-block:: none                                                     |
   |                      |                                                                          |
   |                      |    Out = 0x874D6191B620E3261BEF6864990DB6CE9806F66B7970FDFF8617187BB9FFF |
   |                      |    DFF5AE4DF3EDBD5D35E5B4F09020DB03EAB1E031DDA2FBE03D1792170A0F3009CEE   |
   |                      |    (384 bits)                                                            |
   +----------------------+--------------------------------------------------------------------------+
   | **Steps:**           | #. Create a StreamCipher object                                          |
   |                      |                                                                          |
   |                      | #. Test the name of the mode                                             |
   |                      |                                                                          |
   |                      | #. Set the key *Key* on the StreamCipher object                          |
   |                      |                                                                          |
   |                      | #. Set the IV *Nonce* on the StreamCipher object                         |
   |                      |                                                                          |
   |                      | #. Clone the StreamCipher object and check that it has a different       |
   |                      |    pointer but the same name                                             |
   |                      |                                                                          |
   |                      | #. Set a random key on the cloned StreamCipher object                    |
   |                      |                                                                          |
   |                      | #. Calculate the ciphertext of input value *In* on the original          |
   |                      |    StreamCipher object and compare the result with the expected output   |
   |                      |    value *Out*                                                           |
   +----------------------+--------------------------------------------------------------------------+

.. table::
   :class: longtable
   :widths: 20 80

   +----------------------+--------------------------------------------------------------------------------+
   | **Test Case No.:**   | MODE-CFB-1                                                                     |
   +======================+================================================================================+
   | **Type:**            | Positive vector test                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Description:**     | Runs CFB encrypt/decrypt vectors                                               |
   +----------------------+--------------------------------------------------------------------------------+
   | **Preconditions:**   | None                                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Input Values:**    | (CFB mode vectors)                                                             |
   +----------------------+--------------------------------------------------------------------------------+
   | **Expected Output:** | Ciphertext/plaintext results match expected vectors                            |
   +----------------------+--------------------------------------------------------------------------------+
   | **Steps:**           | #. Execute the Botan 3.11 test case for MODE-CFB-1 (CFB mode vectors)          |
   |                      |                                                                                |
   |                      | #. Verify the observed behavior matches the expected output.                   |
   +----------------------+--------------------------------------------------------------------------------+


.. table::
   :class: longtable
   :widths: 20 80

   +----------------------+--------------------------------------------------------------------------------+
   | **Test Case No.:**   | MODE-CTR-CM-1                                                                  |
   +======================+================================================================================+
   | **Type:**            | Positive vector test                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Description:**     | Covers CTR mode vectors exercised by cipher-mode unit tests                    |
   +----------------------+--------------------------------------------------------------------------------+
   | **Preconditions:**   | None                                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Input Values:**    | (CTR vectors from ``test_modes.cpp`` cipher-mode tests)                        |
   +----------------------+--------------------------------------------------------------------------------+
   | **Expected Output:** | Counter mode output matches expected vector values                             |
   +----------------------+--------------------------------------------------------------------------------+
   | **Steps:**           | #. Execute the Botan 3.11 test case for MODE-CTR-CM-1 (CTR vectors from        |
   |                      | ``test_modes.cpp`` cipher-mode tests)                                          |
   |                      |                                                                                |
   |                      | #. Verify the observed behavior matches the expected output.                   |
   +----------------------+--------------------------------------------------------------------------------+


.. table::
   :class: longtable
   :widths: 20 80

   +----------------------+--------------------------------------------------------------------------------+
   | **Test Case No.:**   | MODE-IV-CARRY-1                                                                |
   +======================+================================================================================+
   | **Type:**            | Positive regression test                                                       |
   +----------------------+--------------------------------------------------------------------------------+
   | **Description:**     | Validates IV carry/update behavior across chunked processing                   |
   +----------------------+--------------------------------------------------------------------------------+
   | **Preconditions:**   | None                                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Input Values:**    | (`Cipher_Mode_IV_Carry_Tests`)                                                 |
   +----------------------+--------------------------------------------------------------------------------+
   | **Expected Output:** | Stateful updates produce the same result as equivalent single-shot processing  |
   +----------------------+--------------------------------------------------------------------------------+
   | **Steps:**           | #. Execute the Botan 3.11 test case for MODE-IV-CARRY-1                        |
   |                      | (`Cipher_Mode_IV_Carry_Tests`)                                                 |
   |                      |                                                                                |
   |                      | #. Verify the observed behavior matches the expected output.                   |
   +----------------------+--------------------------------------------------------------------------------+


.. table::
   :class: longtable
   :widths: 20 80

   +----------------------+--------------------------------------------------------------------------------+
   | **Test Case No.:**   | MODE-XTS-1                                                                     |
   +======================+================================================================================+
   | **Type:**            | Positive vector test                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Description:**     | Covers XTS mode data-unit encryption/decryption vectors                        |
   +----------------------+--------------------------------------------------------------------------------+
   | **Preconditions:**   | None                                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Input Values:**    | (XTS mode vectors)                                                             |
   +----------------------+--------------------------------------------------------------------------------+
   | **Expected Output:** | XTS outputs match the known-answer vectors                                     |
   +----------------------+--------------------------------------------------------------------------------+
   | **Steps:**           | #. Execute the Botan 3.11 test case for MODE-XTS-1 (XTS mode vectors)          |
   |                      |                                                                                |
   |                      | #. Verify the observed behavior matches the expected output.                   |
   +----------------------+--------------------------------------------------------------------------------+

.. table::
   :class: longtable
   :widths: 20 80

   +----------------------+--------------------------------------------------------------------------------+
   | **Test Case No.:**   | MODE-CHACHA20-1                                                                |
   +======================+================================================================================+
   | **Type:**            | Positive vector test                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Description:**     | Runs ChaCha20 stream cipher vectors                                            |
   +----------------------+--------------------------------------------------------------------------------+
   | **Preconditions:**   | None                                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Input Values:**    | (ChaCha20 stream cipher vectors)                                               |
   +----------------------+--------------------------------------------------------------------------------+
   | **Expected Output:** | Keystream/XOR output matches expected vectors                                  |
   +----------------------+--------------------------------------------------------------------------------+
   | **Steps:**           | #. Execute the Botan 3.11 test case for MODE-CHACHA20-1 (ChaCha20 stream cipher|
   |                      | vectors)                                                                       |
   |                      |                                                                                |
   |                      | #. Verify the observed behavior matches the expected output.                   |
   +----------------------+--------------------------------------------------------------------------------+

.. table::
   :class: longtable
   :widths: 20 80

   +----------------------+--------------------------------------------------------------------------------+
   | **Test Case No.:**   | MODE-OFB-1                                                                     |
   +======================+================================================================================+
   | **Type:**            | Positive vector test                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Description:**     | Runs OFB stream-style mode vectors                                             |
   +----------------------+--------------------------------------------------------------------------------+
   | **Preconditions:**   | None                                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Input Values:**    | (OFB stream cipher vectors)                                                    |
   +----------------------+--------------------------------------------------------------------------------+
   | **Expected Output:** | Output stream bytes match expected test vectors                                |
   +----------------------+--------------------------------------------------------------------------------+
   | **Steps:**           | #. Execute the Botan 3.11 test case for MODE-OFB-1 (OFB stream cipher vectors) |
   |                      |                                                                                |
   |                      | #. Verify the observed behavior matches the expected output.                   |
   +----------------------+--------------------------------------------------------------------------------+

.. table::
   :class: longtable
   :widths: 20 80

   +----------------------+--------------------------------------------------------------------------------+
   | **Test Case No.:**   | MODE-RC4-1                                                                     |
   +======================+================================================================================+
   | **Type:**            | Positive vector test                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Description:**     | Runs RC4 known-answer vectors                                                  |
   +----------------------+--------------------------------------------------------------------------------+
   | **Preconditions:**   | None                                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Input Values:**    | (RC4 stream cipher vectors)                                                    |
   +----------------------+--------------------------------------------------------------------------------+
   | **Expected Output:** | Generated stream output matches vector expectations                            |
   +----------------------+--------------------------------------------------------------------------------+
   | **Steps:**           | #. Execute the Botan 3.11 test case for MODE-RC4-1 (RC4 stream cipher vectors) |
   |                      |                                                                                |
   |                      | #. Verify the observed behavior matches the expected output.                   |
   +----------------------+--------------------------------------------------------------------------------+

.. table::
   :class: longtable
   :widths: 20 80

   +----------------------+--------------------------------------------------------------------------------+
   | **Test Case No.:**   | MODE-SALSA20-1                                                                 |
   +======================+================================================================================+
   | **Type:**            | Positive vector test                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Description:**     | Runs Salsa20 stream cipher vectors                                             |
   +----------------------+--------------------------------------------------------------------------------+
   | **Preconditions:**   | None                                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Input Values:**    | (Salsa20 stream cipher vectors)                                                |
   +----------------------+--------------------------------------------------------------------------------+
   | **Expected Output:** | Produced keystream/ciphertext matches expected values                          |
   +----------------------+--------------------------------------------------------------------------------+
   | **Steps:**           | #. Execute the Botan 3.11 test case for MODE-SALSA20-1 (Salsa20 stream cipher  |
   |                      | vectors)                                                                       |
   |                      |                                                                                |
   |                      | #. Verify the observed behavior matches the expected output.                   |
   +----------------------+--------------------------------------------------------------------------------+

.. table::
   :class: longtable
   :widths: 20 80

   +----------------------+--------------------------------------------------------------------------------+
   | **Test Case No.:**   | MODE-SHAKE-1                                                                   |
   +======================+================================================================================+
   | **Type:**            | Positive vector test                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Description:**     | Covers SHAKE XOF/stream-style output tests                                     |
   +----------------------+--------------------------------------------------------------------------------+
   | **Preconditions:**   | None                                                                           |
   +----------------------+--------------------------------------------------------------------------------+
   | **Input Values:**    | (SHAKE stream/XOF vectors)                                                     |
   +----------------------+--------------------------------------------------------------------------------+
   | **Expected Output:** | Generated XOF output bytes match expected vectors                              |
   +----------------------+--------------------------------------------------------------------------------+
   | **Steps:**           | #. Execute the Botan 3.11 test case for MODE-SHAKE-1 (SHAKE stream/XOF vectors)|
   |                      |                                                                                |
   |                      | #. Verify the observed behavior matches the expected output.                   |
   +----------------------+--------------------------------------------------------------------------------+
