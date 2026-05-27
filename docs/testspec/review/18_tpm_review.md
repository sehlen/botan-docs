# Review: TPM Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 18_tpm.rst
**Source File:** `src/tests/test_tpm2.cpp`

> All TPM tests require a TPM 2.0 software emulator (IBM's `tpm2-simulator`) and are
> skipped if the TCTI is configured as `"disabled"`. Some tests additionally require
> `BOTAN_HAS_TPM2_RSA_ADAPTER` or `BOTAN_HAS_TPM2_ECC_ADAPTER` to be enabled at build time.

---


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

---

## New Tests Found in Botan 3.12.0 Not in Spec

| Function | Registration | Description |
|---|---|---|
| `test_tpm2_properties()` | `"tpm2_props"` | Checks TPM vendor/manufacturer strings, max random bytes per request, and which algorithms are (un)supported. |
| `test_tpm2_context()` | `"tpm2_ctx"` | Validates persistent handle list, crypto backend availability, and SRK retrieval. |
| `test_external_tpm2_context()` | `"tpm2_external_ctx"` | Tests Botan's TPM2 wrapper against an externally-provided `ESYS_CONTEXT*`; verifies the ESYS context remains usable after the Botan `TPM2::Context` is destroyed. Also tests the free-standing crypto backend. |
| `test_tpm2_hash()` | `"tpm2_hash"` | Tests `TPM2::HashFunction` for SHA-1/256/384/512 and SHA-3 variants; verifies multi-update, single-process, long messages (> max buffer size), `clear()`, `new_object()`, validation tickets, and that MD-5 and `copy_state` throw appropriately. |

---
