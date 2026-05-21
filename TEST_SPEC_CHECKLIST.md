# Test Specification Verification Checklist

This checklist tracks the verification of test specifications against Botan 3.12.0.

**Target Botan Version:** 3.12.0
**Repository:** randombit/botan
**Generated:** 2026-05-14
**Last Updated:** 2026-05-20 (agent review complete; migrated to five-status model)

## Legend

| Status | Meaning |
|--------|---------|
| `CONFIRMED_NO_CHANGE` | Spec accurately matches source; no .rst change required |
| `MINOR_FIX` | Typo, wording, source-file reference, precondition, or guard issue |
| `SUBSTANTIVE_FIX` | Steps or expected behavior differ materially from source |
| `MISSING_TEST` | Source test exists but has no spec entry |
| `OUT_OF_SCOPE_DECISION` | Source test exists but spec intentionally omits it; rationale documented |

## Review Files

Detailed findings per RST file are in `docs/testspec/review/`:
- `01_aead_review.md` — 01_aead.rst
- `02_cert_store_review.md` — 02_cert_store.rst
- `03_block_ciphers_review.md` — 03_block_ciphers.rst
- `04_entropy_srcs_review.md` — 04_entropy_srcs.rst
- `05_hash_review.md` — 05_hash.rst
- `06_kdf_review.md` — 06_kdf.rst
- `07_mac_review.md` — 07_mac.rst
- `08_modes_of_operation_review.md` — 08_modes_of_operation.rst
- `09_pbkdf_review.md` — 09_pbkdf.rst
- `10_pkcs11_review.md` — 10_pkcs11.rst
- `11_pubkey_enc_review.md` — 11_pubkey_enc.rst
- `12_pubkey_kem_review.md` — 12_pubkey_kem.rst
- `13_pubkey_agree_review.md` — 13_pubkey_agree.rst
- `14_pubkey_sig_review.md` — 14_pubkey_sig.rst
- `15_rng_review.md` — 15_rng.rst
- `18_tpm_review.md` — 18_tpm.rst

## Test Cases by File

### 01_aead.rst (AEAD Modes)
- `SUBSTANTIVE_FIX` AEAD-1
- `SUBSTANTIVE_FIX` AEAD-2
- `MINOR_FIX` AEAD-3
- `CONFIRMED_NO_CHANGE` AEAD-GCM-1

### 02_cert_store.rst (Certificate Store)
- `CONFIRMED_NO_CHANGE` CERTSTOR-ISR-1
- `MINOR_FIX` CERTSTOR-REV-1
- `CONFIRMED_NO_CHANGE` CERTSTOR-SDN-1
- `SUBSTANTIVE_FIX` CERTSTOR-FAC-1
- `SUBSTANTIVE_FIX` CERTSTOR-SCH-1
- `CONFIRMED_NO_CHANGE` CERTSTOR-SYSTEM-1
- `MINOR_FIX` CERTSTOR-SYSTEM-2
- `CONFIRMED_NO_CHANGE` CERTSTOR-SYSTEM-3
- `CONFIRMED_NO_CHANGE` CERTSTOR-SYSTEM-4
- `CONFIRMED_NO_CHANGE` CERTSTOR-SYSTEM-5

### 03_block_ciphers.rst (Block Ciphers)
- `SUBSTANTIVE_FIX` BLOCK-1
- `SUBSTANTIVE_FIX` BLOCK-2
- `SUBSTANTIVE_FIX` BLOCK-3
- `MINOR_FIX` BLOCK-AES-2

### 04_entropy_srcs.rst (Entropy Sources)
- `SUBSTANTIVE_FIX` ENTROPY-1

### 05_hash.rst (Hash Functions)
- `SUBSTANTIVE_FIX` HASH-1
- `SUBSTANTIVE_FIX` HASH-2
- `SUBSTANTIVE_FIX` HASH-3
- `SUBSTANTIVE_FIX` HASH-4
- `SUBSTANTIVE_FIX` HASH-MD5-1
- `SUBSTANTIVE_FIX` HASH-SHA1-1
- `SUBSTANTIVE_FIX` HASH-SHA224-1
- `SUBSTANTIVE_FIX` HASH-SHA256-1
- `SUBSTANTIVE_FIX` HASH-SHA384-1
- `SUBSTANTIVE_FIX` HASH-SHA512-1
- `SUBSTANTIVE_FIX` HASH-SHA512-256-1
- `SUBSTANTIVE_FIX` HASH-SHA3-224-1
- `SUBSTANTIVE_FIX` HASH-SHA3-256-1
- `SUBSTANTIVE_FIX` HASH-SHA3-384-1
- `SUBSTANTIVE_FIX` HASH-SHA3-512-1
- `SUBSTANTIVE_FIX` HASH-SHAKE-128-128
- `SUBSTANTIVE_FIX` HASH-BLAKE2B-384
- `SUBSTANTIVE_FIX` H-PHASH-1
- `SUBSTANTIVE_FIX` H-PHASH-2

### 06_kdf.rst (Key Derivation Functions)
- `MINOR_FIX` KDF-1
- `CONFIRMED_NO_CHANGE` KDF-KDF1-1
- `CONFIRMED_NO_CHANGE` KDF-NISTSP800-108-CTR-1
- `CONFIRMED_NO_CHANGE` KDF-NISTSP800-108-FB-1
- `CONFIRMED_NO_CHANGE` KDF-NISTSP800-108-PI-1
- `CONFIRMED_NO_CHANGE` KDF-TLS1-PRF-1
- `CONFIRMED_NO_CHANGE` KDF-TLS12-PRF-1

### 07_mac.rst (Message Authentication Codes)
- `MINOR_FIX` MAC-1
- `SUBSTANTIVE_FIX` MAC-2
- `MINOR_FIX` MAC-CMAC-1
- `MINOR_FIX` MAC-HMAC-1
- `MINOR_FIX` MAC-GMAC-1
- `MINOR_FIX` MAC-KMAC-1

### 08_modes_of_operation.rst (Modes of Operation)
- `SUBSTANTIVE_FIX` MODE-1
- `SUBSTANTIVE_FIX` MODE-2
- `CONFIRMED_NO_CHANGE` MODE-CBC-1
- `CONFIRMED_NO_CHANGE` MODE-CTS-1
- `SUBSTANTIVE_FIX` MODE-CTR-1

### 09_pbkdf.rst (Password-Based Key Derivation)
- `SUBSTANTIVE_FIX` PBKDF-1
- `MINOR_FIX` PBKDF-PBKDF2-1
- `CONFIRMED_NO_CHANGE` PBKDF-ARGON-1

### 10_pkcs11.rst (PKCS#11)
- `CONFIRMED_NO_CHANGE` PKCS11-MODULE-1
- `CONFIRMED_NO_CHANGE` PKCS11-MODULE-2
- `CONFIRMED_NO_CHANGE` PKCS11-MODULE-3
- `CONFIRMED_NO_CHANGE` PKCS11-MODULE-4
- `MINOR_FIX` PKCS11-MODULE-5
- `CONFIRMED_NO_CHANGE` PKCS11-MODULE-6
- `CONFIRMED_NO_CHANGE` PKCS11-SLOT-1
- `CONFIRMED_NO_CHANGE` PKCS11-SLOT-2
- `CONFIRMED_NO_CHANGE` PKCS11-SLOT-3
- `CONFIRMED_NO_CHANGE` PKCS11-SLOT-4
- `CONFIRMED_NO_CHANGE` PKCS11-SLOT-5
- `CONFIRMED_NO_CHANGE` PKCS11-SLOT-6
- `CONFIRMED_NO_CHANGE` PKCS11-SLOT-7
- `CONFIRMED_NO_CHANGE` PKCS11-SESSION-1
- `CONFIRMED_NO_CHANGE` PKCS11-SESSION-2
- `CONFIRMED_NO_CHANGE` PKCS11-SESSION-3
- `CONFIRMED_NO_CHANGE` PKCS11-SESSION-4
- `CONFIRMED_NO_CHANGE` PKCS11-SESSION-5
- `CONFIRMED_NO_CHANGE` PKCS11-SESSION-6
- `CONFIRMED_NO_CHANGE` PKCS11-SESSION-7
- `MINOR_FIX` PKCS11-SESSION-8
- `CONFIRMED_NO_CHANGE` PKCS11-RSA-1
- `CONFIRMED_NO_CHANGE` PKCS11-RSA-2
- `MINOR_FIX` PKCS11-RSA-3 _(review: "decryption key" should be "encryption key"; `set_encrypt(true)` in source)_
- `CONFIRMED_NO_CHANGE` PKCS11-RSA-4
- `CONFIRMED_NO_CHANGE` PKCS11-RSA-5
- `CONFIRMED_NO_CHANGE` PKCS11-RSA-6
- `MINOR_FIX` PKCS11-RSA-7
- `MINOR_FIX` PKCS11-RSA-8
- `CONFIRMED_NO_CHANGE` PKCS11-RSA-9
- `CONFIRMED_NO_CHANGE` PKCS11-RSA-10
- `CONFIRMED_NO_CHANGE` PKCS11-RSA-11
- `CONFIRMED_NO_CHANGE` PKCS11-RSA-12
- `CONFIRMED_NO_CHANGE` PKCS11-RSA-13
- `CONFIRMED_NO_CHANGE` PKCS11-ECDSA-1
- `CONFIRMED_NO_CHANGE` PKCS11-ECDSA-2
- `CONFIRMED_NO_CHANGE` PKCS11-ECDSA-3
- `MINOR_FIX` PKCS11-ECDSA-4 _(review: export only checked for success, not byte-compared with original)_
- `MINOR_FIX` PKCS11-ECDSA-5
- `MINOR_FIX` PKCS11-ECDSA-6
- `SUBSTANTIVE_FIX` PKCS11-ECDSA-7
- `CONFIRMED_NO_CHANGE` PKCS11-ECDH-1
- `CONFIRMED_NO_CHANGE` PKCS11-ECDH-2
- `CONFIRMED_NO_CHANGE` PKCS11-ECDH-3
- `CONFIRMED_NO_CHANGE` PKCS11-ECDH-4
- `CONFIRMED_NO_CHANGE` PKCS11-ECDH-5
- `CONFIRMED_NO_CHANGE` PKCS11-ECDH-6
- `CONFIRMED_NO_CHANGE` PKCS11-ECDH-7
- `CONFIRMED_NO_CHANGE` PKCS11-RNG-1
- `CONFIRMED_NO_CHANGE` PKCS11-RNG-2
- `MINOR_FIX` PKCS11-RNG-3
- `MINOR_FIX` PKCS11-X509-1
- `CONFIRMED_NO_CHANGE` PKCS11-MGMT-1
- `CONFIRMED_NO_CHANGE` PKCS11-MGMT-2
- `CONFIRMED_NO_CHANGE` PKCS11-MGMT-3
- `CONFIRMED_NO_CHANGE` PKCS11-MGMT-4

### 11_pubkey_enc.rst (Public Key Encryption)
- `CONFIRMED_NO_CHANGE` PKENC-DLIES-1
- `SUBSTANTIVE_FIX` PKENC-DLIES-2
- `SUBSTANTIVE_FIX` PKENC-ECIES-1
- `CONFIRMED_NO_CHANGE` PKENC-ECIES-2
- `CONFIRMED_NO_CHANGE` PKENC-RSAES-1
- `CONFIRMED_NO_CHANGE` PKENC-RSAES-2
- `CONFIRMED_NO_CHANGE` PKENC-RSAES-3

### 12_pubkey_kem.rst (Public Key KEM)
- `CONFIRMED_NO_CHANGE` PKENC-CMCE-1
- `CONFIRMED_NO_CHANGE` PKENC-CMCE-2
- `CONFIRMED_NO_CHANGE` PKENC-FRODO-1
- `SUBSTANTIVE_FIX` PKENC-FRODO-2
- `CONFIRMED_NO_CHANGE` PKENC-ML-KEM-1
- `CONFIRMED_NO_CHANGE` PKENC-ML-KEM-2
- `SUBSTANTIVE_FIX` PKENC-ML-KEM-3
- `MINOR_FIX` PKENC-ML-KEM-4
- `CONFIRMED_NO_CHANGE` PKENC-ML-KEM-5
- `CONFIRMED_NO_CHANGE` PKENC-ML-KEM-6
- `CONFIRMED_NO_CHANGE` PKENC-RSAKEM-1

### 13_pubkey_agree.rst (Public Key Agreement)
- `CONFIRMED_NO_CHANGE` KA-KEY-1
- `CONFIRMED_NO_CHANGE` KA-KEY-2
- `CONFIRMED_NO_CHANGE` KA-KEY-3
- `CONFIRMED_NO_CHANGE` KA-KEY-4
- `CONFIRMED_NO_CHANGE` KA-KEY-5
- `MINOR_FIX` KA-KEY-6
- `CONFIRMED_NO_CHANGE` KA-DH-1
- `CONFIRMED_NO_CHANGE` KA-DH-2
- `CONFIRMED_NO_CHANGE` KA-DH-3
- `CONFIRMED_NO_CHANGE` KA-KEY-DH-1
- `CONFIRMED_NO_CHANGE` KA-KEY-DH-INVALID-1
- `CONFIRMED_NO_CHANGE` KA-ECDH-1
- `MINOR_FIX` KA-KEY-ECDH-1

### 14_pubkey_sig.rst (Public Key Signatures)
- `CONFIRMED_NO_CHANGE` PKSIG-1
- `CONFIRMED_NO_CHANGE` PKSIG-2
- `CONFIRMED_NO_CHANGE` PKSIG-3
- `CONFIRMED_NO_CHANGE` PKSIG-4
- `CONFIRMED_NO_CHANGE` PKSIG-KEY-1
- `CONFIRMED_NO_CHANGE` PKSIG-KEY-2
- `CONFIRMED_NO_CHANGE` PKSIG-KEY-3
- `CONFIRMED_NO_CHANGE` PKSIG-KEY-4
- `CONFIRMED_NO_CHANGE` PKSIG-KEY-5
- `CONFIRMED_NO_CHANGE` PKSIG-KEY-6
- `MINOR_FIX` PKSIG-ML-DSA-1
- `CONFIRMED_NO_CHANGE` PKSIG-ML-DSA-2
- `MINOR_FIX` PKSIG-ML-DSA-3
- `CONFIRMED_NO_CHANGE` PKSIG-DSA-1
- `MINOR_FIX` PKSIG-DSA-2
- `CONFIRMED_NO_CHANGE` PKSIG-DSA-4
- `CONFIRMED_NO_CHANGE` PKSIG-KEY-DSA-1
- `CONFIRMED_NO_CHANGE` PKSIG-ECDSA-1
- `CONFIRMED_NO_CHANGE` PKSIG-ECDSA-2
- `CONFIRMED_NO_CHANGE` PKSIG-ECDSA-4
- `CONFIRMED_NO_CHANGE` PKSIG-KEY-ECDSA-1
- `CONFIRMED_NO_CHANGE` PKSIG-PUBKEY-VAL-ECDSA-1
- `MINOR_FIX` PKSIG-ECGDSA-1
- `CONFIRMED_NO_CHANGE` PKSIG-ECGDSA-2
- `CONFIRMED_NO_CHANGE` PKSIG-KEY-ECGDSA-1
- `CONFIRMED_NO_CHANGE` PKSIG-ECKCDSA-1
- `CONFIRMED_NO_CHANGE` PKSIG-ECKCDSA-2
- `MINOR_FIX` PKSIG-KEY-ECKCDSA-1 _(mislabelled as PKSIG-KEY-ECDSA-1 in RST — fix needed)_
- `CONFIRMED_NO_CHANGE` PKSIG-HSS/LMS-1
- `CONFIRMED_NO_CHANGE` PKSIG-HSS/LMS-2
- `CONFIRMED_NO_CHANGE` PKSIG-RSA-1
- `MINOR_FIX` PKSIG-RSA-2
- `CONFIRMED_NO_CHANGE` PKSIG-KEY-RSA-1
- `CONFIRMED_NO_CHANGE` PKSIG-SLH-DSA-1
- `CONFIRMED_NO_CHANGE` PKSIG-SLH-DSA-2
- `CONFIRMED_NO_CHANGE` PKSIG-SLH-DSA-3
- `CONFIRMED_NO_CHANGE` PKSIG-XMSS-1
- `MINOR_FIX` PKSIG-XMSS-2
- `MINOR_FIX` PKSIG-XMSS-3 _(typo as PKCS-XMSS-3 in RST — fix needed)_

### 15_rng.rst (Random Number Generators)
- `CONFIRMED_NO_CHANGE` RNG-HMAC-DRBG-1
- `MINOR_FIX` RNG-HMAC-DRBG-2 _(file reference: test_rngs.cpp → test_rng_behavior.cpp)_
- `MINOR_FIX` RNG-HMAC-DRBG-3 _(file reference: test_rngs.cpp → test_rng_behavior.cpp)_
- `MINOR_FIX` RNG-HMAC-DRBG-4 _(file reference: test_rngs.cpp → test_rng_behavior.cpp)_
- `SUBSTANTIVE_FIX` RNG-AUTO-RNG-1
- `MINOR_FIX` RNG-SYS-RNG-1 _(file reference: test_rngs.cpp → test_rng_behavior.cpp)_

### 18_tpm.rst (TPM)
> **Note:** TPM-session-1, TPM-session-2, TPM-session-3 exist in the RST but were
> missing from the original checklist. Added here.
- `CONFIRMED_NO_CHANGE` TPM-session-1
- `CONFIRMED_NO_CHANGE` TPM-session-2
- `CONFIRMED_NO_CHANGE` TPM-session-3
- `CONFIRMED_NO_CHANGE` TPM-RNG-1
- `CONFIRMED_NO_CHANGE` TPM-RNG-2
- `CONFIRMED_NO_CHANGE` TPM-RNG-3
- `MINOR_FIX` TPM-RSA-1
- `CONFIRMED_NO_CHANGE` TPM-RSA-2
- `MINOR_FIX` TPM-RSA-3
- `MINOR_FIX` TPM-RSA-4
- `MINOR_FIX` TPM-ECDSA-1
- `CONFIRMED_NO_CHANGE` TPM-ECDSA-2
- `MINOR_FIX` TPM-ECDSA-3

---

## New Tests Found in Botan 3.12.0 (Not Yet in Spec)

These test cases exist in Botan 3.12.0 source but are not documented in the test spec.
See individual review files for full details.

| Status | Category | New Test(s) |
|--------|----------|-------------|
| `MISSING_TEST` | Block Ciphers | `BlockCipher_ParallelOp_Test` (bc_parop) — SIMD/parallel vs. sequential equivalence |
| `MISSING_TEST` | Hash | `Invalid_Hash_Name_Tests`, `hash_truncation_negative_tests`, 15+ additional hash algorithm KATs |
| `MISSING_TEST` | KDF | KDF1 (X9.63), KDF2, SP800-56A, X9.42 PRF, HKDF-Expand-Label |
| `MISSING_TEST` | MAC | BLAKE2b-MAC, Poly1305, SipHash, X9.19 MAC |
| `MISSING_TEST` | Modes | CFB, XTS, CTR cipher mode, IV carry-over tests, ChaCha20/OFB/RC4/Salsa20/SHAKE stream ciphers |
| `MISSING_TEST` | PBKDF | Bcrypt-PBKDF, Scrypt, Pwdhash tuning tests, PGP S2K |
| `MISSING_TEST` | Cert Store | `test_certstor_load_allcert` and other x509 lookup tests |
| `MISSING_TEST` | RNG | `ChaCha_RNG_Unit_Tests`, `processor_rng`, `hmac_drbg_multiple_requests` |
| `MISSING_TEST` | TPM | `test_tpm2_hash`, `test_tpm2_properties`, `test_tpm2_context`, `test_external_tpm2_context` |
| `MISSING_TEST` | PubKey Enc | `dlies_unit`, `rsa_blinding`, `rsa_decrypt_or_random`, `ecies` (non-ISO) |
| `OUT_OF_SCOPE_DECISION` | KEM | `cmce_generic_keygen` — generic `PK_Key_Generation_Test`; .rst explicitly excludes generic tests for Classic McEliece |
| `OUT_OF_SCOPE_DECISION` | KEM | `frodo_keygen` — generic `PK_Key_Generation_Test`; .rst explicitly excludes generic tests for FrodoKEM |
| `OUT_OF_SCOPE_DECISION` | KEM | `kyber_keygen` — generic `PK_Key_Generation_Test`; .rst explicitly excludes generic tests for ML-KEM/Kyber |
| `OUT_OF_SCOPE_DECISION` | KEM | `cmce_utility` — five internal field-arithmetic/RNG unit tests; policy decision whether internal details belong in normative spec |
| `OUT_OF_SCOPE_DECISION` | KEM | `kyber_helpers` — compress/decompress internal function tests; policy decision whether implementation internals belong in normative spec |
| `MISSING_TEST` | KEM | `ecdh_all_groups` — full regression sweep across all named EC groups |
| `MISSING_TEST` | Signatures | `ml_dsa_verify`, ECDSA all-groups/DER/key-recovery, RSA PSS/blinding/bad-RNG, HSS-LMS state/api, SLH-DSA keygen, XMSS keygen/statefulness |
| `MISSING_TEST` | PKCS#11 | Full `pkcs11-object` group (5 tests), PKCS11-SESSION-9 (`test_session_info`), PKCS11-ECDSA-8 (`test_ecdsa_curve_import`) |

---

## Summary

- **Total test specification files reviewed:** 16
- **Total test cases reviewed:** 204 (includes 3 TPM-session entries added during review)
- **`CONFIRMED_NO_CHANGE`:** 119
- **`MINOR_FIX`:** 47
- **`SUBSTANTIVE_FIX`:** 38
- **`MISSING_TEST`:** ~50+ (see New Tests table above)
- **`OUT_OF_SCOPE_DECISION`:** 5 (KEM generic/internal tests)

### Results by File

| File | Tests | CONFIRMED_NO_CHANGE | MINOR_FIX | SUBSTANTIVE_FIX |
|------|-------|---------------------|-----------|-----------------|
| 01_aead.rst | 4 | 1 | 1 | 2 |
| 02_cert_store.rst | 10 | 6 | 2 | 2 |
| 03_block_ciphers.rst | 4 | 0 | 1 | 3 |
| 04_entropy_srcs.rst | 1 | 0 | 0 | 1 |
| 05_hash.rst | 19 | 0 | 0 | 19 |
| 06_kdf.rst | 7 | 6 | 1 | 0 |
| 07_mac.rst | 6 | 0 | 5 | 1 |
| 08_modes_of_operation.rst | 5 | 2 | 0 | 3 |
| 09_pbkdf.rst | 3 | 1 | 1 | 1 |
| 10_pkcs11.rst | 56 | 45 | 10 | 1 |
| 11_pubkey_enc.rst | 7 | 5 | 0 | 2 |
| 12_pubkey_kem.rst | 11 | 8 | 1 | 2 |
| 13_pubkey_agree.rst | 13 | 11 | 2 | 0 |
| 14_pubkey_sig.rst | 39 | 31 | 8 | 0 |
| 15_rng.rst | 6 | 1 | 4 | 1 |
| 18_tpm.rst | 13 | 8 | 5 | 0 |
| **Total** | **204** | **125** | **41** | **38** |

## Notes

- Files 16_tls.rst and 17_x509.rst were excluded (need separate handling)
- File 90_valgrind_sca.rst does not contain standard test case tables
- The `15_rng.rst` spec references `test_rngs.cpp` but the actual RNG unit tests in
  Botan 3.12.0 are in `src/tests/test_rng_behavior.cpp`; `test_rngs.cpp` is a
  test-helper file only
- All detailed findings and proposed text changes are in `docs/testspec/review/`
- `PKSIG-KEY-RSA-1` was previously marked 🔄 in this checklist with a note about an RST
  label mismatch; the review file (`14_pubkey_sig_review.md`) independently confirmed it
  as ✅ CONFIRMED — the RST label issue should be tracked separately
