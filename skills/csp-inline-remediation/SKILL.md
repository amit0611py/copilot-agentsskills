---
name: 'csp-inline-remediation'
description: 'Skill for eliminating unsafe-inline from Content Security Policy (CSP) across Angular and Spring Boot, quarantining legacy hacks, and establishing a nonce-based architecture without UI breakage.'
---

# CSP Inline Remediation Skill

Use this skill when asked to:
- *"Remove unsafe-inline from CSP without breaking Angular UI"*
- *"Fix Content Security Policy in Spring Boot and Angular"*
- *"Configure CSP nonce in Spring Boot filter and Angular CSP_NONCE token"*
- *"Clean up previous messy CSP attempts and install strict security headers"*

---

## 🛠️ Step-by-Step Skill Workflow

### Step 1: Quarantine Legacy CSP Hacks
Search and cleanly comment out previous trial-and-error CSP filters in `SecurityConfig.java` and `index.html` with `// [LEGACY_CSP_CLEANUP]: ...`.

### Step 2: Implement Spring Boot Nonce Pipeline
1. Add `CspNonceFilter` to generate a 128-bit cryptographically secure nonce per request and attach strict CSP headers (`script-src 'self' 'nonce-...' 'strict-dynamic'; style-src 'self' 'nonce-...'`).
2. Add `IndexHtmlNonceInjectorFilter` to replace `__CSP_NONCE__` inside `src/main/resources/static/index.html` dynamically at runtime.

### Step 3: Configure Angular Frontend
1. Set `"inlineCritical": false` under `"optimization.styles"` in `angular.json`.
2. Add `<meta property="csp-nonce" content="__CSP_NONCE__">` and `<app-root ngCspNonce="__CSP_NONCE__">` in `index.html`.
3. Provide `{ provide: CSP_NONCE, useFactory: getCspNonce }` in `app.config.ts` (or `app.module.ts`).

### Step 4: Refactor Dynamic Inline Styles
Scan components and convert `[ngStyle]` or `style="..."` attributes into clean CSS classes or CSS variables (`[style.--custom-var]`).
