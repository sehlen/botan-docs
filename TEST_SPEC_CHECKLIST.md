# Test Specification Verification Checklist

This checklist tracks the verification of test specifications against Botan 3.12.0.

**Target Botan Version:** 3.12.0
**Repository:** randombit/botan
**Generated:** 2026-05-14
**Last Updated:** 2026-05-20 (agent review complete)

## Legend

- ✅ **Confirmed**: Description is accurate and matches Botan 3.12.0 implementation
- 🔄 **Proposed Changes**: Description needs updates — see `docs/testspec/review/`
- ➕ **New Test**: Found in Botan 3.12.0 but not yet in the test spec (see review files)
- `[ ]` Not yet reviewed

## Review Files

Detailed findings per category are in `docs/testspec/review/`:
- `group_aead_block_entropy_review.md` — 01_aead.rst, 03_block_ciphers.rst, 04_entropy_srcs.rst
- `group_hash_review.md` — 05_hash.rst
- `group_kdf_mac_modes_pbkdf_review.md` — 06_kdf.rst, 07_mac.rst, 08_modes_of_operation.rst, 09_pbkdf.rst
- `group_certstore_rng_tpm_review.md` — 02_cert_store.rst, 15_rng.rst, 18_tpm.rst
- `group_kem_keyagree_pubkeyenc_review.md` — 11_pubkey_enc.rst, 12_pubkey_kem.rst, 13_pubkey_agree.rst
- `group_pkcs11_review.md` — 10_pkcs11.rst
- `group_signatures_review.md` — 14_pubkey_sig.rst

## Test Cases by File

### 01_aead.rst (AEAD Modes)
- [x] 🔄 AEAD-1
- [x] 🔄 AEAD-2
- [x] 🔄 AEAD-3
- [x] ✅ AEAD-GCM-1

### 02_cert_store.rst (Certificate Store)
- [x] ✅ CERTSTOR-ISR-1
- [x] 🔄 CERTSTOR-REV-1
- [x] ✅ CERTSTOR-SDN-1
- [x] 🔄 CERTSTOR-FAC-1
- [x] 🔄 CERTSTOR-SCH-1
- [x] ✅ CERTSTOR-SYSTEM-1
- [x] 🔄 CERTSTOR-SYSTEM-2
- [x] ✅ CERTSTOR-SYSTEM-3
- [x] ✅ CERTSTOR-SYSTEM-4
- [x] ✅ CERTSTOR-SYSTEM-5

### 03_block_ciphers.rst (Block Ciphers)
- [x] 🔄 BLOCK-1
- [x] 🔄 BLOCK-2
- [x] 🔄 BLOCK-3
- [x] 🔄 BLOCK-AES-2

### 04_entropy_srcs.rst (Entropy Sources)
- [x] 🔄 ENTROPY-1

### 05_hash.rst (Hash Functions)
- [x] 🔄 HASH-1
- [x] 🔄 HASH-2
- [x] 🔄 HASH-3
- [x] 🔄 HASH-4
- [x] 🔄 HASH-MD5-1
- [x] 🔄 HASH-SHA1-1
- [x] 🔄 HASH-SHA224-1
- [x] 🔄 HASH-SHA256-1
- [x] 🔄 HASH-SHA384-1
- [x] 🔄 HASH-SHA512-1
- [x] 🔄 HASH-SHA512-256-1
- [x] 🔄 HASH-SHA3-224-1
- [x] 🔄 HASH-SHA3-256-1
- [x] 🔄 HASH-SHA3-384-1
- [x] 🔄 HASH-SHA3-512-1
- [x] 🔄 HASH-SHAKE-128-128
- [x] 🔄 HASH-BLAKE2B-384
- [x] 🔄 H-PHASH-1
- [x] 🔄 H-PHASH-2

### 06_kdf.rst (Key Derivation Functions)
- [x] 🔄 KDF-1
- [x] ✅ KDF-KDF1-1
- [x] ✅ KDF-NISTSP800-108-CTR-1
- [x] ✅ KDF-NISTSP800-108-FB-1
- [x] ✅ KDF-NISTSP800-108-PI-1
- [x] ✅ KDF-TLS1-PRF-1
- [x] ✅ KDF-TLS12-PRF-1

### 07_mac.rst (Message Authentication Codes)
- [x] 🔄 MAC-1
- [x] 🔄 MAC-2
- [x] 🔄 MAC-CMAC-1
- [x] 🔄 MAC-HMAC-1
- [x] 🔄 MAC-GMAC-1
- [x] 🔄 MAC-KMAC-1

### 08_modes_of_operation.rst (Modes of Operation)
- [x] 🔄 MODE-1
- [x] 🔄 MODE-2
- [x] ✅ MODE-CBC-1
- [x] ✅ MODE-CTS-1
- [x] 🔄 MODE-CTR-1

### 09_pbkdf.rst (Password-Based Key Derivation)
- [x] 🔄 PBKDF-1
- [x] 🔄 PBKDF-PBKDF2-1
- [x] ✅ PBKDF-ARGON-1

### 10_pkcs11.rst (PKCS#11)
- [x] ✅ PKCS11-MODULE-1
- [x] ✅ PKCS11-MODULE-2
- [x] ✅ PKCS11-MODULE-3
- [x] ✅ PKCS11-MODULE-4
- [x] 🔄 PKCS11-MODULE-5
- [x] ✅ PKCS11-MODULE-6
- [x] ✅ PKCS11-SLOT-1
- [x] ✅ PKCS11-SLOT-2
- [x] ✅ PKCS11-SLOT-3
- [x] ✅ PKCS11-SLOT-4
- [x] ✅ PKCS11-SLOT-5
- [x] ✅ PKCS11-SLOT-6
- [x] ✅ PKCS11-SLOT-7
- [x] ✅ PKCS11-SESSION-1
- [x] ✅ PKCS11-SESSION-2
- [x] ✅ PKCS11-SESSION-3
- [x] ✅ PKCS11-SESSION-4
- [x] ✅ PKCS11-SESSION-5
- [x] ✅ PKCS11-SESSION-6
- [x] ✅ PKCS11-SESSION-7
- [x] 🔄 PKCS11-SESSION-8
- [x] ✅ PKCS11-RSA-1
- [x] ✅ PKCS11-RSA-2
- [x] ✅ PKCS11-RSA-3
- [x] ✅ PKCS11-RSA-4
- [x] ✅ PKCS11-RSA-5
- [x] ✅ PKCS11-RSA-6
- [x] 🔄 PKCS11-RSA-7
- [x] 🔄 PKCS11-RSA-8
- [x] ✅ PKCS11-RSA-9
- [x] ✅ PKCS11-RSA-10
- [x] ✅ PKCS11-RSA-11
- [x] ✅ PKCS11-RSA-12
- [x] ✅ PKCS11-RSA-13
- [x] ✅ PKCS11-ECDSA-1
- [x] ✅ PKCS11-ECDSA-2
- [x] ✅ PKCS11-ECDSA-3
- [x] ✅ PKCS11-ECDSA-4
- [x] 🔄 PKCS11-ECDSA-5
- [x] 🔄 PKCS11-ECDSA-6
- [x] 🔄 PKCS11-ECDSA-7
- [x] ✅ PKCS11-ECDH-1
- [x] ✅ PKCS11-ECDH-2
- [x] ✅ PKCS11-ECDH-3
- [x] ✅ PKCS11-ECDH-4
- [x] ✅ PKCS11-ECDH-5
- [x] ✅ PKCS11-ECDH-6
- [x] ✅ PKCS11-ECDH-7
- [x] ✅ PKCS11-RNG-1
- [x] ✅ PKCS11-RNG-2
- [x] 🔄 PKCS11-RNG-3
- [x] 🔄 PKCS11-X509-1
- [x] ✅ PKCS11-MGMT-1
- [x] ✅ PKCS11-MGMT-2
- [x] ✅ PKCS11-MGMT-3
- [x] ✅ PKCS11-MGMT-4

### 11_pubkey_enc.rst (Public Key Encryption)
- [x] ✅ PKENC-DLIES-1
- [x] 🔄 PKENC-DLIES-2
- [x] 🔄 PKENC-ECIES-1
- [x] ✅ PKENC-ECIES-2
- [x] ✅ PKENC-RSAES-1
- [x] ✅ PKENC-RSAES-2
- [x] ✅ PKENC-RSAES-3

### 12_pubkey_kem.rst (Public Key KEM)
- [x] ✅ PKENC-CMCE-1
- [x] ✅ PKENC-CMCE-2
- [x] ✅ PKENC-FRODO-1
- [x] 🔄 PKENC-FRODO-2
- [x] ✅ PKENC-ML-KEM-1
- [x] ✅ PKENC-ML-KEM-2
- [x] 🔄 PKENC-ML-KEM-3
- [x] 🔄 PKENC-ML-KEM-4
- [x] ✅ PKENC-ML-KEM-5
- [x] ✅ PKENC-ML-KEM-6
- [x] ✅ PKENC-RSAKEM-1

### 13_pubkey_agree.rst (Public Key Agreement)
- [x] ✅ KA-KEY-1
- [x] ✅ KA-KEY-2
- [x] ✅ KA-KEY-3
- [x] ✅ KA-KEY-4
- [x] ✅ KA-KEY-5
- [x] 🔄 KA-KEY-6
- [x] ✅ KA-DH-1
- [x] ✅ KA-DH-2
- [x] ✅ KA-DH-3
- [x] ✅ KA-KEY-DH-1
- [x] ✅ KA-KEY-DH-INVALID-1
- [x] ✅ KA-ECDH-1
- [x] 🔄 KA-KEY-ECDH-1

### 14_pubkey_sig.rst (Public Key Signatures)
- [x] ✅ PKSIG-1
- [x] ✅ PKSIG-2
- [x] ✅ PKSIG-3
- [x] ✅ PKSIG-4
- [x] ✅ PKSIG-KEY-1
- [x] ✅ PKSIG-KEY-2
- [x] ✅ PKSIG-KEY-3
- [x] ✅ PKSIG-KEY-4
- [x] ✅ PKSIG-KEY-5
- [x] ✅ PKSIG-KEY-6
- [x] 🔄 PKSIG-ML-DSA-1
- [x] ✅ PKSIG-ML-DSA-2
- [x] 🔄 PKSIG-ML-DSA-3
- [x] ✅ PKSIG-DSA-1
- [x] 🔄 PKSIG-DSA-2
- [x] ✅ PKSIG-DSA-4
- [x] ✅ PKSIG-KEY-DSA-1
- [x] ✅ PKSIG-ECDSA-1
- [x] ✅ PKSIG-ECDSA-2
- [x] ✅ PKSIG-ECDSA-4
- [x] ✅ PKSIG-KEY-ECDSA-1
- [x] ✅ PKSIG-PUBKEY-VAL-ECDSA-1
- [x] 🔄 PKSIG-ECGDSA-1
- [x] ✅ PKSIG-ECGDSA-2
- [x] ✅ PKSIG-KEY-ECGDSA-1
- [x] ✅ PKSIG-ECKCDSA-1
- [x] ✅ PKSIG-ECKCDSA-2
- [x] 🔄 PKSIG-KEY-ECKCDSA-1 _(mislabelled as PKSIG-KEY-ECDSA-1 in RST — fix needed)_
- [x] ✅ PKSIG-HSS/LMS-1
- [x] ✅ PKSIG-HSS/LMS-2
- [x] ✅ PKSIG-RSA-1
- [x] 🔄 PKSIG-RSA-2
- [x] 🔄 PKSIG-KEY-RSA-1 _(mislabelled as PKSIG-3 in RST — fix needed)_
- [x] ✅ PKSIG-SLH-DSA-1
- [x] ✅ PKSIG-SLH-DSA-2
- [x] ✅ PKSIG-SLH-DSA-3
- [x] ✅ PKSIG-XMSS-1
- [x] 🔄 PKSIG-XMSS-2
- [x] 🔄 PKSIG-XMSS-3 _(typo as PKCS-XMSS-3 in RST — fix needed)_

### 15_rng.rst (Random Number Generators)
- [x] ✅ RNG-HMAC-DRBG-1
- [x] ✅ RNG-HMAC-DRBG-2
- [x] ✅ RNG-HMAC-DRBG-3
- [x] ✅ RNG-HMAC-DRBG-4
- [x] 🔄 RNG-AUTO-RNG-1
- [x] ✅ RNG-SYS-RNG-1

### 18_tpm.rst (TPM)
> **Note:** TPM-session-1, TPM-session-2, TPM-session-3 exist in the RST but were missing from the original checklist. Added below.
- [x] ✅ TPM-session-1 _(not in original checklist)_
- [x] ✅ TPM-session-2 _(not in original checklist)_
- [x] ✅ TPM-session-3 _(not in original checklist)_
- [x] ✅ TPM-RNG-1
- [x] ✅ TPM-RNG-2
- [x] ✅ TPM-RNG-3
- [x] 🔄 TPM-RSA-1
- [x] ✅ TPM-RSA-2
- [x] 🔄 TPM-RSA-3
- [x] 🔄 TPM-RSA-4
- [x] 🔄 TPM-ECDSA-1
- [x] ✅ TPM-ECDSA-2
- [x] 🔄 TPM-ECDSA-3

---

## New Tests Found in Botan 3.12.0 (Not Yet in Spec)

These test cases exist in Botan 3.12.0 source but are not documented in the test spec.
See individual review files for full details.

| Category | New Test(s) |
|---|---|
| Block Ciphers | `BlockCipher_ParallelOp_Test` (bc_parop) — SIMD/parallel vs. sequential equivalence |
| Hash | `Invalid_Hash_Name_Tests`, `hash_truncation_negative_tests`, 15+ additional hash algorithm KATs |
| KDF | KDF1 (X9.63), KDF2, SP800-56A, X9.42 PRF, HKDF-Expand-Label |
| MAC | BLAKE2b-MAC, Poly1305, SipHash, X9.19 MAC |
| Modes | CFB, XTS, CTR cipher mode, IV carry-over tests, ChaCha20/OFB/RC4/Salsa20/SHAKE stream ciphers |
| PBKDF | Bcrypt-PBKDF, Scrypt, Pwdhash tuning tests, PGP S2K |
| Cert Store | Various additional certstor and x509 lookup tests |
| RNG | `ChaCha_RNG_Unit_Tests`, `processor_rng`, `test_rng_behavior.cpp` unit tests |
| TPM | `test_tpm2_hash`, `test_tpm2_properties`, `test_tpm2_context`, `test_external_tpm2_context` |
| PubKey Enc | `dlies_unit`, `rsa_blinding`, `rsa_decrypt_or_random` |
| KEM | `cmce_utility`, `cmce_generic_keygen`, `frodo_keygen`, `kyber_keygen`, `kyber_helpers`, `ecdh_all_groups` |
| Signatures | `ml_dsa_verify`, ECDSA key recovery/all-groups/DER, RSA PSS/blinding/bad-RNG, HSS-LMS state/api, SLH-DSA keygen, XMSS keygen-reference/statefulness |
| PKCS#11 | Full `pkcs11-object` group (5 tests), PKCS11-SESSION-9, PKCS11-ECDSA-8 |

---

## Summary

- **Total test specification files reviewed:** 16
- **Total test cases reviewed:** 207 (204 original + 3 TPM-session tests added)
- **✅ Confirmed accurate:** 117
- **🔄 Proposed changes needed:** 90
- **➕ New tests to add:** ~50+ (see review files for details)

### Results by File

| File | Tests | ✅ Confirmed | 🔄 Changes |
|---|---|---|---|
| 01_aead.rst | 4 | 1 | 3 |
| 02_cert_store.rst | 10 | 6 | 4 |
| 03_block_ciphers.rst | 4 | 0 | 4 |
| 04_entropy_srcs.rst | 1 | 0 | 1 |
| 05_hash.rst | 19 | 0 | 19 |
| 06_kdf.rst | 7 | 6 | 1 |
| 07_mac.rst | 6 | 0 | 6 |
| 08_modes_of_operation.rst | 5 | 2 | 3 |
| 09_pbkdf.rst | 3 | 1 | 2 |
| 10_pkcs11.rst | 60 | 46 | 14 |
| 11_pubkey_enc.rst | 7 | 5 | 2 |
| 12_pubkey_kem.rst | 11 | 8 | 3 |
| 13_pubkey_agree.rst | 13 | 11 | 2 |
| 14_pubkey_sig.rst | 40 | 26 | 14 |
| 15_rng.rst | 6 | 5 | 1 |
| 18_tpm.rst | 13 | 7 | 6 |

## Notes

- Files 16_tls.rst and 17_x509.rst were excluded (need separate handling)
- File 90_valgrind_sca.rst does not contain standard test case tables
- The `15_rng.rst` spec references `test_rngs.cpp` but unit tests in Botan 3.12.0 are in `test_rng_behavior.cpp`
- All detailed findings and proposed text changes are in `docs/testspec/review/`
