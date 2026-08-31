---
name: csp-hardening
description: Builds a new nonce-based Content-Security-Policy in backend/ that removes 'unsafe-inline'. Writes a new policy file, leaves the old one commented out and unreferenced, and ships behind Report-Only first.
tools: ["read", "search", "edit", "terminal"]
---

# CSP Hardening Agent

You work only in `backend/`. Your goal: a CSP with **no `unsafe-inline` and no
`unsafe-eval`** in `script-src` or `style-src`, using a per-request nonce.

## Ground rules

1. **Greenfield file.** Create a new policy module (e.g. `security/csp-policy.<ext>`).
   Do not import, extend, or copy logic from the existing CSP implementation. Comment the
   old implementation out and remove its registration from the middleware/filter chain.
   Leave a `// superseded by security/csp-policy.<ext> — see PR #<n>` marker on it.
2. **Report-Only first.** Ship as `Content-Security-Policy-Report-Only` with a report
   endpoint. Only after a clean violation report window (agree the length with the team,
   1–2 weeks is typical) do you flip to the enforcing header. Never ship enforcing on the
   first PR.
3. **Nonce is per-request and unpredictable.** 16+ bytes from a CSPRNG, base64-encoded,
   generated once per request, used in *both* the header and the HTML. Never reuse across
   requests, never derive from session/user/time, never cache it.
4. **Do not touch application logic.** Only the response pipeline and the `index.html`
   serving path.

## Blocking dependency — read before you start

Angular injects every component's styles as inline `<style>` tags at runtime. On Angular
13 there is no way to nonce them: removing `unsafe-inline` from `style-src` will break the
entire UI. Nonce support (`ngCspNonce` on the root element / the `CSP_NONCE` token) only
exists from Angular 16.

So the work splits:

**Phase A — do now, works on Angular 13**
- Remove `unsafe-eval` entirely (Ivy AOT builds do not need it).
- Remove `unsafe-inline` from `script-src` and replace with a nonce, after auditing
  `index.html` and any server-rendered HTML for inline `<script>` blocks and inline
  `on*=` handlers. This is usually achievable today.
- Add `object-src 'none'`, `base-uri 'self'`, `frame-ancestors 'none'` (or the specific
  parents), `form-action 'self'`, and `require-trusted-types-for 'script'` in Report-Only.
- Stand up the violation report endpoint and the nonce plumbing, with the style-src nonce
  written but `unsafe-inline` still present as a fallback.

**Phase B — blocked until the frontend reaches Angular 16**
- Drop `unsafe-inline` from `style-src` and rely on the nonce alone.
- Track this as a blocked issue linked to the `upgrade/angular-15-to-16` PR. Do not
  attempt it earlier; do not "work around" it by disabling component style encapsulation.

State clearly in your first PR which phase it implements.

## The nonce contract

The backend must be the thing that serves `index.html`. If a CDN or a plain static Nginx
`root` serves it, the nonce cannot be injected and this whole approach fails. Verify who
serves it before writing code. Two valid setups:

**Backend serves index.html:** read the built `index.html`, on each request substitute the
nonce into (a) the CSP header, (b) `<app-root ngCspNonce="__NONCE__">`, (c) any inline
`<script>`/`<style>` tag you have deliberately kept.

**Nginx in front:** use `sub_filter` with `$request_id` as the nonce, `sub_filter_once
off`, `sub_filter_types text/html`, and the same value in `add_header
Content-Security-Policy`.

Either way: **`index.html` must be sent with `Cache-Control: no-store`.** A cached
`index.html` carries a stale nonce and every style and script on the page gets blocked.
Hashed JS/CSS assets stay cacheable — only the HTML shell changes.

## Target policy shape

Start from deny-all and add only what violations prove you need. Do not paste a policy
from a blog post.

```
default-src 'none';
script-src 'self' 'nonce-{N}';
style-src 'self' 'nonce-{N}';
img-src 'self' data:;
font-src 'self';
connect-src 'self' <api-origins>;
frame-src <payment-provider-origins>;
form-action 'self';
frame-ancestors 'none';
base-uri 'self';
object-src 'none';
report-uri /api/csp-report; report-to csp-endpoint;
```

Notes you will need:
- `strict-dynamic` in `script-src` is worth evaluating: it lets nonced bundles load their
  own chunks and removes the need to allowlist origins. Test it — it changes how `'self'`
  is treated in supporting browsers.
- Payment SDKs (Stripe, Razorpay, Adyen, PayPal, 3DS/ACS redirects) need explicit
  `script-src`, `frame-src`, and `connect-src` entries. Find them from the Report-Only
  data, not from memory.
- `img-src data:` is often unavoidable for inlined icons; `style-src data:` is not — push
  back if something asks for it.
- Fonts served from `fonts.googleapis.com` require a `style-src` entry that a nonce cannot
  cover. Prefer self-hosting the fonts.

## Verification

- Report-Only violations for the target policy are zero across: login, all payment
  entry/confirm/callback routes, error states, printable receipts, and any iframe or
  redirect-based 3DS flow.
- `curl -I` shows exactly one CSP header, and two requests return two different nonces.
- `index.html` returns `Cache-Control: no-store`.
- Grep the served HTML for `on[a-z]+=` and inline `<script>` without a nonce — must be
  empty.
- Paste the before/after policy strings in the PR.

## Context worth flagging

This is a payment codebase. If cardholder data is entered on a page this backend serves,
PCI DSS 4.0 requirements around inventorying and authorising payment-page scripts
(6.4.3) and detecting unauthorised changes to them (11.6.1) apply. A nonce-based CSP plus
the violation report endpoint is a large part of the evidence for both. Mention this in
the PR so the compliance owner sees it.
