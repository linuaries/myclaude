# myclaude Code Review Report
**Date:** 2026-01-03
**Backend:** Codex

## Executive Summary

This review covers the myclaude repository, which appears to be a Claude Code wrapper/tooling project. Several security, performance, and maintainability issues were identified.

---

## Critical Security Issues

### 1. Unverified Binary Download in install.sh
**Location:** `install.sh:25-43`

**Issue:** Downloads the wrapper from a moving "latest" tag to a fixed `/tmp/codeagent-wrapper` path without checksum or signature verification.

**Risk:** A cache/TOCTOU attack or MITM with a populated `/tmp` directory allows attackers to replace the binary.

**Recommendation:**
- Pin a versioned URL instead of using "latest" tag
- Verify checksum or signature before install
- Write to a unique temp file with restrictive permissions

---

### 2. Command Injection in install.py
**Location:** `install.py:365-444`

**Issue:** Executes arbitrary commands from config using `shell=True` with unvalidated `env` and no sandbox.

**Risk:** Combined with `config.json` being user-editable, this creates a supply-chain attack vector.

**Recommendation:**
- Use `shell=False` with argv lists
- Whitelist allowed commands
- Validate `env` keys and values before execution

---

### 3. Path Traversal in _target_path
**Location:** `install.py:247-263`

**Issue:** Resolves paths but doesn't enforce they stay under `install_dir`.

**Risk:** A malicious config could overwrite arbitrary files outside the intended directory.

**Recommendation:**
- Reject targets that escape `install_dir` after `resolve()`
- Compare `Path.resolve()` parents
- Refuse symlinks that could bypass restrictions

---

## Performance & Resilience Issues

### 4. Unbounded Parallel Workers
**Locations:**
- `config.go:267-287`
- `executor.go:321-441`

**Issue:** Parallel execution defaults to unbounded workers when `CODEAGENT_MAX_PARALLEL_WORKERS` is unset.

**Risk:** A large task file can spawn many backend processes, saturating CPU and RAM.

**Recommendation:**
- Default to a sane cap (e.g., `runtime.NumCPU()`)
- Expose an explicit flag to raise the limit when needed

---

## Maintainability Issues

### 5. Verbose CLI Output Leaking Secrets
**Location:** `main.go:364-369`

**Issue:** The CLI prints the full backend command (including user prompt/task) to stderr before execution.

**Risk:** Secrets can leak and logs become bloated.

**Recommendation:**
- Add a `--quiet/--redact` mode (default-on for sensitive content)
- Strip or mask sensitive data from logged commands

---

### 6. Dead Code and Reduced Debuggability
**Locations:**
- `utils.go:264-274` (dead test/placeholder helpers)
- `main.go:139-160` (log files removed immediately after runs)

**Issue:** Unused helper functions and immediate log deletion reduce maintainability and debuggability.

**Recommendation:**
- Remove unused helpers
- Keep logs by default or gate deletion behind `--cleanup` flag

---

## Documentation Gaps

The following topics are not covered in the documentation:

1. **Unverified binary download** - No warning about the security implications
2. **Environment variable effects** - `CODEAGENT_SKIP_PERMISSIONS` and `CODEX_BYPASS_SANDBOX` effects undocumented
3. **Resource limits** - Default unbounded parallelism and `CODEAGENT_MAX_PARALLEL_WORKERS` not documented
4. **Log retention** - No guidance on log cleanup behavior

**Recommendation:** Add a "Security & Resource Limits" section to:
- `README.md`
- `docs/CODEAGENT-WRAPPER.md`

Cover: verification, path pinning, safe defaults, and log-retention/cleanup guidance.

---

## Priority Matrix

| Issue | Severity | Impact | Effort |
|-------|----------|--------|--------|
| Unverified Binary Download | Critical | High | Medium |
| Command Injection | Critical | High | Medium |
| Path Traversal | High | High | Low |
| Unbounded Parallel Workers | Medium | Medium | Low |
| Secret Leaking | Medium | Medium | Low |
| Dead Code | Low | Low | Low |

---

## Summary

The repository has solid architecture but needs hardening in three critical areas:
1. **Binary integrity verification** - Essential for supply-chain security
2. **Command execution safety** - Prevent injection attacks
3. **Path validation** - Prevent file overwrite attacks

The documentation should be updated to warn users about security implications and resource management.
