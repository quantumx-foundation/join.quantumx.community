# Join QuantumX Community

The landing page behind **join.quantumx.community**. It shows the QuantumX
Discord card, then sends visitors to the Discord invite.

Everything in `public/` is served as-is — there is no build step and no
dependencies.

## Why this isn't a plain 301

The obvious way to do this is a server redirect, but that breaks the link
preview. Discord, X, WhatsApp, LinkedIn and Slack crawlers follow 3xx responses,
so they would land on `discord.gg` and show *Discord's* card instead of ours.

So the page returns real HTML with the Open Graph tags, and only browsers get
moved along:

- **Crawlers** read the meta tags and `og.png`, and never run the script.
- **Browsers** see the card for 2.2s, then `location.replace()` to the invite.
- **No JavaScript** falls back to a `<meta http-equiv="refresh">` that is kept
  inside `<noscript>`, so crawlers ignore it.

`location.replace` (rather than `href`) keeps the back button working: it
returns people to wherever they came from instead of bouncing them into Discord
again.

## Editing

| What | Where |
| --- | --- |
| Discord invite | `INVITE` in the inline script **and** the `href` on `#invite` in `public/index.html` |
| Headline / copy | `<main>` in `public/index.html` |
| Social preview | `public/og.png` (1200×630, generated from the site's Discord card) |
| Favicons | `public/favicon.ico`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` |

The invite is written in two places on purpose: the `href` is what crawlers and
no-JS visitors see, and the script rewrites it with UTM tags at runtime. Keep
them in sync.

Arrivals are tagged `utm_source=join.quantumx.community&utm_medium=join`, matching
the convention in `quantumx.community/src/lib/site.ts`. Discord reports usage per
invite code rather than per query string, so to measure this domain separately
you need a dedicated invite from Server settings → Invites.

## Local preview

```sh
python3 -m http.server 4173 --directory public
```

Then open http://localhost:4173 — add `?stay=1` to stop the auto-redirect while
you work on the page.

## Deploy

Netlify, publishing `public/` (see `netlify.toml`). Point the
`join.quantumx.community` DNS record at the site, then re-scrape the preview at
<https://www.opengraph.xyz> or Discord's own embed cache to confirm the card.
