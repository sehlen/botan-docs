# Test Specification Verification Checklist

This checklist tracks the verification of test specifications against Botan 3.12.0.

**Target Botan Version:** 3.12.0
**Repository:** randombit/botan
**Generated:** 2026-05-14

## Purpose

For each test case below, verify against Botan 3.12.0 source code:
- ✅ **Confirm**: Test description is accurate and matches current implementation
- 🔄 **Propose changes**: Test description needs updates to match current implementation
- ➕ **Add new**: Identify new tests in Botan that need to be documented

## Test Cases by File

### 01_aead.rst (AEAD Modes)
- [ ] AEAD-1
- [ ] AEAD-2
- [ ] AEAD-3
- [ ] AEAD-GCM-1

### 02_cert_store.rst (Certificate Store)
- [ ] CERTSTOR-ISR-1
- [ ] CERTSTOR-REV-1
- [ ] CERTSTOR-SDN-1
- [ ] CERTSTOR-FAC-1
- [ ] CERTSTOR-SCH-1
- [ ] CERTSTOR-SYSTEM-1
- [ ] CERTSTOR-SYSTEM-2
- [ ] CERTSTOR-SYSTEM-3
- [ ] CERTSTOR-SYSTEM-4
- [ ] CERTSTOR-SYSTEM-5

### 03_block_ciphers.rst (Block Ciphers)
- [ ] BLOCK-1
- [ ] BLOCK-2
- [ ] BLOCK-3
- [ ] BLOCK-AES-2

### 04_entropy_srcs.rst (Entropy Sources)
- [ ] ENTROPY-1

### 05_hash.rst (Hash Functions)
- [ ] HASH-1
- [ ] HASH-2
- [ ] HASH-3
- [ ] HASH-4
- [ ] HASH-MD5-1
- [ ] HASH-SHA1-1
- [ ] HASH-SHA224-1
- [ ] HASH-SHA256-1
- [ ] HASH-SHA384-1
- [ ] HASH-SHA512-1
- [ ] HASH-SHA512-256-1
- [ ] HASH-SHA3-224-1
- [ ] HASH-SHA3-256-1
- [ ] HASH-SHA3-384-1
- [ ] HASH-SHA3-512-1
- [ ] HASH-SHAKE-128-128
- [ ] HASH-BLAKE2B-384
- [ ] H-PHASH-1
- [ ] H-PHASH-2

### 06_kdf.rst (Key Derivation Functions)
- [ ] KDF-1
- [ ] KDF-KDF1-1
- [ ] KDF-NISTSP800-108-CTR-1
- [ ] KDF-NISTSP800-108-FB-1
- [ ] KDF-NISTSP800-108-PI-1
- [ ] KDF-TLS1-PRF-1
- [ ] KDF-TLS12-PRF-1

### 07_mac.rst (Message Authentication Codes)
- [ ] MAC-1
- [ ] MAC-2
- [ ] MAC-CMAC-1
- [ ] MAC-HMAC-1
- [ ] MAC-GMAC-1
- [ ] MAC-KMAC-1

### 08_modes_of_operation.rst (Modes of Operation)
- [ ] MODE-1
- [ ] MODE-2
- [ ] MODE-CBC-1
- [ ] MODE-CTS-1
- [ ] MODE-CTR-1

### 09_pbkdf.rst (Password-Based Key Derivation)
- [ ] PBKDF-1
- [ ] PBKDF-PBKDF2-1
- [ ] PBKDF-ARGON-1

### 10_pkcs11.rst (PKCS#11)
- [ ] PKCS11-MODULE-1
- [ ] PKCS11-MODULE-2
- [ ] PKCS11-MODULE-3
- [ ] PKCS11-MODULE-4
- [ ] PKCS11-MODULE-5
- [ ] PKCS11-MODULE-6
- [ ] PKCS11-SLOT-1
- [ ] PKCS11-SLOT-2
- [ ] PKCS11-SLOT-3
- [ ] PKCS11-SLOT-4
- [ ] PKCS11-SLOT-5
- [ ] PKCS11-SLOT-6
- [ ] PKCS11-SLOT-7
- [ ] PKCS11-SESSION-1
- [ ] PKCS11-SESSION-2
- [ ] PKCS11-SESSION-3
- [ ] PKCS11-SESSION-4
- [ ] PKCS11-SESSION-5
- [ ] PKCS11-SESSION-6
- [ ] PKCS11-SESSION-7
- [ ] PKCS11-SESSION-8
- [ ] PKCS11-RSA-1
- [ ] PKCS11-RSA-2
- [ ] PKCS11-RSA-3
- [ ] PKCS11-RSA-4
- [ ] PKCS11-RSA-5
- [ ] PKCS11-RSA-6
- [ ] PKCS11-RSA-7
- [ ] PKCS11-RSA-8
- [ ] PKCS11-RSA-9
- [ ] PKCS11-RSA-10
- [ ] PKCS11-RSA-11
- [ ] PKCS11-RSA-12
- [ ] PKCS11-RSA-13
- [ ] PKCS11-ECDSA-1
- [ ] PKCS11-ECDSA-2
- [ ] PKCS11-ECDSA-3
- [ ] PKCS11-ECDSA-4
- [ ] PKCS11-ECDSA-5
- [ ] PKCS11-ECDSA-6
- [ ] PKCS11-ECDSA-7
- [ ] PKCS11-ECDH-1
- [ ] PKCS11-ECDH-2
- [ ] PKCS11-ECDH-3
- [ ] PKCS11-ECDH-4
- [ ] PKCS11-ECDH-5
- [ ] PKCS11-ECDH-6
- [ ] PKCS11-ECDH-7
- [ ] PKCS11-RNG-1
- [ ] PKCS11-RNG-2
- [ ] PKCS11-RNG-3
- [ ] PKCS11-X509-1
- [ ] PKCS11-MGMT-1
- [ ] PKCS11-MGMT-2
- [ ] PKCS11-MGMT-3
- [ ] PKCS11-MGMT-4

### 11_pubkey_enc.rst (Public Key Encryption)
- [ ] PKENC-DLIES-1
- [ ] PKENC-DLIES-2
- [ ] PKENC-ECIES-1
- [ ] PKENC-ECIES-2
- [ ] PKENC-RSAES-1
- [ ] PKENC-RSAES-2
- [ ] PKENC-RSAES-3

### 12_pubkey_kem.rst (Public Key KEM)
- [ ] PKENC-CMCE-1
- [ ] PKENC-CMCE-2
- [ ] PKENC-FRODO-1
- [ ] PKENC-FRODO-2
- [ ] PKENC-ML-KEM-1
- [ ] PKENC-ML-KEM-2
- [ ] PKENC-ML-KEM-3
- [ ] PKENC-ML-KEM-4
- [ ] PKENC-ML-KEM-5
- [ ] PKENC-ML-KEM-6
- [ ] PKENC-RSAKEM-1

### 13_pubkey_agree.rst (Public Key Agreement)
- [ ] KA-KEY-1
- [ ] KA-KEY-2
- [ ] KA-KEY-3
- [ ] KA-KEY-4
- [ ] KA-KEY-5
- [ ] KA-KEY-6
- [ ] KA-DH-1
- [ ] KA-DH-2
- [ ] KA-DH-3
- [ ] KA-KEY-DH-1
- [ ] KA-KEY-DH-INVALID-1
- [ ] KA-ECDH-1
- [ ] KA-KEY-ECDH-1

### 14_pubkey_sig.rst (Public Key Signatures)
- [ ] PKSIG-1
- [ ] PKSIG-2
- [ ] PKSIG-3
- [ ] PKSIG-4
- [ ] PKSIG-KEY-1
- [ ] PKSIG-KEY-2
- [ ] PKSIG-KEY-3
- [ ] PKSIG-KEY-4
- [ ] PKSIG-KEY-5
- [ ] PKSIG-KEY-6
- [ ] PKSIG-ML-DSA-1
- [ ] PKSIG-ML-DSA-2
- [ ] PKSIG-ML-DSA-3
- [ ] PKSIG-DSA-1
- [ ] PKSIG-DSA-2
- [ ] PKSIG-DSA-4
- [ ] PKSIG-KEY-DSA-1
- [ ] PKSIG-ECDSA-1
- [ ] PKSIG-ECDSA-2
- [ ] PKSIG-ECDSA-4
- [ ] PKSIG-KEY-ECDSA-1
- [ ] PKSIG-PUBKEY-VAL-ECDSA-1
- [ ] PKSIG-ECGDSA-1
- [ ] PKSIG-ECGDSA-2
- [ ] PKSIG-KEY-ECGDSA-1
- [ ] PKSIG-ECKCDSA-1
- [ ] PKSIG-ECKCDSA-2
- [ ] PKSIG-HSS/LMS-1
- [ ] PKSIG-HSS/LMS-2
- [ ] PKSIG-RSA-1
- [ ] PKSIG-RSA-2
- [ ] PKSIG-KEY-RSA-1
- [ ] PKSIG-SLH-DSA-1
- [ ] PKSIG-SLH-DSA-2
- [ ] PKSIG-SLH-DSA-3
- [ ] PKSIG-XMSS-1
- [ ] PKSIG-XMSS-2
- [ ] PKSIG-XMSS-3

### 15_rng.rst (Random Number Generators)
- [ ] RNG-HMAC-DRBG-1
- [ ] RNG-HMAC-DRBG-2
- [ ] RNG-HMAC-DRBG-3
- [ ] RNG-HMAC-DRBG-4
- [ ] RNG-AUTO-RNG-1
- [ ] RNG-SYS-RNG-1

### 18_tpm.rst (TPM)
- [ ] TPM-RNG-1
- [ ] TPM-RNG-2
- [ ] TPM-RNG-3
- [ ] TPM-RSA-1
- [ ] TPM-RSA-2
- [ ] TPM-RSA-3
- [ ] TPM-RSA-4
- [ ] TPM-ECDSA-1
- [ ] TPM-ECDSA-2
- [ ] TPM-ECDSA-3

---

## Summary

- **Total test specification files:** 16
- **Total test cases:** 204
- **Files requiring verification:**
  - 01_aead.rst (4 tests)
  - 02_cert_store.rst (10 tests)
  - 03_block_ciphers.rst (4 tests)
  - 04_entropy_srcs.rst (1 test)
  - 05_hash.rst (19 tests)
  - 06_kdf.rst (7 tests)
  - 07_mac.rst (6 tests)
  - 08_modes_of_operation.rst (5 tests)
  - 09_pbkdf.rst (3 tests)
  - 10_pkcs11.rst (60 tests)
  - 11_pubkey_enc.rst (7 tests)
  - 12_pubkey_kem.rst (11 tests)
  - 13_pubkey_agree.rst (13 tests)
  - 14_pubkey_sig.rst (38 tests)
  - 15_rng.rst (6 tests)
  - 18_tpm.rst (10 tests)

## Notes

- Files 16_tls.rst and 17_x509.rst were excluded from the main checklist (need separate handling)
- File 90_valgrind_sca.rst does not contain standard test case tables
