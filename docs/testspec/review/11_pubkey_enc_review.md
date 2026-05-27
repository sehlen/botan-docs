# Review: Public Key Encryption Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 11_pubkey_enc.rst
**Source Files:** `src/tests/test_dlies.cpp`, `src/tests/test_ecies.cpp`, `src/tests/test_rsa.cpp`

---


### PKENC-DLIES-1
**Status:** ✅ CONFIRMED (with minor note)
**Notes:**
The test class `DLIES_KAT_Tests` (registered as `pubkey/dlies`) reads from
`pubkey/dlies.vec` with fields `Kdf, Mac, MacKeyLen, Group, X1, X2, Msg,
Ciphertext` and optional `IV`. The spec accurately describes the encrypt/decrypt
roundtrip. The negative ciphertext check (`check_invalid_ciphertexts`) is also
executed within each KAT vector run, matching the spec's implicit negative test
intent.

Minor note: The spec steps say "Create a DH_PrivateKey object from *P, Q, G* and
*X1*", but in the actual test the group is loaded by named identifier via
`DL_Group::from_name(group_name)`, not from raw P, Q, G parameters. The test
vector file contains the group name (e.g. `modp/ietf/2048`), not raw field
values. The spec's description of the group as "2048 bits (MODP Group, RFC 3526)"
is conceptually correct.

---

### PKENC-DLIES-2
**Status:** 🔄 PROPOSED CHANGES
**Notes:**
Two issues were found:

1. **Wrong terminology in Description**: The spec says *"Invalid signatures
   should not verify"*. DLIES is a hybrid *encryption* scheme, not a signature
   scheme. The description should read *"Invalid ciphertexts should not decrypt
   correctly"*.

2. **Steps describe normal decryption, not the negative test**: The three listed
   steps simply reproduce the normal decryption path from PKENC-DLIES-1 (using
   P2 and P1 to decrypt). They do not describe the actual negative test behavior,
   which is performed by the generic `check_invalid_ciphertexts` helper embedded
   in the same KAT test loop. That helper modifies the ciphertext (bit flips /
   length changes) and asserts that decryption throws or returns wrong output.

**Proposed Changes:**
- Replace description: *"Invalid ciphertexts should not decrypt correctly"*
- Replace Steps section:
  > 1. For each KAT vector, after successful decryption, generate multiple
  >    mutated versions of the *Ciphertext* by randomly flipping bits or altering
  >    the ciphertext length.
  > 2. Attempt to decrypt each mutated ciphertext with the same decryptor.
  > 3. Verify that each mutated ciphertext either causes an exception or produces
  >    output that does not match the original *Msg*.

---

### PKENC-ECIES-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:**
The test class is `ECIES_ISO_Tests` (registered as `pubkey/ecies_iso`), reading
from `pubkey/ecies-18033.vec`. The 96-test-case count (2 vectors × 48 mode
combinations) and the general test structure are correctly described.

**Error in Step 5**: The spec says:
> *"Use PR1 and PU1 to derive a shared secret of 128 bytes using KDF1-18033(SHA-1)"*

This is incorrect. In the test, **PR1** is Bob's private key and **PU1** is
Bob's public key — using a party's own private key with their own public key
does not constitute a key agreement. The actual code is:
```cpp
const ECIES_KA_Operation ka(eph_private_key, ka_params, true, this->rng());
const SymmetricKey secret_key = ka.derive_secret(eph_public_key_bin,
                                                 other_public_key_point);
```
The shared secret is derived from **PR2** (Alice's ephemeral private key) and
**PU1** (Bob's public key).

**Proposed change for Step 5:**
> *"Use PR2 (Alice's ephemeral private key) and PU1 (Bob's public key) to derive
>  a shared secret of 128 bytes using KDF1-18033(SHA-1) and *Format*, and compare
>  with the expected output *K*"*

Additional minor note: `ECIES_ISO_Tests::skip_this_test` returns `true` when
`!Botan::EC_Group::supports_application_specific_group()`, meaning the ISO 18033
test is silently skipped in builds that do not support application-specific
groups. The spec does not mention this conditional availability.

---

### PKENC-ECIES-2
**Status:** ✅ CONFIRMED
**Notes:**
The test checks that constructing `ECIES_System_Params` with more than one of
`{cofactor_mode, old_cofactor_mode, check_mode}` simultaneously enabled throws
an exception:
```cpp
if(size_t(cofactor_mode) + size_t(check_mode) + size_t(old_cofactor_mode) > 1) {
    result.test_throws("throw on invalid ECIES_Flags", onThrow);
    continue;
}
```
The spec correctly identifies the input (cofactor_mode=enabled,
old_cofactor_mode=enabled) and expected behavior (exception thrown).

---

### PKENC-RSAES-1
**Status:** ✅ CONFIRMED
**Notes:**
`RSA_ES_KAT_Tests` (registered as `pubkey/rsa_encrypt`) uses
`PK_Encryption_Decryption_Test` with `pubkey/rsaes.vec` and fields `E, P, Q,
Msg, Ciphertext` (optional `Nonce`). The spec steps accurately describe the
load-key → decrypt → encrypt → re-decrypt flow. The negative test
(`check_invalid_ciphertexts`) is automatically called within the base class.

---

### PKENC-RSAES-2
**Status:** ✅ CONFIRMED
**Notes:**
The negative test is embedded in `PK_Encryption_Decryption_Test::run_one_test()`
via `check_invalid_ciphertexts`. The spec correctly describes modifying the
ciphertext and checking that decryption fails.

---

### PKENC-RSAES-3
**Status:** ✅ CONFIRMED
**Notes:**
`RSA_Decryption_KAT_Tests` (registered as `pubkey/rsa_decrypt`) uses
`PK_Decryption_Test` with `pubkey/rsa_decrypt.vec` and fields `E, P, Q,
Ciphertext, Msg`. The test decrypts only (no encryption step). The spec
accurately describes this.

---

---

## New Tests Found in Botan 3.12.0 Not in Spec


| Test ID | Class | Registration | File | First Added |
|---------|-------|--------------|------|-------------|
| — | `ECIES_Tests` | `pubkey/ecies` | `test_ecies.cpp` | Pre-3.7.1 (before 2024) |
| — | `DLIES_Unit_Tests` | `pubkey/dlies_unit` | `test_dlies.cpp` | Pre-3.7.1 (before 2024) |
| — | `RSA_Blinding_Tests` | `pubkey/rsa_blinding` | `test_rsa.cpp` | Dec 2016 (commit 2f9d7b7) |
| — | `RSA_DecryptOrRandom_Tests` | `pubkey/rsa_decrypt_or_random` | `test_rsa.cpp` | Jan 2025 (commit ca4f797) |

### Timeline Context

**ECIES_Tests** — Tests ECIES (not ISO 18033 variant) on named curves (secp192r1, secp256r1, secp384r1, secp521r1, secp112r2 with cofactor) with full parameter grid. This test existed before the 3.7.1 baseline and was missed during previous documentation.

**DLIES_Unit_Tests** — Tests DLIES XOR-stream cipher mode (no block cipher) with various KDF/MAC combinations. Includes negative tests for "other public key not set" and "ciphertext too short". This test existed before the 3.7.1 baseline and was missed.

**RSA_Blinding_Tests** — Added in December 2016 (commit 2f9d7b7), well before the 3.7.1 baseline. This test verifies that RSA signing and decryption blinding reinitialisation works correctly when the fixed-output RNG is exhausted. It predates 3.7.1 and was missed during previous documentation.

**RSA_DecryptOrRandom_Tests** — Added in January 2025 (commit ca4f797), after the 3.7.1 baseline. This is a NEW test that validates the `PK_Decryptor_EME::decrypt_or_random` API: it verifies that the method always returns a plausible-length output for malformed ciphertexts (for PKCS#1 v1.5 and OAEP), and that content-checking works correctly for both valid and invalid content requirements. This test was added after the baseline.

**Conclusion:** ECIES_Tests, DLIES_Unit_Tests, and RSA_Blinding_Tests were missed during previous documentation cycles (existed before 3.7.1). RSA_DecryptOrRandom_Tests is a new test added after the 3.7.1 baseline.
