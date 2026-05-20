# Review: Public Key Signatures Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**Files Reviewed:** `14_pubkey_sig.rst`
**Source Files Checked:**
- `src/tests/test_pubkey.cpp`
- `src/tests/test_dilithium.cpp`
- `src/tests/test_ml_dsa.cpp`
- `src/tests/test_dsa.cpp`
- `src/tests/test_ecdsa.cpp`
- `src/tests/test_ecgdsa.cpp`
- `src/tests/test_eckcdsa.cpp`
- `src/tests/test_hss_lms.cpp`
- `src/tests/test_rsa.cpp`
- `src/tests/test_xmss.cpp`
- `src/tests/test_sphincsplus.cpp` (SLH-DSA; no `test_slh_dsa.cpp` exists in 3.12.0)

---

## Generic Signature Tests (test_pubkey.cpp)

### PKSIG-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `PK_Signature_Generation_Test::run_one_test`. The implementation:
1. Creates a private key, verifies the KAT signature, signs the message and compares against KAT, then verifies the generated signature. Accurately described. The spec step #2 says "Verify the signature *Signature* on the *Msg*" which matches `result.test_is_true("KAT signature valid", verifier->verify_message(...))`.

---

### PKSIG-2
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `check_invalid_signatures()` called from within `PK_Signature_Generation_Test::run_one_test`. Tests zero signature and bit-flipped/length-modified signatures. Accurately described.

---

### PKSIG-3
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `PK_Signature_NonVerification_Test::run_one_test`. Loads a known-invalid signature from test vectors and checks it does not verify. Accurately described.

---

### PKSIG-4
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `PK_Signature_Verification_Test::run_one_test` with `Valid=1`. Creates a public key and verifies a given signature. Accurately described.

---

### PKSIG-KEY-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to the PEM public key encode/decode portion of `PK_Key_Generation_Test::run()`. Accurately described.

---

### PKSIG-KEY-2
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to the BER public key encode/decode portion of `PK_Key_Generation_Test::run()`. Accurately described.

---

### PKSIG-KEY-3
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to the PEM private key encode/decode portion of `PK_Key_Generation_Test::run()`. Accurately described.

---

### PKSIG-KEY-4
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to the BER private key encode/decode portion of `PK_Key_Generation_Test::run()`. Accurately described.

---

### PKSIG-KEY-5
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to the PEM private key encode/decode with password protection portion of `PK_Key_Generation_Test::run()`. Accurately described.

---

### PKSIG-KEY-6
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to the BER private key encode/decode with password protection portion of `PK_Key_Generation_Test::run()`. Accurately described.

---

## ML-DSA (formerly Dilithium)

**Section-level issues:**
- The spec states (line 411): *"All ML-DSA-specific test code can be found in `:srcref:`src/tests/test_dilithium.cpp``."*
  In Botan 3.12.0, ML-DSA test code is split across **two** files:
  - `src/tests/test_dilithium.cpp` — contains `Dilithium_KAT_Tests` template (covering both Dilithium and ML-DSA KAT via macros), `DilithiumRoundtripTests`, and `Dilithium_Keygen_Tests`.
  - `src/tests/test_ml_dsa.cpp` — contains a separate `ML_DSA_Verify_KAT_Tests` class (registered as `ml_dsa_verify`) using a different test vector format (`pubkey/ml_dsa_verify.vec` with fields `Mode,Key,Msg,Signature,Valid`).
  **Proposed change:** Update the source file reference to mention both files.

---

### PKSIG-ML-DSA-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Corresponds to `Dilithium_KAT_Tests` / `REGISTER_ML_DSA_KAT_TEST` macros in `test_dilithium.cpp`. The spec description is accurate for the Dilithium-based KAT test. However, Botan 3.12.0 also registers a separate `ml_dsa_verify` test (`ML_DSA_Verify_KAT_Tests` in `test_ml_dsa.cpp`) that uses a distinct vector format (`Mode,Key,Msg,Signature,Valid` from `pubkey/ml_dsa_verify.vec`). This second ML-DSA verification KAT is not mentioned in PKSIG-ML-DSA-1 and should either be added here or as a new test case.

**Proposed Changes:**
- Add a reference to `src/tests/test_ml_dsa.cpp` (registered test `ml_dsa_verify`) and its vector file `pubkey/ml_dsa_verify.vec` in the input values list or as a note.

---

### PKSIG-ML-DSA-2
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `DilithiumRoundtripTests::run()` in `test_dilithium.cpp`. All described steps map correctly: random key generation, signing, encode/decode cycle, cross-verification with all key combinations, tampered-message rejection, and generic API decoding. Accurately described.

---

### PKSIG-ML-DSA-3
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Corresponds to `Dilithium_Keygen_Tests` in `test_dilithium.cpp`, which is a `PK_Key_Generation_Test`. The spec describes a sign/verify cycle after key encoding/decoding, but `PK_Key_Generation_Test` is the generic key encoding test (PKSIG-KEY-1 through PKSIG-KEY-6), not a dedicated sign-then-verify flow. The step-by-step description in PKSIG-ML-DSA-3 (encode keys, decode private key, sign, decode public key, verify) is actually a subset of what PKSIG-ML-DSA-2 covers. The spec step description somewhat duplicates PKSIG-ML-DSA-2 and may be confusing.

**Proposed Changes:**
- Clarify that PKSIG-ML-DSA-3 maps to the generic `Dilithium_Keygen_Tests` (and thus also the PKSIG-KEY-1 through PKSIG-KEY-6 key serialization tests), and note the distinction from PKSIG-ML-DSA-2.

---

## DSA

### PKSIG-DSA-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `DSA_KAT_Tests` (registered as `dsa_kat_sign`). When `BOTAN_HAS_RFC6979_GENERATOR` is available (default), uses `pubkey/dsa_rfc6979.vec` with fields `P,Q,G,X,Hash,Msg,Signature`; otherwise uses `dsa_prob.vec` with an extra `Nonce` field. The spec example shows a `Nonce` field and references `dsa_prob.vec`, which reflects the non-RFC6979 build configuration. This is a minor inconsistency for users building with RFC6979 (the standard).

**Proposed Changes:**
- Note that with `BOTAN_HAS_RFC6979_GENERATOR` (default), test vectors come from `dsa_rfc6979.vec` (no Nonce). The `dsa_prob.vec` path applies to non-RFC6979 builds. The spec's example vector and file reference reflect the non-RFC6979 path.

---

### PKSIG-DSA-2
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Corresponds to `PK_Signature_Generation_Test` negative checks from the same `DSA_KAT_Tests` flow. Accurately described as manipulation/zero-signature checks. However, the spec's input values reference `Nonce` which only applies to the non-RFC6979 build. Same DSA RFC6979 note applies.

---

### PKSIG-DSA-4
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `DSA_Verification_Tests` (registered as `dsa_misc_verify`) using `pubkey/dsa_verify.vec`. The spec description and example vector match correctly.

---

### PKSIG-KEY-DSA-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `DSA_Keygen_Tests` (registered as `dsa_keygen`) with param `dsa/jce/1024`. Steps match `PK_Key_Generation_Test::run()` which exercises PEM/BER encode/decode plus validity checks. The spec describes DSA-specific validity checks (Miller-Rabin primality tests, group structure checks) which are accurate for DSA.

---

## ECDSA

### PKSIG-ECDSA-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `ECDSA_Signature_KAT_Tests` (registered as `ecdsa_sign`). With RFC6979, uses `ecdsa_rfc6979.vec`; without, `ecdsa_prob.vec`. Example in spec shows a Nonce and matches the probabilistic path. Accurately described.

---

### PKSIG-ECDSA-2
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to the manipulation/negative checks in `PK_Signature_Generation_Test`. Accurately described. Same RFC6979 note as PKSIG-DSA-1 applies.

---

### PKSIG-ECDSA-4
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `ECDSA_Verification_Tests` (registered as `ecdsa_verify`) using `pubkey/ecdsa_verify.vec`. Fields `Group,Px,Py,Msg,Signature,Valid`. Accurately described; the example uses raw (no hash) verification.

---

### PKSIG-KEY-ECDSA-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `ECDSA_Keygen_Tests` (registered as `ecdsa_keygen`) covering all known named groups. The spec says constraints are `secp256r1, secp384r1, secp521r1` but the actual test uses *all* `EC_Group::known_named_groups()`. Minor underspecification in the constraint list, but not incorrect.

---

### PKSIG-PUBKEY-VAL-ECDSA-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `ECDSA_Invalid_Key_Tests` (registered as `ecdsa_invalid`) using `pubkey/ecdsa_invalid.vec`. The test attempts to construct an `EC_AffinePoint` from the given (x,y) coordinates and expects it to fail for invalid points. Accurately described.

---

## ECGDSA

### PKSIG-ECGDSA-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Corresponds to `ECGDSA_Signature_KAT_Tests` (registered as `ecgdsa_sign`) using `pubkey/ecgdsa.vec`. The spec example shows `Curve = secp224r1` but the spec's constraint summary lists `brainpool192r1, brainpool256r1, brainpool320r1, brainpool384r1, brainpool512r1`. The test vector file is the actual source of truth; the example curve in the spec (`secp224r1`) may not be present in the constraint list. The keygen tests actually use `secp256r1, secp384r1, secp521r1`.

**Proposed Changes:**
- Verify the example input `Curve = secp224r1` against the actual `ecgdsa.vec` vectors, and reconcile with the constraint summary (which lists brainpool curves). The keygen constraints should reflect the actual `ECGDSA_Keygen_Tests` params: `secp256r1, secp384r1, secp521r1`.

---

### PKSIG-ECGDSA-2
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to the manipulation/negative checks in `PK_Signature_Generation_Test` flow. Accurately described.

---

### PKSIG-KEY-ECGDSA-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `ECGDSA_Keygen_Tests` (registered as `ecgdsa_keygen`) using `secp256r1, secp384r1, secp521r1`. Accurately described.

---

## ECKCDSA

### PKSIG-ECKCDSA-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `ECKCDSA_Signature_KAT_Tests` (registered as `eckcdsa_sign`) using `pubkey/eckcdsa.vec`. Accurately described; example input and steps match the code.

---

### PKSIG-ECKCDSA-2
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to the manipulation/negative checks within `PK_Signature_Generation_Test`. Accurately described.

---

### PKSIG-KEY-ECKCDSA-1 (labelled PKSIG-KEY-ECDSA-1 in RST — BUG)
**Status:** 🔄 PROPOSED CHANGES
**Notes:** The spec at line 1258 shows the test ID as `PKSIG-KEY-ECDSA-1` and the description as "Encode and decode an ECKCDSA public key as PEM", but this is the *ECKCDSA* keygen test case. The test ID is incorrect — it should be `PKSIG-KEY-ECKCDSA-1`. Furthermore, step 1 says "Generate a random keypair on the ECDSA *Curve*" (should be ECKCDSA), and there is no separate `PKSIG-KEY-ECKCDSA-1` ID defined anywhere else in the document. This is a copy-paste error from the ECDSA section.

**Proposed Changes:**
- Change the test ID from `PKSIG-KEY-ECDSA-1` to `PKSIG-KEY-ECKCDSA-1`.
- Fix step 1: "Generate a random keypair on the **ECKCDSA** *Curve*".

---

## HSS/LMS

### PKSIG-HSS/LMS-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `HSS_LMS_Negative_Tests::test_too_short_signature()` (part of `hss_lms_negative`). The spec describes truncating the signature iteratively and verifying that each truncation fails. The code iterates from 0 to `valid_sig.size()-1` bytes and checks each fails. Accurately described.

---

### PKSIG-HSS/LMS-2
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `HSS_LMS_Statefulness_Test::test_max_sig_count()` (part of `hss_lms_state`). The spec describes creating a key with one remaining signature, using it, then confirming exhaustion. The code does exactly this. Accurately described.

---

## RSA

### PKSIG-RSA-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `RSA_Signature_KAT_Tests` (registered as `rsa_sign`) using `pubkey/rsa_sig.vec` with fields `E,P,Q,Msg,Signature` and optional `Nonce`. Default padding is `Raw`. Accurately described.

---

### PKSIG-RSA-2
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Corresponds to the manipulation/negative checks in `PK_Signature_Generation_Test`. Accurately described. Minor wording issue: step 3 in the spec ends with a question mark ("does not verify?") — likely a typo.

**Proposed Changes:**
- Fix the question mark in step 3: "Check that a signature with all zeros ... does not verify?" → "does not verify."

---

### PKSIG-3 (inside RSA section — labelled incorrectly as PKSIG-3, not PKSIG-RSA-3)
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Lines 1482–1510 show a table with `Test Case No.: PKSIG-3` inside the RSA section. This is clearly intended to be `PKSIG-RSA-3` (the RSA-specific application of the generic PKSIG-3 pattern). The test ID `PKSIG-3` is already used at the top of the file for the generic test. This is a duplicate/incorrect ID.

**Proposed Changes:**
- Rename this test case to `PKSIG-RSA-3`.

---

### PKSIG-KEY-RSA-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `RSA_Keygen_Tests` (registered as `rsa_keygen`) with params `1024` and `1280` bits. Steps match `PK_Key_Generation_Test::run()` with RSA-specific validity checks (N >= 35, N odd, E >= 2). Accurately described.

---

## SLH-DSA (formerly SPHINCS+)

**Section-level notes:**
- The spec does not mention the source file for SLH-DSA tests. The tests are in `src/tests/test_sphincsplus.cpp` (there is no `test_slh_dsa.cpp` in Botan 3.12.0).
- `test_sphincsplus.cpp` handles both legacy SPHINCS+ (Round 3) and standardized SLH-DSA through the same `SPHINCS_Plus_Test_Base` class, registered separately as `sphincsplus` and `slh_dsa`.

**Proposed Changes:**
- Add `:srcref:`src/tests/test_sphincsplus.cpp`` as the source file reference in the SLH-DSA section introduction.

---

### PKSIG-SLH-DSA-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `SPHINCS_Plus_Test_Base` / `SLH_DSA_Test` (registered as `slh_dsa`) using `pubkey/slh_dsa.vec` and `SPHINCS_Plus_Test` (registered as `sphincsplus`) using `pubkey/sphincsplus.vec`. The test:
1. Seeds a CTR-DRBG, generates a keypair, compares pk/sk to vectors.
2. Signs the message in randomized mode, compares signature hash to vector.
3. Verifies the randomized signature.
4. Optionally (for deterministic-capable modes) runs deterministic signing and verification.
5. For `128bit Fast` params: exercises deserialization and invalid-signature rejection.
Accurately described.

---

### PKSIG-SLH-DSA-2
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to WOTS+ unit tests in `test_sphincsplus.cpp` using `pubkey/sphincsplus_wots.vec`. The spec accurately describes recreating a WOTS+ signature and key and checking against stored hashes.

---

### PKSIG-SLH-DSA-3
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to FORS unit tests in `test_sphincsplus.cpp` using `pubkey/sphincsplus_fors.vec`. The spec accurately describes generating a FORS signature and public key and comparing against test vectors.

---

## XMSS

### PKSIG-XMSS-1
**Status:** ✅ CONFIRMED
**Notes:** Corresponds to `XMSS_Signature_Tests` (registered as `xmss_sign`) using `pubkey/xmss_sig.vec` with fields `Params,Msg,PrivateKey,Signature`. Steps match the `PK_Signature_Generation_Test` flow. Accurately described.

---

### PKSIG-XMSS-2
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Corresponds to `XMSS_Signature_Verify_Tests` (registered as `xmss_verify`) using `pubkey/xmss_verify.vec`. The description says "Valid signatures should verify" (positive test), which is correct. However, the step description at line 1825 reads: "Check that the **is modified** signature does not verifies" — this is copied from the negative test (PKSIG-XMSS-3) and is wrong for a positive test.

**Proposed Changes:**
- Fix step 2 to: "Check that the signature *Signature* on *Msg* verifies."
- Fix grammatical errors: "Check that the is modified signature does not verifies" → proper description matching a positive verification test.

---

### PKSIG-XMSS-3
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Corresponds to `XMSS_Signature_Verify_Invalid_Tests` (registered as `xmss_verify_invalid`) using `pubkey/xmss_invalid.vec`. The spec accurately describes a negative test. Minor issue: the spec text at line 1785-1786 says "The test case PKSIG-XMSS-2 and **PKCS**-XMSS-3" — "PKCS" should be "PKSIG".

**Proposed Changes:**
- Fix "PKCS-XMSS-3" to "PKSIG-XMSS-3" in the prose (line 1785–1786).

---

## New Tests Found in Botan 3.12.0 Not in Spec

### test_ml_dsa.cpp
- **`ml_dsa_verify`** (`ML_DSA_Verify_KAT_Tests`): Verification-only KAT test using `pubkey/ml_dsa_verify.vec` with a distinct format (`Mode,Key,Msg,Signature,Valid`). Not referenced in the spec at all.

### test_dilithium.cpp
- **`dilithium_keygen`** (`Dilithium_Keygen_Tests`): Key generation test covering all Dilithium and ML-DSA instances. Partially described as PKSIG-ML-DSA-3 but not fully.

### test_ecdsa.cpp
- **`ecdsa_sign_verify_der`** (`ECDSA_Sign_Verify_DER_Test`): Tests DER-formatted ECDSA signature creation and verification with SHA-512 and secp256r1.
- **`ecdsa_all_groups`** (`ECDSA_AllGroups_Test`): Runs sign/verify across all known EC groups and multiple hash functions (SHA-256, SHA-384, SHA-512, SHAKE variants).
- **`ecdsa_key_recovery`** (`ECDSA_Key_Recovery_Tests`): Tests ECDSA public key recovery from a signature, using `pubkey/ecdsa_key_recovery.vec`.
- **`ecdsa_keygen_stability`** (`ECDSA_Keygen_Stability_Tests`): Tests key generation stability across repeated keygen calls using `pubkey/ecdsa_keygen.vec`.
- **`ecdsa_explicit_curve_key`** (`ECDSA_ExplicitCurveKey_Test`): Tests loading ECDSA keys with explicitly encoded (non-OID) curve parameters from `pubkey/ecdsa_explicit.vec`.

### test_hss_lms.cpp
- **`hss_lms_params_parsing`** (`test_hss_lms_params_parsing`): Tests correct parsing of HSS-LMS parameter strings (e.g., `"SHA-256,HW(5,1),HW(25,8)"`), checking level count, hash name, and algorithm types.
- **`hss_lms_verify_invalid`** (`HSS_LMS_Signature_Verify_Invalid_Tests`): KAT-based negative test using `pubkey/hss_lms_invalid.vec`. Distinct from the bit-flip test (PKSIG-HSS/LMS-1).
- **`hss_lms_negative`** – also covers `test_too_short_private_key` and `test_too_short_public_key` (verifying that partial key bytes throw `Decoding_Error`). Only the signature truncation portion is described in PKSIG-HSS/LMS-1; the key truncation tests are not mentioned.
- **`hss_lms_api`** (`HSS_LMS_Missing_API_Test`): Tests miscellaneous API coverage: `key_length()`, `hash_function()`, `raw_private_key_bits()`, `algorithm_identifier()`. Not mentioned in spec.
- **`hss_lms_sign`** (`HSS_LMS_Signature_Generation_Test`): KAT-based signing test using `pubkey/hss_lms_sig.vec`. Listed in spec's test vector section but not explicitly as a named test case.
- **`hss_lms_verify`** (`HSS_LMS_Signature_Verify_Tests`): KAT-based verification test using `pubkey/hss_lms_verify.vec`. Same note.
- **`hss_lms_keygen`** (`HSS_LMS_Key_Generation_Test`): Key generation test using param `"SHA-256,HW(10,4),HW(5,8)"`. Exercises generic key generation infrastructure (PKSIG-KEY-1 through PKSIG-KEY-6) but not explicitly listed.
- **`hss_lms_state`** – also covers `test_sig_changes_state` (signing decrements remaining count, first sig uses index 0, second uses index 1). Only the key exhaustion scenario is described in PKSIG-HSS/LMS-2; the state-change-per-signature test is not mentioned.

### test_rsa.cpp
- **`rsa_pss`** (`RSA_PSS_KAT_Tests`): KAT for RSA-PSS using `pubkey/rsa_pss.vec` with `PSS(hash,MGF1,saltlen)` padding. Not mentioned in spec.
- **`rsa_pss_raw`** (`RSA_PSS_Raw_KAT_Tests`): KAT for RSA-PSS-Raw using `pubkey/rsa_pss_raw.vec` with `PSS_Raw(...)` padding. Not mentioned in spec.
- **`rsa_keygen_stability`** (`RSA_Keygen_Stability_Tests`): Key generation stability test using `pubkey/rsa_keygen.vec`. Not mentioned in spec.
- **`rsa_keygen_badrng`** (`RSA_Keygen_Bad_RNG_Test`): Tests that key generation with a bad (repeating-pattern) RNG fails with `Internal_Error`. Not mentioned in spec.
- **`rsa_blinding`** (`RSA_Blinding_Tests`): Tests RSA blinding for signing and decryption, including blinder re-initialization at `Blinder::ReinitInterval`. Not mentioned in spec.
- **`rsa_decrypt_or_random`** (`RSA_DecryptOrRandom_Tests`): Tests `decrypt_or_random()` API (constant-time decryption with random fallback) for PKCS1v15 and OAEP. Not mentioned in spec (also partly relevant to encryption, not just signing).

### test_sphincsplus.cpp (SLH-DSA)
- **`sphincsplus`** (`SPHINCS_Plus_Test`): KAT test for legacy SPHINCS+ (Round 3.1) instances using `pubkey/sphincsplus.vec`. Not mentioned in spec alongside `slh_dsa`.
- **`slh_dsa_keygen`** (`SPHINCS_Plus_Keygen_Tests`): Key generation test covering all SLH-DSA and SPHINCS+ parameter sets. Not mentioned in spec.
- **`slh_dsa_sign_generic`** (`Generic_SlhDsa_Signature_Tests`): Generic sign KAT using `pubkey/slh_dsa_generic.vec` with Randomized or Deterministic padding. Not mentioned in spec.
- **`slh_dsa_verify_generic`** (`Generic_SlhDsa_Verification_Tests`): Generic verify test using `pubkey/slh_dsa_generic.vec`. Not mentioned in spec.

### test_xmss.cpp
- **`xmss_keygen`** (`XMSS_Keygen_Tests`): Key generation test with `XMSS-SHA2_10_256` and `XMSS-SHA2_10_192`. Not explicitly mentioned in spec.
- **`xmss_keygen_reference`** (`XMSS_Keygen_Reference_Test`): Tests XMSS key generation compatibility with the reference implementation using `pubkey/xmss_keygen_reference.vec` (fields: `Params,SecretSeed,PublicSeed,SecretPrf,PublicKey,PrivateKey`). Not mentioned in spec.
- **`xmss_unit_tests`** (via `xmss_statefulness` + `xmss_legacy_private_key`):
  - `xmss_statefulness`: Tests that signing decrements remaining operations (1024 → 1023), and that a fully exhausted key raises an exception.
  - `xmss_legacy_private_key`: Tests backward compatibility with XMSS private keys generated before Botan 3.0's multi-target attack mitigation (SP.800-208). Verifies that legacy keys can still sign and that legacy signatures still verify. Not mentioned in spec.

---

## Summary of All Proposed Changes

| Issue | Location in RST | Change Needed |
|-------|----------------|---------------|
| ML-DSA source file incomplete | line 411 | Add `src/tests/test_ml_dsa.cpp` alongside `test_dilithium.cpp` |
| `ml_dsa_verify` test not covered | PKSIG-ML-DSA-1 | Add reference to `ML_DSA_Verify_KAT_Tests` and `ml_dsa_verify.vec` |
| PKSIG-ML-DSA-3 scope unclear | PKSIG-ML-DSA-3 | Clarify it maps to `Dilithium_Keygen_Tests` (generic keygen test) |
| DSA RFC6979 dual-path not described | PKSIG-DSA-1/2 | Note that RFC6979 build uses `dsa_rfc6979.vec` without Nonce |
| ECKCDSA keygen ID is wrong | line 1258 | Change `PKSIG-KEY-ECDSA-1` → `PKSIG-KEY-ECKCDSA-1` |
| ECKCDSA step 1 says "ECDSA Curve" | line 1270 | Fix to say "ECKCDSA Curve" |
| ECGDSA constraint/example curve mismatch | PKSIG-ECGDSA-1 | Reconcile curve constraints (spec lists brainpool; keygen uses secp) |
| RSA section has duplicate test ID | line 1482 | Rename `PKSIG-3` to `PKSIG-RSA-3` |
| Typo question mark in PKSIG-RSA-2 | line 1470 | Remove trailing "?" from "does not verify?" |
| PKSIG-XMSS-2 step description is wrong | line 1825 | Replace negative-test language with positive-verification language |
| Typo "PKCS-XMSS-3" | line 1785 | Fix to "PKSIG-XMSS-3" |
| SLH-DSA source file not mentioned | SLH-DSA section | Add reference to `src/tests/test_sphincsplus.cpp` |
| Many new tests not in spec | Multiple | See "New Tests Found" section above for full list |
