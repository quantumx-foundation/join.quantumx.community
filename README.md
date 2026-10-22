# Join QuantumX Community

The landing page behind **join.quantumx.community**. It shows the QuantumX
Discord card, collects an email address, and then unlocks the Discord invite.

Everything in `public/` is served as-is — there is no build step and no
dependencies.

## The flow

1. The page renders with the **Join Discord Server** button visibly locked.
   JavaScript removes the button's `href` on load, so it is neither clickable
   nor focusable.
2. The visitor enters an email and submits. The form is posted to Netlify
   Forms with `fetch()`, so the page never navigates away.
3. On success the button unlocks and a confirmation replaces the input.

If the save fails, the page **fails open**: it shows an error but unlocks the
button anyway. A storage hiccup should not stand between somebody and the
community — we lose the address, they still get the invite.

> **This is a soft gate, not access control.** The invite URL ships in the HTML
> so that crawlers and no-JS visitors can see a real link, which means anyone
> reading the page source can skip the form. Making it enforceable would mean
> holding the invite in a Netlify Function and handing it out only after the
> address is stored.

## Why this isn't a plain 301

An earlier version redirected to Discord automatically. Even then a server
redirect was the wrong tool, and the reasoning still governs the meta tags:
Discord, X, WhatsApp, LinkedIn and Slack crawlers follow 3xx responses, so they
would land on `discord.gg` and show *Discord's* card instead of ours.

The page therefore serves real HTML with the Open Graph tags, and crawlers read
the meta tags and `og.png` without ever running the script. Now that nothing
auto-redirects, this is simply the normal path for everybody.

## Netlify Forms

Submissions land in **Site configuration → Forms** in the Netlify dashboard,
under the form name `join`.

Two things are easy to miss:

- **Form detection must be enabled** on the site (Site configuration → Forms).
  On newer Netlify sites it is off by default, and until it is on, submissions
  return 404. Enabling it requires a redeploy before the form is registered.
- Netlify discovers forms by **parsing the deployed HTML at build time**, which
  is why the `<form>` is real markup with `data-netlify="true"` rather than
  something JavaScript builds.

Spam is filtered by a honeypot field (`bot-field`), hidden off-screen. The free
tier includes 100 submissions per month.

Visitors without JavaScript post the form natively and land on
`public/thanks.html`, which carries the same invite. That page is `noindex` and
deliberately absent from `sitemap.xml`.

## Editing

| What | Where |
| --- | --- |
| Discord invite | Three spots in `public/index.html` (the `#invite` `href`, the `INVITE` constant, the JSON-LD `sameAs`) **and** `public/thanks.html` |
| Headline / copy | `<main>` in `public/index.html` |
| Consent line | `.note` in `public/index.html` |
| Social preview | `public/og.png` (1200×630, generated from the site's Discord card) |
| Favicons | `public/favicon.ico`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` |

The invite is written in several places on purpose: the `href` is what crawlers
and no-JS visitors see, and the script rewrites it with UTM tags at runtime.
Keep them in sync — `grep -rn "discord.gg/" public/` lists them all.

**Invites expire.** The code `2w3pgqTQY` silently expired once already and sent
everyone to a dead page, and `gFZvXRsxr` was set to expire too. The current
code (`3f7BnEe4DF`) is non-expiring — if you ever replace it, keep it that way
(Server settings → Invites → *Expire after: Never*, *Max uses: No limit*) and
check it with:

```sh
curl -s https://discord.com/api/v10/invites/<code> | python3 -m json.tool
```

Arrivals are tagged `utm_source=join.quantumx.community&utm_medium=join`, matching
the convention in `quantumx.community/src/lib/site.ts`. Discord reports usage per
invite code rather than per query string, so to measure this domain separately
you need a dedicated invite from Server settings → Invites.

## Local preview

```sh
python3 -m http.server 4173 --directory public
```

Then open http://localhost:4173. The layout and the lock/unlock behaviour work,
but **form submission does not** — the POST is handled by Netlify, not by this
server, so it will fail and the page will fall through to its error path. To
exercise the real form, use `netlify dev` or a deploy preview.

## Deploy

Netlify, publishing `public/` (see `netlify.toml`). Point the
`join.quantumx.community` DNS record at the site, then re-scrape the preview at
<https://www.opengraph.xyz> or Discord's own embed cache to confirm the card.
