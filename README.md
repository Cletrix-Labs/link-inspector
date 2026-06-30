# Link Inspector

A single-page tool that gives you a real safety check on a link before you click it - not just pattern-matching, an actual lookup against Google's live threat database, plus a heuristic checklist that explains *why* something looks off.

**Nothing is sent anywhere except Google's Safe Browsing lookup endpoint**, and only when you supply your own API key. No tracking, no analytics, no third-party server in between.

## Why this version is different

The first version was heuristics-only, which means it could give a clean "LOOKS OK" verdict to a link that's actively malicious - modern phishing sites use valid HTTPS and clean-looking domains, so string pattern-matching alone misses most real attacks.

This version adds a real signal on top: a live lookup against **Google Safe Browsing**, the same database Chrome itself checks before warning you off a site. That's the part that actually catches currently-active phishing and malware. The heuristic checklist stays as a secondary layer that explains specific red flags, but it is no longer the headline.

## Setup

The Safe Browsing check needs a free API key (you bring your own - nothing is bundled or shared):

1. [Get a free Safe Browsing API key](https://developers.google.com/safe-browsing/v4/get-started) (Google Cloud Console → enable "Safe Browsing API" → create credentials).
2. In the app, open **"Real-time threat database"** and paste the key in.
3. The key lives only in that browser tab's memory - it resets on refresh and is never written to disk, localStorage, or any server you don't control.
4. **Recommended:** restrict the key by HTTP referrer in Google Cloud Console to whatever domain you deploy this to, so it can't be reused elsewhere if it leaks.

Without a key, the app still runs - it just skips straight to the heuristic checklist and says so plainly.

## How it works

**Primary check — Safe Browsing lookup**
Sends the URL to `safebrowsing.googleapis.com/v4/threatMatches:find` and checks it against Google's live list of malware, social-engineering (phishing), unwanted-software, and harmful-application sites. A match here overrides everything else and stamps **CONFIRMED THREAT**.

**Secondary checks - local heuristics (10 checks, no network)**

| Check | Flags |
|---|---|
| Protocol | Plain HTTP instead of HTTPS |
| Hidden destination | `@` tricks that disguise the real target |
| Raw IP address | Links pointing at an IP instead of a domain name |
| Punycode / IDN | Disguised international characters (`xn--`) |
| Brand lookalike | Domains within edit-distance 2 of ~70 commonly impersonated sites (Levenshtein) |
| Suspicious TLD | Endings like `.zip`, `.top`, `.click`, `.tk` that show up often in spam |
| URL shorteners | Links where the real destination is hidden until you click |
| Subdomain chain length | Long subdomain chains used to bury the real domain |
| Credential-bait wording | "login" / "verify" / "secure" combined with no HTTPS |
| Domain entropy | Unusual digit/hyphen ratio typical of generated phishing domains |

A **Copy report** button lets you paste the full result (verdict + every finding) somewhere else - useful for reporting a suspicious link to IT/security or a friend.

## Running it

Still one HTML file. No build step, no backend.

```
python3 -m http.server 8080
# or just open index.html directly in a browser
```

## Deploying

Works on any static host:

```
git init
git add .
git commit -m "Link Inspector"
gh repo create link-inspector --public --source=. --push
```

Then enable GitHub Pages on `main` in Settings → Pages, or drag the folder onto [Netlify Drop](https://app.netlify.com/drop).

## Honest limitations

- **Safe Browsing only knows about *known* threats.** A "not on Google's threat list" result means it isn't in their database yet - not that it's verified safe. Brand-new phishing domains can slip through for hours before they're indexed.
- **No domain-age / WHOIS check.** That data isn't available client-side without a paid API or a backend, so freshly-registered scam domains won't be flagged by that signal here.
- **Doesn't unwrap shortened links.** Following a redirect from the browser would need a server-side proxy, which breaks the "nothing leaves your device by default" promise - flagged as a shortener instead, with a note that the destination can't be verified.
- **Brand lookalike list (~70 domains) is still a sample, not exhaustive.** Easy to extend in `KNOWN_DOMAINS`.
- A clean report is a *better* signal now than before, but still not a guarantee. When genuinely unsure, type the address in yourself instead of clicking the link.

## Extending it

All logic lives in `index.html`. Heuristic checks live in `inspect()` and push findings:

```js
findings.push({
  level: 'warn',           // 'ok' | 'warn' | 'bad'
  title: 'Short title',
  detail: 'One sentence explaining what was found and why it matters.'
});
```

The Safe Browsing call lives in `checkSafeBrowsing()`. To extend the brand list, edit `KNOWN_DOMAINS`; for suspicious endings, edit `SUSPICIOUS_TLDS`.

## License

MIT.
