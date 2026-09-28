# Security Policy

## Supported Versions

| Version | Supported |
| ------- | --------- |
| 2.0.x   | ✅ |
| < 2.0   | ❌ |

## Reporting a Vulnerability

If you discover a security issue (e.g., XSS via `policyUrl`, cookie injection, CSP bypass):

1. **Do not** open a public issue.
2. Email the maintainer via GitHub Security Advisories or open a draft security advisory at `https://github.com/Ramnik01/GDPR-Script/security/advisories/new`.
3. Include steps to reproduce and impact.

We aim to acknowledge within 48 hours and release a fix within 7 days for critical issues.

## Security Fixes in 2.0

- `policyUrl` is allow-listed to `http:`/`https:`/`mailto:` only; other schemes fall back to `#`.
- No `innerHTML` with user-controlled strings; all text uses `textContent`.
- No `eval`/`Function`; only `addEventListener`.
- Consent cookie set with `SameSite=Lax` and `Secure` on HTTPS.
- Category IDs sanitized to `[a-z0-9_-]`.
