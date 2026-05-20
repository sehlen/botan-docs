# Review: Key Derivation Functions Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 06_kdf.rst

---


All KDF tests use the single `KDF_KAT_Tests` class in `src/tests/test_kdf.cpp`:

```cpp
KDF_KAT_Tests() : Text_Based_Test("kdf", "Secret,Output", "Salt,Label,IKM,XTS") {}
```

Required fields: `Secret`, `Output`. Optional fields: `Salt`, `Label`, `IKM`, `XTS`.
The test loads all `.vec` files under `src/tests/data/kdf/`.

---

### KDF-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- Core flow (create, name check, derive key, clone check) is accurately described.
- Step 2 has a typo: says "*InputSalt*" — should be "*Salt*".
- The spec does not mention the optional `IKM` (Input Keying Material) and `XTS` fields accepted by the test framework. Some KDF algorithms (notably HKDF variants) pass these via test vectors.
- The spec omits an additional test path: when the expected output is exactly 32 bytes, the code also calls the fixed-size variant `kdf->derive_key<32>(secret, salt, label)` and compares results. This is an additional positive test not described.
- The spec does not mention that output length is inferred from the `Output` field length (not a separate input parameter).

**Proposed Changes:**
- Fix typo "*InputSalt*" → "*Salt*" in Step 2.
- Add a note on optional `IKM` and `XTS` parameters (used by HKDF and HKDF-Extract).
- Add a step: "If the expected output is 32 bytes, also derive using the fixed-size `derive_key<32>()` overload and compare results."

---

### KDF-KDF1-1
**Status:** ✅ CONFIRMED

**Notes:**
- The spec correctly references `src/tests/data/kdf/kdf1_iso18033.vec` as the source of test vectors for KDF1 (ISO 18033-2).
- Number of test cases (2), hash functions (SHA-1, SHA-256), input/output sizes, and example vector are all accurate.
- Steps accurately describe creating the KDF1_18033 object and deriving a key.
- Note: there is also a separate `kdf1.vec` file for KDF1 per X9.63/IEEE 1363a — that algorithm is not mentioned anywhere in the spec (see New Tests section).

---

### KDF-NISTSP800-108-CTR-1
**Status:** ✅ CONFIRMED

**Notes:**
- Spec correctly identifies the test class, data file (`sp800_108_ctr.vec`), MAC variants, and parameter ranges.
- The example vector (HMAC-SHA1, 80-bit salt, 160-bit secret, 16-bit output) is accurate.
- Steps correctly describe passing `Salt` and `Secret` via the KDF interface.
- Minor omission: `Label` is also an optional parameter consumed by `derive_key()`, though it may be empty in these test vectors.

---

### KDF-NISTSP800-108-FB-1
**Status:** ✅ CONFIRMED

**Notes:**
- Spec correctly references `sp800_108_fb.vec`, constraints (240 test cases, same MAC set), and example vector.
- Steps are accurate.

---

### KDF-NISTSP800-108-PI-1
**Status:** ✅ CONFIRMED

**Notes:**
- Spec correctly references `sp800_108_pipe.vec`, constraints, and example vector.
- Steps are accurate.

---

### KDF-TLS1-PRF-1
**Status:** ✅ CONFIRMED

**Notes:**
- Spec correctly identifies the TLS 1.0/1.1 PRF, references `tls_prf.vec`, and the example vector (208-bit salt, 152-bit secret, 8-bit output) is accurate.

---

### KDF-TLS12-PRF-1
**Status:** ✅ CONFIRMED

**Notes:**
- Spec correctly identifies the TLS 1.2 PRF, references `tls_prf.vec`, constraints (4 test cases, SHA-224/256/384/512), and the example vector is accurate.
- Label is correctly listed as an additional input for TLS 1.2 PRF (unlike TLS 1.0/1.1).

---

---

## New Tests Found in Botan 3.12.0 Not in Spec


| Test data file | Algorithm | Notes |
|---|---|---|
| `src/tests/data/kdf/kdf1.vec` | KDF1 (X9.63 / IEEE 1363a) | Different from KDF1 ISO 18033-2; not mentioned |
| `src/tests/data/kdf/kdf2.vec` | KDF2 (ISO 18033-2) | Not mentioned anywhere in spec |
| `src/tests/data/kdf/sp800_56a.vec` | SP800-56A KDF | ACVP-sourced vectors with HMAC and KMAC variants; not mentioned |
| `src/tests/data/kdf/x942_prf.vec` | X9.42 PRF | Not mentioned |
| `src/tests/data/kdf/hkdf_label.vec` | HKDF-Expand-Label | Tested by a **separate** class `HKDF_Expand_Label_Tests` (not `KDF_KAT_Tests`); not covered in spec |

The `KDF_KAT_Tests` class also accepts `IKM` and `XTS` optional fields used by certain KDFs; the spec does not document these fields.
