# Review: Public Key Encryption, KEM, Key Agreement Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**Files Reviewed:** 11_pubkey_enc.rst, 12_pubkey_kem.rst, 13_pubkey_agree.rst
**Source Files Checked:**
- `src/tests/test_dlies.cpp` (SHA: 6dad11d)
- `src/tests/test_ecies.cpp` (SHA: 1848505)
- `src/tests/test_rsa.cpp` (SHA: 17480a8)
- `src/tests/test_pubkey.cpp` (SHA: 9ed5867)
- `src/tests/test_cmce.cpp` (SHA: 817c0ef)
- `src/tests/test_frodokem.cpp` (SHA: 11a2e38)
- `src/tests/test_kyber.cpp` (SHA: aa2640c) ← file is **not** renamed in 3.12.0
- `src/tests/test_dh.cpp` (SHA: 6c6152a)
- `src/tests/test_ecdh.cpp` (SHA: 77c2133)

---

## 11_pubkey_enc.rst

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

## 12_pubkey_kem.rst

### PKENC-CMCE-1
**Status:** ✅ CONFIRMED
**Notes:**
`Classic_McEliece_KAT_Tests` (registered as `cmce/cmce_generic_kat`) uses the
`PK_PQC_KEM_KAT_Test` base class with `pubkey/cmce_kat_hashed.vec`. The test
infrastructure seeds an AES-256-CTR-DRBG with the test vector seed, draws the
required seed bytes from it into a `Fixed_Output_RNG`, and passes that to key
generation. The hashing uses SHAKE-256(512) for key material, as stated in the
spec. All described steps match the implementation.

---

### PKENC-CMCE-2
**Status:** ✅ CONFIRMED
**Notes:**
`CMCE_Invalid_Test` (registered as `cmce/cmce_invalid`) reads from
`pubkey/cmce_negative.vec` with fields `seed, ct_invalid, ss_invalid` and
optional `ct_invalid_c1, ss_invalid_c1`. It uses `CTR_DRBG_AES256` seeded with
the KAT seed to generate the key pair, decapsulates the invalid ciphertext (a
valid one with a single bit flipped), and compares the result against the
expected invalid shared secret. For `pc` variants it additionally checks that
flipping a bit in C₁ also changes the shared secret. The spec accurately
describes all of this.

---

### PKENC-FRODO-1
**Status:** ✅ CONFIRMED
**Notes:**
`Frodo_KAT_Tests` (registered as `frodokem/frodo_kat_tests`) uses
`PK_PQC_KEM_KAT_Test` with `pubkey/frodokem_kat.vec`. The `map_value()` method
applies `SHAKE-256` with 16-byte (128-bit) output to public/private key values
(but NOT to the shared secret), matching the spec's statement "hashed using
SHAKE-256(128) to save disk space". All described steps match the implementation.

---

### PKENC-FRODO-2
**Status:** 🔄 PROPOSED CHANGES
**Notes:**
`test_frodo_roundtrips()` (registered as `frodokem/frodo_roundtrips`) is
accurately described overall. However, **Step 6** in the spec says:
> *"Truncate the ciphertext by a single byte and attempt a decapsulation with
>  the original private key. Expect a decryption failure."*

The actual behavior for a truncated ciphertext is that an **exception is thrown**
(`"FrodoKEM ciphertext does not have the correct byte count"`), not a silent
decryption failure returning a different shared secret:
```cpp
result.test_throws("malformed encapsulation value",
    "FrodoKEM ciphertext does not have the correct byte count", [&] {
        auto short_encaps_value = enc_res.encapsulated_shared_key();
        short_encaps_value.pop_back();
        dec1.decrypt(short_encaps_value, 0);
    });
```

**Proposed change for Step 6:**
> *"Truncate the ciphertext by a single byte and attempt a decapsulation with
>  the original private key. Expect an exception with the message 'FrodoKEM
>  ciphertext does not have the correct byte count'."*

---

### PKENC-ML-KEM-1
**Status:** ✅ CONFIRMED (with note)
**Notes:**
The file reference `src/tests/test_kyber.cpp` is **correct** — the file was
**not** renamed to `test_ml_kem.cpp` in Botan 3.12.0. Both the Kyber R3 KAT
(`KyberR3_KAT_Tests`, registered as `pubkey/kyber_kat`) and the ML-KEM KAT
(`ML_KEM_KAT_Tests`, registered as `pubkey/ml_kem_kat`) live in this file.

The `ml_kem.vec` test file additionally contains `CT_N` (invalid ciphertext) and
`SS_N` (shared secret for invalid ciphertext) fields, enabling the implicit
rejection / decapsulation-failure path, which matches the spec's listed expected
output values.

Note on RNG seeding for ML-KEM: the key-gen RNG is seeded by drawing `z` (32
bytes) then `d` (32 bytes) from the outer AES-256-CTR-DRBG and concatenating
them as `d ‖ z` for `Fixed_Output_RNG`. The spec's generic statement "Seed an
AES-256-CTR-DRBG with the specified RNG seed" is accurate at the infrastructure
level.

---

### PKENC-ML-KEM-2
**Status:** ✅ CONFIRMED
**Notes:**
`KYBER_Tests::run_kyber_test()` (registered as `pubkey/kyber_pairwise`) covers
all ML-KEM and Kyber mode variants. It serializes both keys, reconstructs them
from the serialized bytes, encapsulates with the reconstructed public key, and
decapsulates with the reconstructed private key, confirming shared-secret
equality. All steps match.

---

### PKENC-ML-KEM-3
**Status:** 🔄 PROPOSED CHANGES
**Notes:**
Two issues:

1. **Inconsistent wording in Step 3**: The spec says *"Generate a kyber key
   pair"*. Since this test covers all ML-KEM and Kyber instances, it should say
   *"Generate an ML-KEM (or Kyber) key pair (one for each supported instance)"*.

2. **Step 6 incorrectly treats both failure modes the same**: The spec says
   *"Expect a failure in both cases"* for the short ciphertext and the reversed
   ciphertext. The actual behaviors differ:
   - **Short ciphertext** → throws an exception:
     `"Kyber: unexpected ciphertext length"`
   - **Reversed ciphertext** → returns silently but with a *different*
     pseudo-random shared secret (FO-transform implicit rejection); no exception
     is thrown.

**Proposed change for Step 6:**
> *"Decode the private key and try to decapsulate both altered ciphertexts:*
> - *For the truncated ciphertext: expect an exception ('Kyber: unexpected
>   ciphertext length').*
> - *For the reversed ciphertext: expect that decapsulation succeeds but
>   produces a different shared secret than the one from Step 3
>   (implicit rejection)."*

---

### PKENC-ML-KEM-4
**Status:** 🔄 PROPOSED CHANGES
**Notes:**
`Kyber_Encoding_Test` (registered as `pubkey/kyber_encodings`) correctly reads
from `pubkey/kyber_encodings.vec`. The described steps (decode, check error
message on failure, re-encode and validate on success) are accurate.

However, the implementation also performs **seed vs. expanded format testing**
that is not mentioned in the spec. When the private key is in *seed* format
(`MlPrivateKeyFormat::Seed`), the test additionally:
- Verifies the seed encoding is round-trippable.
- Derives the *expanded* encoding from the seed.
- Confirms that requesting a seed encoding from the expanded key throws
  `Botan::Encoding_Error`.
- Performs a full encapsulation/decapsulation roundtrip using the expanded key
  to confirm functional equivalence.

**Proposed addition to Steps section:**
> *"If the decoded private key is in seed (compact) format:*
> - *Verify that re-encoding in seed format is byte-compatible with the input.*
> - *Derive the expanded encoding and confirm it is self-consistent.*
> - *Verify that requesting seed format from an expanded key raises an error.*
> - *Perform an encap/decap roundtrip using the expanded key."*

---

### PKENC-ML-KEM-5
**Status:** ✅ CONFIRMED
**Notes:**
`ML_KEM_PQC_KEM_ACVP_KAT_Encap_Test` (registered as
`pubkey/ml_kem_acvp_kat_encap`) reads the public key from the `EK` field
(encapsulation key) and the encapsulation message, constructs a
`Fixed_Output_RNG` containing the message, and compares ciphertext and shared
secret against the expected values. The spec accurately describes this.

---

### PKENC-ML-KEM-6
**Status:** ✅ CONFIRMED
**Notes:**
`ML_KEM_ACVP_KAT_KeyGen_Tests` (registered as `pubkey/ml_kem_acvp_kat_keygen`)
reads `D` (key-gen seed) and `Z` (implicit rejection seed) from
`pubkey/ml_kem_acvp_keygen.vec`, concatenates them as `d ‖ z` into a
`Fixed_Output_RNG`, runs key generation, and compares against expected public
and private keys. The spec accurately describes this.

---

### PKENC-RSAKEM-1
**Status:** ✅ CONFIRMED
**Notes:**
`RSA_KEM_Tests` (registered as `pubkey/rsa_kem`) uses `PK_KEM_Test` with
`pubkey/rsa_kem.vec` and fields `E, P, Q, R, C0, KDF, K`. The test loads the
RSA private key, uses the KDF to derive a shared secret, and compares both the
encapsulated key C0 and the shared secret K. The spec accurately describes this.

---

## 13_pubkey_agree.rst

### KA-KEY-1
**Status:** ✅ CONFIRMED
**Notes:**
Implemented in `PK_Key_Generation_Test::run()` (`src/tests/test_pubkey.cpp`).
Generates a key pair, encodes the public key as PEM, decodes it, and checks
validity and algorithm name. All steps match.

---

### KA-KEY-2
**Status:** ✅ CONFIRMED
**Notes:**
Same infrastructure as KA-KEY-1, but encodes/decodes the public key as BER
instead of PEM. Steps match.

---

### KA-KEY-3
**Status:** ✅ CONFIRMED
**Notes:**
Encodes/decodes the private key as PEM. Steps match.

---

### KA-KEY-4
**Status:** ✅ CONFIRMED
**Notes:**
Encodes/decodes the private key as BER. Steps match.

---

### KA-KEY-5
**Status:** ✅ CONFIRMED
**Notes:**
Encodes/decodes the private key as PEM protected by a random password. Steps
match.

---

### KA-KEY-6
**Status:** 🔄 PROPOSED CHANGES
**Notes:**
The steps in the spec are in the wrong order. Step 2 reads:
> *"Check that the generated public key is valid and its estimated strength
>  satisfies the requirements"*

but this appears **before** Step 3:
> *"Generate a random keypair on the Group/Curve"*

A key cannot be checked for validity before it is generated. This is likely a
copy-paste error from KA-KEY-5 where the order is correct.

**Proposed corrected Steps section:**
> 1. Generate a random password string of length between 1–32 characters.
> 2. Generate a random keypair on the *Group*/*Curve*.
> 3. Check that the generated public key is valid and its estimated strength
>    satisfies the requirements.
> 4. Encode the keypair as a BER-encoded byte array, protected with the password.
> 5. Create a Private_Key object from the BER-encoded byte array.
> 6. Check that the key object is valid.
> 7. Check that the key object algorithm name equals that of the generated
>    keypair.
> 8. Check that the key is valid (see KA-KEY-1).

---

### KA-DH-1
**Status:** ✅ CONFIRMED
**Notes:**
`Diffie_Hellman_KAT_Tests` (registered as `pubkey/dh_kat`) reads from
`pubkey/dh.vec` with required fields `P, G, X, Y, K` and optional `Q, KDF,
OutLen`. The example test vector values and the two-step derivation description
match the implementation.

---

### KA-DH-2
**Status:** ✅ CONFIRMED
**Notes:**
`Diffie_Hellman_KAT_Tests::run_final_tests()` tests that DH rejects Y values
greater than P−1. The error message thrown is `"DH agreement - invalid key
provided"`. The spec describes this correctly.

---

### KA-DH-3
**Status:** ✅ CONFIRMED
**Notes:**
`run_final_tests()` also tests that DH rejects Y ≤ 1 (using Y = 1). The same
error message applies. The spec describes this correctly.

---

### KA-KEY-DH-1
**Status:** ✅ CONFIRMED
**Notes:**
`Diffie_Hellman_Keygen_Tests` (registered as `pubkey/dh_keygen`) uses
`keygen_params = {"modp/ietf/1024"}` and inherits the full
`PK_Key_Generation_Test` encode/decode suite. The spec's described key validity
checks (primality tests, 1 < Y < P, G ≥ 2, P ≥ 3, subgroup order conditions)
accurately reflect the Botan DH key check logic.

---

### KA-KEY-DH-INVALID-1
**Status:** ✅ CONFIRMED
**Notes:**
`DH_Invalid_Key_Tests` (registered as `pubkey/dh_invalid`) reads from
`pubkey/dh_invalid.vec` with fields `P, Q, G, InvalidKey`. It loads a
`DH_PublicKey` and asserts `check_key()` returns false. The described 7 NIST
CAVP test vectors and the validity check conditions in the spec match the
implementation.

---

### KA-ECDH-1
**Status:** ✅ CONFIRMED
**Notes:**
`ECDH_KAT_Tests` (registered as `pubkey/ecdh_kat`) reads from `pubkey/ecdh.vec`
with required fields `Secret, CounterKey, K` and optional `KDF`. The two-step
derivation (load key from Curve + Secret, derive with CounterKey, compare K) is
correctly described. The 150-test-case count and NIST CAVS source are plausible.

---

### KA-KEY-ECDH-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:**
`ECDH_Keygen_Tests` (registered as `pubkey/ecdh_keygen`) uses the following
`keygen_params`:
```cpp
{"secp256r1", "secp384r1", "secp521r1",
 "brainpool256r1", "brainpool384r1", "brainpool512r1", "frp256v1"}
```

The spec's constraints section lists only:
> *"secp256r1, secp384r1, secp521r1, brainpool256r1, brainpool384r1, frp256v1"*

**`brainpool512r1` is missing from the spec's curve list.**

**Proposed change:** Add `brainpool512r1` to the constraints list:
> *"Curve: secp256r1, secp384r1, secp521r1, brainpool256r1, brainpool384r1,
>  brainpool512r1, frp256v1"*

---

## New Tests Found in Botan 3.12.0 Not in Spec

### 11_pubkey_enc.rst — missing coverage

| Test ID | Class | Registration | File |
|---------|-------|--------------|------|
| — | `ECIES_Tests` | `pubkey/ecies` | `test_ecies.cpp` |
| — | `DLIES_Unit_Tests` | `pubkey/dlies_unit` | `test_dlies.cpp` |
| — | `RSA_Blinding_Tests` | `pubkey/rsa_blinding` | `test_rsa.cpp` |
| — | `RSA_DecryptOrRandom_Tests` | `pubkey/rsa_decrypt_or_random` | `test_rsa.cpp` |

**`pubkey/ecies`** (`ECIES_Tests`) reads from `pubkey/ecies.vec` (distinct from
the ISO 18033 file) and tests ECIES on named curves (secp192r1, secp256r1,
secp384r1, secp521r1, secp112r2 with cofactor) with a full parameter grid. This
is a significant test class not mentioned anywhere in the spec.

**`pubkey/dlies_unit`** (`DLIES_Unit_Tests`) exercises the XOR-stream cipher
mode of DLIES (no block cipher needed) using several KDF/MAC combinations. It
includes negative tests for "other public key not set" and "ciphertext too
short".

**`pubkey/rsa_blinding`** (`RSA_Blinding_Tests`) verifies that RSA signing and
decryption blinding reinitialisation works correctly when the fixed-output RNG
is exhausted.

**`pubkey/rsa_decrypt_or_random`** (`RSA_DecryptOrRandom_Tests`) tests the
`PK_Decryptor_EME::decrypt_or_random` API: it verifies that the method always
returns a plausible-length output for malformed ciphertexts (for PKCS#1 v1.5 and
OAEP), and that content-checking works correctly for both valid and invalid
content requirements.

### 12_pubkey_kem.rst — missing coverage

| Test ID | Class | Registration | File |
|---------|-------|--------------|------|
| — | `CMCE_Utility_Tests` | `cmce/cmce_utility` | `test_cmce.cpp` |
| — | `CMCE_Generic_Keygen_Tests` | `cmce/cmce_generic_keygen` | `test_cmce.cpp` |
| — | `Frodo_Keygen_Tests` | `frodokem/frodo_keygen` | `test_frodokem.cpp` |
| — | `Kyber_Keygen_Tests` | `pubkey/kyber_keygen` | `test_kyber.cpp` |
| — | `test_kyber_helpers` | `pubkey/kyber_helpers` | `test_kyber.cpp` |

**`cmce/cmce_utility`** contains five internal unit tests: seed expansion
against a reference, irreducible polynomial generation, GF inversion, GF
polynomial multiplication, and a "rigged RNG" test that verifies the
encapsulation rejection loop terminates gracefully.

**`cmce/cmce_generic_keygen`** and **`frodokem/frodo_keygen`** are generic
`PK_Key_Generation_Test` instances that check key serialisation round-trips and
validity for CMCE and FrodoKEM respectively.

**`pubkey/kyber_keygen`** (`Kyber_Keygen_Tests`) is the generic key generation /
round-trip test covering all Kyber-90s, Kyber-R3, and ML-KEM parameter sets.

**`pubkey/kyber_helpers`** tests the internal compress/decompress functions
(for `d ∈ {1,4,5,10,11}`) against their mathematical definitions, and verifies
that `compress(decompress(x)) == x`.

### 13_pubkey_agree.rst — missing coverage

| Test ID | Class | Registration | File |
|---------|-------|--------------|------|
| — | `ECDH_AllGroups_Tests` | `pubkey/ecdh_all_groups` | `test_ecdh.cpp` |

**`pubkey/ecdh_all_groups`** (`ECDH_AllGroups_Tests`) iterates over every named
EC group known to Botan and for each one:
- Regression test: all-zero private key is rejected with `Invalid_Argument`.
- Regression test: identity point (point at infinity) as public key is rejected.
- Regression test: all-zero SEC1-encoded public value triggers
  `Decoding_Error`.
- Regression test: a point not on the curve triggers `Decoding_Error`.
- Runs 100 randomised ECDH agreement roundtrips and asserts both sides produce
  the same shared secret.

This is a comprehensive live test across all supported groups and should be
documented in the spec.
