# Review: Certificate Store, RNG, TPM Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**Files Reviewed:** 02_cert_store.rst, 15_rng.rst, 18_tpm.rst
**Source Files Checked:**
- `src/tests/test_certstor.cpp`
- `src/tests/test_certstor_system.cpp`
- `src/tests/test_rng_kat.cpp`
- `src/tests/test_rng_behavior.cpp` (unit tests; see critical note on file reference below)
- `src/tests/test_rngs.cpp` (test helper implementation only; NOT a test file in Botan 3.12.0)
- `src/tests/test_tpm2.cpp`

---

## 02_cert_store.rst

### CERTSTOR-ISR-1
**Status:** ✅ CONFIRMED
**Source:** `test_certstor_sqlite3_insert_find_remove_test()` in `test_certstor.cpp`
**Notes:** All eight steps in the spec are faithfully reflected in the implementation:
find by subject DN (`find_cert` without key ID), find by subject DN + key ID, look up private key by
cert, look up certs for key, remove cert, verify removal, remove key, verify key removal.
The test is run against all 6 test certificate/key pairs.

---

### CERTSTOR-REV-1
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_certstor_sqlite3_crl_test()` in `test_certstor.cpp`
**Notes:** The spec only mentions revoking Certs[3] once. The actual code calls
`store.revoke_cert(certsandkeys[3].certificate(), ...)` **twice** before the affirmation step.
This double-revocation is deliberate (idempotency check). The spec should document this.
**Proposed Changes:**
- In the Steps table, change the second occurrence of step 2 to:
  *"Revoke Certs[3] with reason CA Compromise a second time (idempotency check)"*

---

### CERTSTOR-SDN-1
**Status:** ✅ CONFIRMED
**Source:** `test_certstor_sqlite3_all_subjects_test()` in `test_certstor.cpp`
**Notes:** Spec accurately describes: insert all 6 certs, call `all_subjects()`, verify the returned
list has exactly 6 entries, and cross-check each returned DN against the known set.

---

### CERTSTOR-FAC-1
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_certstor_sqlite3_find_all_certs_test()` in `test_certstor.cpp`
**Notes:** The spec only documents the single-match lookup per certificate. The actual test also
inserts two certificates with **identical subject DNs** (from BSI test corpus:
`x509/bsi/common_14/common_14_sub_ca.ca.pem.crt` and `common_14_wrong_sub_ca.ca.pem.crt`) and
verifies that `find_all_certs(dn, {})` returns exactly 2 entries. This duplicate-DN branch is
unspecified.
**Proposed Changes:**
- Add a second scenario to the Steps:
  *"Insert two certificates sharing the same subject DN. Query by that DN with empty key ID.
  Check that exactly two certificates are returned."*

---

### CERTSTOR-SCH-1
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_certstor_all_finders()` in `test_certstor.cpp`
**Notes:** The spec describes only the SHA-256-hashed subject DN lookup
(`find_cert_by_raw_subject_dn_sha256`). The same function also tests
`find_cert_by_issuer_dn_and_serial_number`, which is not mentioned in the spec. Additionally, the
function tests the negative case: looking up with a 32-byte all-zero dummy hash returns no result.
This test uses `Certificate_Store_In_Memory`, not SQLite, so it covers the in-memory backend.
**Proposed Changes:**
- Expand Steps to include:
  1. *(existing)* Find each cert by SHA-256 hash of its raw subject DN.
  2. *(new)* Find each cert by its issuer DN and serial number.
  3. *(new)* Confirm that a lookup with a known-invalid 32-byte dummy hash returns no result.
- Clarify the Description to note that the store under test is `Certificate_Store_In_Memory`.

---

### CERTSTOR-SYSTEM-1
**Status:** ✅ CONFIRMED
**Source:** `find_certificate_by_pubkey_sha1()` and
`find_certificate_by_pubkey_sha1_with_unmatching_key_id()` in `test_certstor_system.cpp`
**Notes:** The spec correctly documents both sub-cases:
(a) the "typical" root certificate where the Subject Key Identifier equals the SHA-1 of the public
key, and (b) "SecureTrust CA" whose Subject Key Identifier deliberately differs from the SHA-1 of
the public key (regression test for GH #2779). Both functions are invoked from `Certstor_System_Tests::run()`.

---

### CERTSTOR-SYSTEM-2
**Status:** 🔄 PROPOSED CHANGES
**Source:** `find_cert_by_subject_dn()`, `find_cert_by_utf8_subject_dn()`, and
`find_all_certs_by_subject_dn()` in `test_certstor_system.cpp`
**Notes:** **The Steps table contains a copy-paste error:** Step 1 reads *"Query certificates by
their public key's SHA-1"*, which is the description for CERTSTOR-SYSTEM-1, not SYSTEM-2.
The actual test functions query by Subject Distinguished Name (PrintableString encoding and
UTF-8 encoding). The "find all certs by subject DN" variant also verifies that no duplicate
certificates are returned. The D-TRUST certificate note states it is disabled on Windows CI, which
is correctly captured in the Preconditions.
**Proposed Changes:**
- Fix Step 1: change *"Query certificates by their public key's SHA-1"* to
  *"Query certificates by their Subject Distinguished Name"*.
- Add to Steps: confirm that no duplicate certificates are returned (tested via sort+unique check
  in `find_all_certs_by_subject_dn`).

---

### CERTSTOR-SYSTEM-3
**Status:** ✅ CONFIRMED
**Source:** `find_cert_by_subject_dn_and_key_id()` and `find_certs_by_subject_dn_and_key_id()` in
`test_certstor_system.cpp`
**Notes:** Both the singular (`find_cert`) and plural (`find_all_certs`) variants of the DN+key ID
lookup are exercised. The plural variant additionally calls `certstore.contains()` to confirm the
returned certificate is recognised by the store. The spec captures the intent accurately.

---

### CERTSTOR-SYSTEM-4
**Status:** ✅ CONFIRMED
**Source:** `find_all_subjects()` in `test_certstor_system.cpp`
**Notes:** Spec accurately describes: call `all_subjects()`, confirm the list is non-empty, and
check that the DN of "ISRG Root X1" is present in the result.

---

### CERTSTOR-SYSTEM-5
**Status:** ✅ CONFIRMED
**Source:** `no_certificate_matches()` in `test_certstor_system.cpp`
**Notes:** Three distinct queries are issued with dummy data
(`find_all_certs`, `find_cert`, `find_cert_by_pubkey_sha1`) and all must return empty/null.
The spec's two sub-queries ((a) DN + key ID and (b) SHA-1 of public key) correspond exactly to
the three calls in the code. Minor: the code also passes a dummy key ID as the SHA-1 hash, which
is technically the same 28-byte value used for the DN query – this is an implementation detail
that does not affect spec accuracy.

---

## 15_rng.rst

> **Critical file reference error:** The RST states *"All unit tests for various RNGs are
> implemented in `src/tests/test_rngs.cpp`."* In Botan 3.12.0 this file is a **test-helper
> implementation** (providing `Fixed_Output_RNG` and `CTR_DRBG_AES256` for use by other test
> files). It registers no test cases and contains no `BOTAN_REGISTER_TEST` calls. The actual RNG
> unit tests (HMAC-DRBG unit tests, AutoSeeded_RNG, System_RNG, ChaCha_RNG unit tests) are in
> `src/tests/test_rng_behavior.cpp`. This reference must be corrected throughout the section.

---

### RNG-HMAC-DRBG-1
**Status:** ✅ CONFIRMED
**Source:** `HMAC_DRBG_Tests` (registered as `"hmac_drbg"`) in `test_rng_kat.cpp`
**Notes:** The spec correctly describes the KAT structure: `initialize_with(EntropyInput)`,
`add_entropy(EntropyInputReseed)`, two calls to `randomize_with_input` (first with AdditionalInput1
to discard, second with AdditionalInput2 producing the expected output). Test vectors come from
NIST CAVS 14.3. The example input/output hex values in the RST match entries in the vector file.
The test is registered with `BOTAN_REGISTER_SMOKE_TEST` (not `BOTAN_REGISTER_TEST`).

---

### RNG-HMAC-DRBG-2
**Status:** ✅ CONFIRMED
**Source:** `HMAC_DRBG_Unit_Tests::test_max_number_of_bytes_per_request()` in
`test_rng_behavior.cpp`
**Notes:** All four spec steps are present in the code:
1. MNBPR = 0 → `test_throws` (constructor rejects it).
2. MNBPR = 64 KiB + 1 → `test_throws` (constructor rejects it).
3. HMAC_DRBG instantiated with MNBPR = 64 and RI = 1.
4. Request of 65 bytes → 2 underlying RNG calls (split into ≤ 64-byte chunks).
The file reference in the RST should be updated from `test_rngs.cpp` to `test_rng_behavior.cpp`.

---

### RNG-HMAC-DRBG-3
**Status:** ✅ CONFIRMED
**Source:** `HMAC_DRBG_Unit_Tests::test_security_level()` in `test_rng_behavior.cpp`
**Notes:** Hash functions and corresponding security levels listed in the spec
(SHA-1 → 128, SHA-224 → 192, SHA-256/SHA-512-256/SHA-384/SHA-512 → 256) exactly match the
`approved_hash_fns` / `security_strengths` vectors in the source. The file reference in the RST
should be updated from `test_rngs.cpp` to `test_rng_behavior.cpp`.

---

### RNG-HMAC-DRBG-4
**Status:** ✅ CONFIRMED
**Source:** `HMAC_DRBG_Unit_Tests::test_reseed_kat()` in `test_rng_behavior.cpp`
**Notes:** All input/output hex values in the spec match the code exactly:
- SeedData: `0x00112233...EEFF` (32 bytes, repeated) ✓
- OutFirstRequest: `48D3B45AAB65EF92CCFCB9427EF20C90297065ECC1B8A525BFE4DC6FF36D0E38` ✓
- OutSecondRequest: `2F8FCA696832C984781123FD64F4B20C7379A25C87AB29A21C9BF468B0081CE2` ✓
- ReseedInterval = 2 ✓
The spec correctly notes that the second request triggers an automatic reseed (verifiable via
`counting_rng.randomize_count()` jumping from 0 to 1). File reference must be corrected.

---

### RNG-AUTO-RNG-1
**Status:** 🔄 PROPOSED CHANGES
**Source:** `AutoSeeded_RNG_Tests` (registered as `"auto_rng_unit"`) in `test_rng_behavior.cpp`
**Notes:**
1. **Step order mismatch:** The spec lists Step 1 as "empty entropy sources → throws", Step 2 as
   "Null_RNG → throws". The code tests **Null_RNG first, empty entropy sources second**. The
   logical meaning is the same but the order should match for traceability.
2. **Exception type imprecision:** The spec says each of steps 1–3 "throws a PRNG_Unseeded
   exception". The code uses `test_success` after a `try/catch(Botan::Not_Implemented&)` or
   `catch(std::exception&)` — not specifically `PRNG_Unseeded`. The actual exception is
   implementation-defined; the spec should not be overly specific.
3. **Steps 10 and 11 are duplicates** (both say "Check that the AutoSeeded_RNG is seeded").
   These correspond to two consecutive `is_seeded()` checks in the code at lines ~737/739. One
   can be removed from the spec.
4. **Missing edge-case steps:** The code also iterates over buffer sizes 0–4095, calling
   `randomize` and `add_entropy` for each — a coverage check not described in the spec.
5. **File reference:** Must be corrected from `test_rngs.cpp` to `test_rng_behavior.cpp`.

**Proposed Changes:**
- Swap Steps 1 and 2 to match code order (or add a note that order is implementation-dependent).
- Replace "throws a PRNG_Unseeded exception" with "throws an exception (construction fails)".
- Remove the duplicate seeded-check step (retain only one instance).
- Add a step: "Verify that the RNG accepts arbitrary-length input and output buffers (edge-case
  sweep over sizes 0–4095)."
- Correct the file reference.

---

### RNG-SYS-RNG-1
**Status:** ✅ CONFIRMED
**Source:** `System_RNG_Tests` (registered as `"system_rng"`) in `test_rng_behavior.cpp`
**Notes:** All steps in the spec are confirmed:
- Name is non-empty ✓
- Always seeded; `clear()` is a no-op that does not unseed ✓
- Reseed from global entropy sources ✓
- Buffer sweep (1–128 bytes) with `randomize` and `add_entropy` ✓
- 64-bit regression test (4 GiB + 1024 bytes) guarded by `run_long_tests() &&
  run_memory_intensive_tests() && (sizeof(size_t) > 4)` ✓

The precondition "(partially) 64-bit system" is accurate.
**File reference must be corrected** from `test_rngs.cpp` to `test_rng_behavior.cpp`.

---

## 18_tpm.rst

> All TPM tests are in `src/tests/test_tpm2.cpp`. They require a TPM 2.0 software emulator
> (IBM's `tpm2-simulator`, identified by vendor `"SW   TPM"` / manufacturer `"IBM"`) and are
> skipped if the TCTI is configured as `"disabled"` or the emulator reports unexpected values.
> Some tests additionally require `BOTAN_HAS_TPM2_RSA_ADAPTER` or
> `BOTAN_HAS_TPM2_ECC_ADAPTER` to be enabled at build time.
>
> **Checklist note:** TPM-session-1/2/3 are present in the RST but absent from the
> TEST_SPEC_CHECKLIST. They should be added to the checklist.

---

### TPM-session-1
**Status:** ✅ CONFIRMED
**Source:** `test_tpm2_sessions()` → CHECK `"Unauthenticated sessions"` in `test_tpm2.cpp`
**Notes:** All four session variants tested in the spec match the code:
default (no args), `"CFB(AES-128)"`, `"CFB(AES-128)" + "SHA-384"`, `"CFB(AES-128)" + "SHA-1"`.
Each session is validated for a non-null pointer, a valid (non-NONE) ESYS handle, and a
non-empty TPM nonce.

---

### TPM-session-2
**Status:** ✅ CONFIRMED
**Source:** `test_tpm2_sessions()` → CHECK `"Authenticated sessions SRK"` in `test_tpm2.cpp`
**Notes:** Requires `BOTAN_HAS_TPM2_RSA_ADAPTER`. The SRK is fetched via
`ctx->storage_root_key({}, {})`, confirmed to be RSA-2048 with a persistent handle, and then used
as the salt key for four `Session::authenticated_session` variants (same algorithm combinations
as TPM-session-1). All returned sessions are validated for validity.

---

### TPM-session-3
**Status:** ✅ CONFIRMED
**Source:** `test_tpm2_sessions()` → CHECK `"Authenticated sessions ECC"` in `test_tpm2.cpp`
**Notes:** Requires `BOTAN_HAS_TPM2_ECC_ADAPTER`. Loads the ECC key from the persistent handle
configured via `tpm2_persistent_ecc_handle()`, confirms `algo_name() == "ECDSA"` and that the
handle is persistent, then creates four authenticated sessions using the ECC key as salt key.
This maps exactly to the spec's Step 3–7.

---

### TPM-RNG-1
**Status:** ✅ CONFIRMED
**Source:** `test_tpm2_rng()` → CHECK `"Basic functionalities"` in `test_tpm2.cpp`
**Notes:** All six spec steps confirmed:
1. Context created ✓
2. `TPM2::RandomNumberGenerator` object created with unauthenticated session ✓
3. `rng.accepts_input()` returns true ✓
4. `rng.is_seeded()` returns true ✓
5. `rng.name()` equals `"TPM2_RNG"` ✓
6. `rng.clear()` completes without error (`test_no_throw`) ✓

---

### TPM-RNG-2
**Status:** ✅ CONFIRMED
**Source:** `test_tpm2_rng()` → CHECK `"Random number generation"` in `test_tpm2.cpp`
**Notes:** Three buffer sizes (8, 15, 256 bytes) exactly match the spec. Each output is validated
by `not_zero_64()` which checks that no aligned 64-bit word is zero, a probabilistic non-zero check.

---

### TPM-RNG-3
**Status:** ✅ CONFIRMED
**Source:** `test_tpm2_rng()` → CHECK `"Randomize with inputs"` in `test_tpm2.cpp`
**Notes:** Three (output, entropy-input) size pairs match spec exactly:
(9, 30), (66, 64), (256, 196). All use `randomize_with_input` and the non-zero check.

---

### TPM-RSA-1
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_tpm2_rsa()` → CHECKs `"verify signature"` and
`"sign and verify multiple messages with the same Signer/Verifier objects"` in `test_tpm2.cpp`
**Notes:** The spec describes a single sign+verify flow. In the code this is split across multiple
CHECK blocks. The "verify signature" block:
- Signs a message (`baadcafe`) using the TPM private key with `PSS(SHA-256)`.
- Verifies the signature using a `TPM2::RSA_PublicKey` (which routes the public-key operation
  through the TPM).
- Mutates the message and confirms verification fails.
- Importantly, also tests that a ESAPI session-attribute bug workaround is correct (encrypt flag
  not cleared after a failed verify).

The spec step 4 says *"Verify with Botan's software implementation of RSA"* but the code verifies
using `TPM2::RSA_PublicKey` (TPM path). Software verification is tested in the
`"sign and verify multiple messages"` block. The order of software vs. TPM verification in the
spec should match the code.

**Proposed Changes:**
- Reorder Steps 4 and 5 to: (4) verify using the TPM, (5) verify using Botan's software RSA.
- Mention the PSS(SHA-256) padding scheme used.
- Note that this covers multiple CHECK sub-cases in the source.

---

### TPM-RSA-2
**Status:** ✅ CONFIRMED
**Source:** `test_tpm2_rsa()` → CHECK `"Wrong password is not accepted during signing"` in
`test_tpm2.cpp`
**Notes:** Spec matches code exactly: load the persistent key with wrong auth value
(`deadbeef`), attempt to sign, confirm `Botan::TPM2::Error` is thrown.

---

### TPM-RSA-3
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_tpm2_rsa()` → CHECKs `"Encrypt a message"` and `"Decrypt a message"` in
`test_tpm2.cpp`
**Notes:** **Step 3 has an inaccuracy.** The spec says *"Encrypt the plaintext message
"feedc0debaadcafe" using RSA-OAEP on the TPM"*. RSA-OAEP encryption is a public-key operation and
does **not** require the TPM. In the code, `PK_Encryptor_EME` is instantiated with
`TPM2::RSA_PublicKey` and a `Null_RNG`, but the actual encryption runs in software (only decryption
uses the TPM hardware). Steps 4–9 are accurate.

The padding scheme used throughout is `OAEP(SHA-256)`, not the unqualified "RSA-OAEP" mentioned in
the spec.

Decryption failure (step 9) results in `Botan::Decoding_Error`, not a generic "padding failure".

**Proposed Changes:**
- Fix Step 3: *"Encrypt the plaintext 'feedc0debaadcafe' using RSA-OAEP(SHA-256) in software
  with the TPM's public key"*.
- Specify `OAEP(SHA-256)` where "RSA-OAEP" appears.
- Fix Step 9: *"Verify that decryption fails with a Botan::Decoding_Error"*.

---

### TPM-RSA-4
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_tpm2_rsa()` → CHECKs `"Create a transient key and encrypt/decrypt a message"`,
`"Create a new transient key"`, and `"Make a transient key persistent then remove it again"` in
`test_tpm2.cpp`
**Notes:**
1. **Typo in spec:** Step 6 reads *"Decrypt the ciphertext using RSA-RSA-PKCSv1.5 on the TPM"*.
   The duplicated "RSA-RSA-" is a typo; it should be *"PKCS1v15"*.
2. **Encryption paths:** As with TPM-RSA-3, steps 3 and 5 describe encryption; both use a
   software-side encryptor with the TPM's exported public key. The wording "via Botan's software
   RSA implementation" in the spec is correct for step 3/5, but contradicts the TPM-RSA-3 steps
   where the same operation is called "on the TPM". Normalise the wording.
3. **Key export check (step 7):** The code checks `raw_private_key_bits()` returns a non-empty
   blob and `raw_public_key_bits()` returns a non-empty blob. It also verifies that calling
   `private_key_bits()` on a **persistent** key (not transient) throws `Not_Implemented`, and
   `raw_private_key_bits()` on a persistent key throws `Invalid_State`. This distinction is not in
   the spec.
4. **Signing algorithm:** Code uses PSS(SHA-256); spec does not specify the padding scheme.

**Proposed Changes:**
- Fix typo "RSA-RSA-PKCSv1.5" → "PKCS1v15".
- Note that key export is only possible for transient keys; persistent keys throw on export attempts.
- Mention PSS(SHA-256) as the signing algorithm.

---

### TPM-ECDSA-1
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_tpm2_ecc()` → CHECKs `"Sign a message ECDSA"`, `"verify signature ECDSA"`,
`"sign and verify multiple messages with the same Signer/Verifier objects"` in `test_tpm2.cpp`
**Notes:** Same structural observation as TPM-RSA-1 regarding step ordering. Code verifies
signatures using `TPM2::EC_PublicKey` (TPM path) **and** a software-extracted `ECDSA_PublicKey`.
The signing algorithm is `"SHA-256"` (default ECDSA scheme in Botan; corresponds to ECDSA with
SHA-256). The spec does not mention the hash function.
The session-attribute ESAPI workaround is also tested here (as in RSA).

**Proposed Changes:**
- Reorder Steps 4/5 to match code (TPM verify before software verify).
- Mention SHA-256 as the hash algorithm used in the ECDSA scheme.

---

### TPM-ECDSA-2
**Status:** ✅ CONFIRMED
**Source:** `test_tpm2_ecc()` → CHECK `"Wrong password is not accepted during ECDSA signing"` in
`test_tpm2.cpp`
**Notes:** Identical pattern to TPM-RSA-2: wrong auth value → sign attempt → `Botan::TPM2::Error`
thrown.

---

### TPM-ECDSA-3
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_tpm2_ecc()` → CHECK `"Create a transient ECDSA key and sign/verify a message"`
in `test_tpm2.cpp`
**Notes:** The code creates the transient ECDSA key on curve `secp521r1` (P-521), which is not
mentioned in the spec. The session used for key creation is an **authenticated** session backed by
the **ECC persistent key** (not the SRK) — the spec says "authenticated session via the Storage
Root Key", which is only partly correct (the authenticated session here uses the ECC key directly).
Persistence/eviction steps match the spec.

**Proposed Changes:**
- Note that the transient key uses curve `secp521r1`.
- Correct Step 1: the authenticated session is created using the **ECC persistent key**, not the
  SRK. (The SRK is used indirectly as the parent key for key creation, but the session salt key is
  the ECC key.)

---

## New Tests Found in Botan 3.12.0 Not in Spec

### 02_cert_store.rst
| Function | Registration | Description |
|---|---|---|
| `test_certstor_load_allcert()` | `"certstor"` (via `Certstor_Tests::run()`) | Loads all certificates from a directory containing a **bundled** PEM file (two concatenated certificates). Verifies that `Certificate_Store_In_Memory` loads both certs from a single multi-cert file, while `X509_Certificate` only loads the first. No spec test case exists for this. |

### 15_rng.rst
| Function | Registration | Description |
|---|---|---|
| `ChaCha_RNG_Tests` | `"chacha_rng"` (test_rng_kat.cpp) | KAT tests for ChaCha_RNG using vector file `rng/chacha_rng.vec`. |
| `ChaCha_RNG_Unit_Tests` | `"chacha_rng_unit"` (test_rng_behavior.cpp) | Unit tests for ChaCha RNG paralleling HMAC_DRBG unit tests. |
| `hmac_drbg_multiple_requests` | `"hmac_drbg_multi_request"` (test_rng_behavior.cpp) | Tests that a bulk randomize request produces the same output as the equivalent split into max-size chunks, both with and without additional input. |
| `Processor_RNG_Tests` | `"processor_rng"` (test_rng_behavior.cpp) | Tests CPU hardware RNG (e.g., RDRAND). |

### 18_tpm.rst
| Function | Registration | Description |
|---|---|---|
| `test_tpm2_properties()` | `"tpm2_props"` | Checks TPM vendor/manufacturer strings, max random bytes per request, and which algorithms are (un)supported. |
| `test_tpm2_context()` | `"tpm2_ctx"` | Validates persistent handle list, crypto backend availability, and SRK retrieval. |
| `test_external_tpm2_context()` | `"tpm2_external_ctx"` | Tests Botan's TPM2 wrapper against an externally-provided `ESYS_CONTEXT*`; verifies the ESYS context remains usable after the Botan `TPM2::Context` is destroyed. Also tests the free-standing crypto backend. |
| `test_tpm2_hash()` | `"tpm2_hash"` | Tests `TPM2::HashFunction` for SHA-1/256/384/512 and SHA-3 variants; verifies multi-update, single-process, long messages (> max buffer size), `clear()`, `new_object()`, validation tickets, and that MD-5 and `copy_state` throw appropriately. |

---

## Checklist Discrepancies Found

1. **TPM-session-1, TPM-session-2, TPM-session-3 are in `18_tpm.rst` but absent from
   `TEST_SPEC_CHECKLIST.md`** (under section `18_tpm.rst (TPM)`). These should be added.

2. **`test_rngs.cpp` is misidentified** in the RST as the unit test file. In Botan 3.12.0 it is a
   helper implementation file. The correct file is `test_rng_behavior.cpp`. All cross-references
   in `15_rng.rst` that point to `test_rngs.cpp` for unit tests must be updated.

3. **`CERTSTOR-SYSTEM-2` Steps table copy-paste error:** Step 1 says "Query certificates by their
   public key's SHA-1" — should be "Query certificates by their Subject Distinguished Name".

4. **No spec test cases exist** for `test_certstor_load_allcert`, `test_tpm2_properties`,
   `test_tpm2_context`, `test_external_tpm2_context`, `test_tpm2_hash`, `ChaCha_RNG_Tests`,
   `ChaCha_RNG_Unit_Tests`, `hmac_drbg_multiple_requests`, and `Processor_RNG_Tests`. Consider
   adding spec entries for these test functions in future spec revisions.
