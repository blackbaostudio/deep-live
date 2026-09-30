# CLAUDE.md — Email Signatures (BUSINESS-CRITICAL)

**STOP. Read before touching anything in this directory.**

These email signatures are live in clients' mail clients (Gmail/Outlook). Each
client pasted HTML that hardcodes **absolute URLs** pointing at this site:

```
https://deepestateagency.com/email-signature/<file>.html
https://deepestateagency.com/email-signature/assets/<file>
```

Those URLs are a **permanent public contract**. We do not control the pasted
copies — we can only keep the URLs they point at alive and correct. A rename,
delete, move, or extension change breaks every signature already in the wild,
silently, with no way to push a fix to clients. This already happened once
(commit `822d3c0` deleted the logos + photos + HTML → live 404s).

## Invariants — NEVER break these

1. **Never delete, rename, move, or change the extension** of any file under
   `assets/` or any `*.html` here. The filename *is* the contract.
2. To update an image, **replace the file in place** — same path, same name,
   same extension. Re-export to match the existing dimensions.
3. These files **must survive the build**. `bun run build` copies `public/` →
   `out/`, and `make deploy-live` pushes `out/` to gh-pages. If a file is gone
   from `public/`, it 404s on the next deploy. Removing a file here is removing
   it from production.
4. Hosting is **GitHub Pages** (`server: GitHub.com`, Fastly CDN). Not
   Cloudflare. SSL is GitHub-managed. Cache propagation after deploy is ~1 min.

## Current contract (exact filenames that must keep resolving)

HTML pages:
- `david-ceo.html`, `david-founder.html`
- `pedro-coordinacion.html`, `pedro-real-estate.html`

Assets (referenced by the HTML — photos are **PNG**, not JPG):
- `assets/logo-icon.png`
- `assets/logo-text.png`
- `assets/david-soriano.png`
- `assets/pedro-soriano.png`

`.bak/` holds the exact HTML delivered to clients — source of truth, do not
edit, restore from here if a working-tree copy is lost.

## Email-client compatibility (do not regress)

- Inline styles only. Table-based layout. **No external CSS, no `<script>`.**
- Keep it caniemail-safe (the signatures were authored against caniemail).
- Absolute `https://` URLs for every image — relative paths do not work in mail.

## After ANY change here — verify before considering it done

```bash
bun run build
# every referenced asset must exist in out/
for ref in $(grep -rhoE 'assets/[a-zA-Z0-9_-]+\.(png|jpg)' out/email-signature/*.html | sort -u); do
  [ -f "out/email-signature/$ref" ] && echo "OK  $ref" || echo "MISSING  $ref"
done
```

After deploy, confirm each page + image returns `200` live:

```bash
base=https://deepestateagency.com/email-signature
for p in david-ceo david-founder pedro-coordinacion pedro-real-estate; do
  curl -sS -o /tmp/s.html -w "%{http_code}  $p.html\n" "$base/$p.html"
  grep -ohE 'src="[^"]+\.(png|jpg)"' /tmp/s.html | sed -E 's/src="//;s/"$//' | sort -u |
  while read u; do case "$u" in http*) url="$u";; *) url="$base/$u";; esac
    curl -sS -o /dev/null -w "  %{http_code}  $u\n" "$url"; done
done
```
