# Review: Public Key KEM Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 12_pubkey_kem.rst
**Source Files:** `src/tests/test_cmce.cpp`, `src/tests/test_frodokem.cpp`, `src/tests/test_kyber.cpp`

---


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

---

## New Tests Found in Botan 3.12.0 Not in Spec


| Test ID | Class | Registration | File | First Added |
|---------|-------|--------------|------|-------------|
| — | `CMCE_Utility_Tests` | `cmce/cmce_utility` | `test_cmce.cpp` | Nov 2024 (commit c256e1c) |
| — | `CMCE_Generic_Keygen_Tests` | `cmce/cmce_generic_keygen` | `test_cmce.cpp` | Nov 2024 (commit c256e1c) |
| — | `Frodo_Keygen_Tests` | `frodokem/frodo_keygen` | `test_frodokem.cpp` | Jan 2024 (commit 21b52d3) |
| — | `Kyber_Keygen_Tests` | `pubkey/kyber_keygen` | `test_kyber.cpp` | Jan 2023 (commit 7025017) |
| — | `test_kyber_helpers` | `pubkey/kyber_helpers` | `test_kyber.cpp` | Oct 2024 (commit 7cd161a, ML-KEM standardization) |

**Timeline Context:**

Based on commit history analysis:
- **Previous docs baseline (3.7.1)**: Released ~mid-2024
- **Classic McEliece tests** (Nov 2024): Added in Botan 3.11.0+ after 3.7.1 baseline → **New tests, not missed**
- **FrodoKEM keygen test** (Jan 2024): Added in Botan 3.9.0, likely before or around 3.7.1 → **Possibly missed during 3.7.1 docs**
- **Kyber keygen test** (Jan 2023): Added in Botan 3.4.0, well before 3.7.1 → **Missed during previous docs update**
- **Kyber helpers test** (Oct 2024): Added with ML-KEM FIPS 203 standardization in 3.11.0+ → **New tests, not missed**

**Conclusion**: The Kyber_Keygen_Tests was missed during the 3.7.1 documentation update. FrodoKEM and Classic McEliece tests were added after that baseline, so these represent new gaps rather than documentation oversights.

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

## Scope Classification for "New Tests"

The tests listed above fall into three distinct categories. The current .rst text explicitly
states that generic public-key API tests and some utility tests are **intentionally not
discussed in detail** for Classic McEliece, FrodoKEM, and ML-KEM. This must be noted
before treating these as simple omissions.

| Test | Classification | Rationale |
|------|----------------|-----------|
| `cmce/cmce_utility` | **INTERNAL_POLICY_DECISION** | Five internal unit tests (field arithmetic, RNG rejection loop). The spec may intentionally omit low-level implementation detail. Needs policy decision. |
| `cmce/cmce_generic_keygen` | **GENERIC_API_OUT_OF_SCOPE** | Generic `PK_Key_Generation_Test` for CMCE. The existing .rst explicitly says generic public-key API tests are not discussed in detail for Classic McEliece. This should NOT be listed as an unqualified omission. |
| `frodokem/frodo_keygen` | **GENERIC_API_OUT_OF_SCOPE** | Generic `PK_Key_Generation_Test` for FrodoKEM. Same scope rule applies — the .rst says generic tests are not discussed in detail for FrodoKEM. |
| `pubkey/kyber_keygen` | **GENERIC_API_OUT_OF_SCOPE** | Generic key generation / round-trip test for ML-KEM and Kyber variants. Same scope rule applies. |
| `pubkey/kyber_helpers` | **INTERNAL_POLICY_DECISION** | Tests compress/decompress internal functions. Policy decision needed whether implementation internals belong in the normative spec. |

**Action required:** Review the scope rule in `12_pubkey_kem.rst` and decide whether to:
1. Keep the existing exclusion and document it as `OUT_OF_SCOPE_DECISION` in the checklist.
2. Add a brief note in the spec acknowledging these tests exist but are intentionally excluded.
3. Promote any of the above to spec entries if the review determines they should be covered.
