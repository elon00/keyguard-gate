# Keyguard Gate — QMOOSA-0785

**Status: prototype / security review required. Not production-ready.**

This repository is the TESTS component of QMOOSA-1000 Batch 157 (Cryptographic Seed & Key Leakage Auditor). Its existing Python reality battery is corrupted and its smoke harness contains assertions that do not verify live RPC connectivity, NIST FIPS conformance, or cryptographic seed/key leakage prevention.

## Local, fail-closed verification

```bash
python -m compileall -q tests
python -m pip install pytest
python -m pytest -q tests -o python_files='*.py'
cargo check --offline
cargo test --offline
```

The Python compilation command is **expected to fail on the existing corrupted test file** until a real implementation-coupled replacement is reviewed. Do not delete, skip, or mock tests to make CI green. Rust compilation or CodeQL success alone is not security approval.

## Acceptance blockers

1. Replace the malformed `tests/reality_battery.py` with tests calling real key-leak detection and redaction APIs (positive, negative, boundary and adversarial cases). Preserve failing tests until real implementations exist.
2. Remove placeholder `assert True`, synthetic `status: 200` and arithmetic-only checks from `tests/reality_check_harness.py`; verify real code paths instead.
3. Validate cross-repository integration with Batch 157 protocol, engine and SDK. Add non-disclosure and deterministic test vectors without exposing actual secrets.
4. Require independent security review, current-HEAD CI evidence and explicit sign-off before any production or deployment-ready claim.

No FIPS validation, live QuickNode integration, production readiness, 100/100 score or deployment is asserted. See `audits/SECURITY_CI_REMEDIATION_2026-10-25.md`.
