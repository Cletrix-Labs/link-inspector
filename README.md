# Link Inspector

A zero-backend, single-page tool that gives you a plain-language safety report on any link before you click it. Paste a URL, run ten heuristic checks, get a verdict - all in your browser.

**Nothing is ever sent anywhere.** No server, no analytics, no tracking. The URL never leaves your device.

## Why

Phishing links rely on you not looking closely - a `paypa1.com`, a `bit.ly` redirect, an `@` symbol hiding the real destination. Link Inspector runs the same checks a careful person would do by eye, instantly, and explains each one in plain English.

## How it works

Paste a link and it's run through ten checks:

| Check | Flags |
|---|---|
| Protocol | Plain HTTP instead of HTTPS |
| Hidden destination | `@` tricks that disguise the real target |
| Raw IP address | Links pointing at an IP instead of a domain name |
| Punycode / IDN | Disguised international characters (`xn--`) |
| Brand lookalike | Domains within edit-distance 2 of commonly impersonated sites (Levenshtein) |
| Suspicious TLD | Endings like `.zip`, `.top`, `.click`, `.tk` that show up often in spam |
| URL shorteners | Links where the real destination is hidden until you click |
| Subdomain chain length | Long subdomain chains used to bury the real domain |
| Credential-bait wording | "login" / "verify" / "secure" combined with no HTTPS |
| Domain entropy | Unusual digit/hyphen ratio typical of generated phishing domains |

Each check returns ok / caution / high-risk, and the page stamps an overall verdict: **LOOKS OK**, **MINOR FLAGS**, **USE CAUTION**, or **HIGH RISK**.

This is heuristic, not authoritative. A clean report isn't a guarantee of safety, and a flagged link isn't always malicious - it's a second pair of eyes, not a verdict to blindly trust.

## Running it

It's one HTML file. No build step, no dependencies, no server.

```
python3 -m http.server 8080
# or just open index.html directly in a browser
```
