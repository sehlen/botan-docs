# Review: Public Key Agreement Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 13_pubkey_agree.rst
**Source Files:** `src/tests/test_dh.cpp`, `src/tests/test_ecdh.cpp`, `src/tests/test_pubkey.cpp`

---


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

---

## New Tests Found in Botan 3.12.0 Not in Spec


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
