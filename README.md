# Vernon Ouyang — Personal Site

Source for [vernonouyang.com](https://vernonouyang.com) (coming soon).

## Stack
- Pure static HTML + CSS + JS (no build step)
- Typography: JetBrains Mono, Instrument Serif, Inter
- EN / 中文 i18n toggle, persists in `localStorage`
- Typing animation on hero
- Hosted on **Cloudflare Pages** (primary, global)
- Mirrored through **Tencent Cloud CDN** (China)

## Deploy
Any push to `main` auto-deploys via Cloudflare Pages.

```bash
# local preview
python3 -m http.server 8000
# then open http://localhost:8000
```

## Structure
```
index.html              # main page
papers/                 # watermarked public papers (CC BY-NC-ND 4.0)
resume/                 # EN + CN resume PDFs
_headers                # Cloudflare Pages cache + security headers
_redirects              # legacy URL redirects
robots.txt              # crawler policy
```

## License
Code: MIT.
Written content and PDFs under `papers/`: CC BY-NC-ND 4.0 — non-commercial, no derivatives, attribution required.
