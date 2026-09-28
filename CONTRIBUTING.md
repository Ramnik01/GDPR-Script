# Contributing

Thanks for helping make GDPR-Script more compliant and accessible!

## Principles

- No dependencies, CSP-safe, XSS-safe.
- Accessibility first (keyboard, screen reader, reduced-motion).
- No dark patterns — equal choice by default.

## Dev

```bash
# no build step — just open the demo
open index.html
# or
npx serve .

# run tests
npm test  # or: node --test tests/*.test.js
```

## Pull Requests

- Keep changes small and explain the GDPR/ePrivacy rationale.
- Update `policyVersion` handling if consent shape changes.
- Add a test for new logic in `tests/`.
- Ensure `gdpr.js` passes `node --check gdpr.js`.

## Code style

- 2 spaces, semicolons off? This repo uses 2 spaces, single quotes in JS.
- Run `npx prettier --write gdpr.js gdpr.css` before committing if you like.
