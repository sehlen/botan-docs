# Test Specification Review Files

This directory contains review findings from automated agents that verified the test specifications against Botan 3.12.0.

## Format

Each review file corresponds to one or more test specification RST files. For each test case reviewed:

- ✅ **CONFIRMED**: Description accurately reflects the current Botan 3.12.0 implementation
- 🔄 **PROPOSED CHANGES**: Description needs updates; proposed new text is included
- ➕ **NEW TEST**: Found in Botan 3.12.0 but not yet in the test spec

## Files

| Review File | Covers |
|---|---|
| group_aead_block_entropy_review.md | 01_aead.rst, 03_block_ciphers.rst, 04_entropy_srcs.rst |
| group_hash_review.md | 05_hash.rst |
| group_kdf_mac_modes_pbkdf_review.md | 06_kdf.rst, 07_mac.rst, 08_modes_of_operation.rst, 09_pbkdf.rst |
| group_certstore_rng_tpm_review.md | 02_cert_store.rst, 15_rng.rst, 18_tpm.rst |
| group_kem_keyagree_pubkeyenc_review.md | 11_pubkey_enc.rst, 12_pubkey_kem.rst, 13_pubkey_agree.rst |
| group_pkcs11_review.md | 10_pkcs11.rst |
| group_signatures_review.md | 14_pubkey_sig.rst |
