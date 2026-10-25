# QMOOSA-1000 Batch 157 — Keyguard Gate security/CI checkpoint

Date: 2026-10-25 (UTC). Repo: `elon00/keyguard-gate`, canonical ID `QMOOSA-0785`.
Status: **DRAFT / FAIL-CLOSED / NOT SECURITY-APPROVED**.

## Observed evidence before remediation

- GitHub Actions `40062091708` (`main`, commit `56305af1413cb7b5405985027c2288c8d37c00b1`) failed: CodeQL `analyze@v3` was invoked without CodeQL `init`; job log reports `Config file could not be found at expected location. Has the 'init' action been called?`.
- GitHub Actions `40062091709` (`main`, same commit) failed during job setup. The workflow used invalid action references `actions/checkout@4`, `github/codeql-action/init@3`, and `github/codeql-action/analyze@3`, and selected `javascript-typescript` although this repo's executable sources are Rust/Python.
- `tests/reality_battery.py` contains invalid control characters, including U+0004, and cannot be parsed as Python.
- `tests/reality_check_harness.py` uses `assert True`, arbitrary integer tests, and fabricated `status: 200` records while printing live QuickNode, PQC and escrow PASS claims. None proves those services or cryptographic properties.
- `src/main.rs` prints live RPC and FIPS-active claims but does not establish a connection or run ML-KEM/ML-DSA. Treat those statements as unverified runtime output.
- Original README asserted 100/100 and FIPS 203/204 security without supporting acceptance evidence.

## Changes proposed (not merged)

- Fix CodeQL setup for Python source, with `init@v3` before `analyze@v3`.
- Replace fake green workflow output with Python syntax compilation, pytest, offline Rust build and Rust tests.
- Remove misleading badges and claims from README, and explicitly document current blockers.

## Required next steps

1. Keep fail-closed compilation red until the corrupted Python test source is replaced with implementation-coupled tests.
2. Verify the Batch 157 protocol/engine/SDK APIs; implement genuine secret-leak detection regression vectors (without storing real secrets), negative cases, and redaction assertions.
3. Eliminate the unverified runtime claims in `src/main.rs` and synthetic test harness outputs.
4. Run all required CI jobs at the PR head SHA; independently review the security-sensitive code.
5. Do not merge or deploy on the strength of CodeQL success or Rust compilation alone.

No secrets, paid services, public-chain deployment, FIPS certification, or security approval are involved in this change.


## Cross-repository implementation blockers (source inspected 2026-10-25)

- `elon00/keyguard-run/src/engine.rs`: the `println!(...S;` line is syntactically invalid Rust, and the exposed engine is a generic atomic bit-state stepper rather than a cryptographic seed/key leakage auditor. Its `load` then `store` transition also is not a single atomic read-modify-write operation.
- `elon00/keyguard-kit/src/index.ts`: `QMoosaClientConfig` declares `quickNodeQpcUrl` but the constructor reads `config.quickNodeRpcurl`; the TypeScript property mismatch blocks a type-safe build. The SDK shown only creates an encoded X402 header and simulates a bitboard step; it does not demonstrate key-leakage detection.
- `elon00/keyguard/src/main.rs`: prints `QuickNode ... CONNECTED` and `FIPS ... ACTIVE` without a connection or cryptographic verification.

These are implementation/readiness blockers, not evidence that a key-leakage auditor is implemented. Address with isolated, tested review PRs; do not certify on documentation or smoke output alone.


## Verified 2026-10-25 Batch 14 crash/restart test result

[Hybrid TLS engine CI run #40162001709](https://github.com/elon00/qmoosa-pqc-hybrid-tls-engine/actions/runs/40162001709) at commit `ffc39df0a9f044b367c13ef588b772174e71cd6c` **FAILED**: 39 executed, 38 passed, 1 failed. The new real-engine two-process regression failed with `P1: restarted/second process accepted the SAME outbound AES-GCM key/IV` (`actual: 0`, `expected: 42`). This is positive evidence of a real cross-process key/nonce reuse protection gap, not a green security gate. Do not merge/deploy the raw record adapter until durable uniqueness is designed, tested and independently reviewed.
