# Test Specification Review Files

This directory contains review findings from automated agents that verified the test specifications against Botan 3.12.0.

## Format

Each review file corresponds to exactly one test specification RST file. For each test case reviewed:

- ✅ **CONFIRMED**: Description accurately reflects the current Botan 3.12.0 implementation
- 🔄 **PROPOSED CHANGES**: Description needs updates; proposed new text is included
- ➕ **NEW TEST**: Found in Botan 3.12.0 but not yet in the test spec

## Files

| Review File | RST File |
|---|---|
| 01_aead_review.md | 01_aead.rst |
| 02_cert_store_review.md | 02_cert_store.rst |
| 03_block_ciphers_review.md | 03_block_ciphers.rst |
| 04_entropy_srcs_review.md | 04_entropy_srcs.rst |
| 05_hash_review.md | 05_hash.rst |
| 06_kdf_review.md | 06_kdf.rst |
| 07_mac_review.md | 07_mac.rst |
| 08_modes_of_operation_review.md | 08_modes_of_operation.rst |
| 09_pbkdf_review.md | 09_pbkdf.rst |
| 10_pkcs11_review.md | 10_pkcs11.rst |
| 11_pubkey_enc_review.md | 11_pubkey_enc.rst |
| 12_pubkey_kem_review.md | 12_pubkey_kem.rst |
| 13_pubkey_agree_review.md | 13_pubkey_agree.rst |
| 14_pubkey_sig_review.md | 14_pubkey_sig.rst |
| 15_rng_review.md | 15_rng.rst |
| 18_tpm_review.md | 18_tpm.rst |
