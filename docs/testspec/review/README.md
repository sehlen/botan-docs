# Test Specification Review Files

This directory contains review findings from automated agents that verified
the test specifications against Botan 3.12.0.

## Access Limitation Notice

The review agents ran in a sandboxed environment with firewall restrictions
that blocked access to GitHub MCP/API endpoints and the Botan source tree
via HTTP. All source verification was performed **locally** using the
Botan 3.12.0 source code that was already cloned on disk. Lines marked
"source-verified" were confirmed by reading the local source files directly.
Any finding that could not be verified locally is explicitly marked as
"inferred" or "unverified" in the notes. The overall review is structurally
sound, but individual source-line anchors and registration names should be
cross-checked when applying changes.

## Status Codes

Each reviewed test case carries one of the following statuses:

| Status | Symbol | Meaning |
|--------|--------|---------|
| CONFIRMED_NO_CHANGE | ✅ | Spec accurately matches source; no .rst change required |
| MINOR_FIX | 🔧 | Typo, wording, source-file reference, precondition, or guard issue |
| SUBSTANTIVE_FIX | 🔄 | Steps or expected behavior differ materially from source |
| MISSING_TEST | ➕ | Source test exists but has no spec entry |
| OUT_OF_SCOPE_DECISION | ⚠️ | Source test exists but spec intentionally omits it; rationale documented |

> **Note:** The initial review used a binary confirmed/proposed-changes model.
> These files are being migrated to the five-status model above.
> Until migration is complete, entries may still use the old ✅ / 🔄 symbols.

## Review Files

| Review File | RST File |
|-------------|----------|
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
