# CLAUDE.md

Guidance for Claude Code when working in this repo. Global rules already apply and are
NOT repeated here — see `~/.claude/CLAUDE.md` and `~/projetos/CLAUDE.md` (Doppler,
Hetzner/VPS rules, dev-port allocation, forbidden Syne font, PT-BR writing style).
None of the VPS/Docker/Doppler rules apply to this project — it deploys to Vercel.

## Hard rules (read first)

- **No build step, no framework, no dependencies — keep it that way.** Do not add a
  bundler, npm package.json, React/Vue/etc., or a CSS preprocessor unless explicitly
  asked. The whole point of this site is that it's one file you can open and edit.
- **`home.html` does not exist.** The page was renamed to `index.html` in commit
  `1fdfb9b` (2026-06-08). If you see `home.html` referenced anywhere (old docs, old
  chat context), it's stale — the real file is `index.html`.
- **Never add text without both `data-en` and `data-pt` attributes.** The bilingual
  toggle (`setLang()` in the inline `<script>`) overwrites `innerHTML` by reading
  `el.dataset[lang]`. Text with only one language attribute (or none) silently breaks
  or disappears when the visitor switches languages.
- **Never delete or move files in `public/`, `posters-tours/`, or `epk/files*`
  without asking first.** These are raw asset dumps and are untracked (not in git,
  see `.gitignore`/`.vercelignore`) — deleting them is unrecoverable, there is no
  backup copy in version control.
- **Never hand-edit `epk/gigs.json` or `epk/gigs/` as the source of truth.** They are
  generated: the Circuit Mapper admin (`~/projetos/cult-circuit`, tab "Shows MS")
  commits them into this repo via the GitHub API. Editing them here works until the
  next publish, which overwrites `gigs.json` wholesale. Change shows in the admin.
- **`index.html` and `epk/index.html` use two different, unrelated design systems.**
  `index.html`'s tokens are `--ink/--glow/--rust/...`; the EPK's are
  `--bg/--panel/--amber/--hot/...`. Don't assume a token from one exists in the other.

## Verification before destructive actions

Before deleting or overwriting any media file (`.mp4`, `.gif`, `.psd`, images):
1. `git status` — check if it's tracked.
2. `git log --follow -- <file>` — if there's no history, it's untracked and there is
   no way to recover it once removed.
Only `Mars_and_Venus_colliding.gif` (root) and the files under `epk/` are actually
versioned; everything in `public/`, `posters-tours/`, and the loose `*.mp4` files at
repo root are untracked working copies.

## Development

Open the file directly in a browser — no server needed for normal editing:
```
xdg-open /home/paulo/projetos/morningstorm/index.html
xdg-open /home/paulo/projetos/morningstorm/epk/index.html
```
For anything that needs relative paths / CORS to behave like production (e.g.
testing the EPK's video/image loading), serve the directory on the project's
reserved local port:
```
cd /home/paulo/projetos/morningstorm && python3 -m http.server 3314
# http://localhost:3314/       -> index.html
# http://localhost:3314/epk/   -> epk/index.html
```
<!-- CONFIRMAR COM PAULO: porta 3314 e o comando python3 -m http.server são inferidos
das permissões salvas em .claude/settings.local.json (curls para localhost:3314);
não há package.json nem script registrado que confirme esse é o método oficial. -->

## Deploy

Vercel project `morningstorm` (see `.vercel/project.json`), live at
`https://www.morningstorm.com.br` (root = crowdfunding page, `/epk/` = press kit,
`trailingSlash: true` in `vercel.json` makes `/epk/` resolve).
<!-- CONFIRMAR COM PAULO: deploy é automático via GitHub integration no push pra main,
ou manual via `vercel deploy --prod`? settings.local.json tem permissão pra
`vercel deploy *`, sugerindo uso manual, mas não há confirmação. -->

## Definition of done

There's no lint/build/test tooling (no package.json) — verification is manual:
- Open the page in a browser in both languages (default EN, click PT toggle) and
  confirm nothing renders empty or broken.
- Resize to mobile width and confirm the hamburger menu / drawer still works
  (this broke once before shipping, see Anti-patterns below).
- Any new/edited copy has matching `data-en` + `data-pt` pairs (`grep -c 'data-en='`
  vs `grep -c 'data-pt='` on the file should match).
- If you touched the fundraising numbers, all three displays are consistent: the
  `R$ X raised of R$ Y goal` text, the `N% funded` text, and the `.bar-fill` inline
  width/inset — they are not computed, they're three separate hand-edited values.

## Current state (as of last commit touching each file)

- `index.html` — last changed 2026-06-11 (`76ea5ac`). Crowdfunding landing page.
- `epk/index.html` — last changed 2026-06-18 (`d7d581d`), actively worked on more
  recently than the main page.
- Branch: `main`. Deployed at https://www.morningstorm.com.br
- **Everything on the crowdfunding page is placeholder/mock — nothing is wired to a
  real payment or donation flow yet:**
  - All tier "Choose" buttons, the "Join" email-capture button, streaming links
    (Spotify/Bandcamp/Apple Music/YouTube), and "Get in touch" are `href="#"`.
  - Lineup section has `[Name]` / `[ photo ]` placeholders for all 4 members and a
    bracketed placeholder blurb.
  - Budget list (`#pitch`) has `[N]` token placeholders for date/country counts.
  - Hero tag has a `[year]` placeholder ("est. `[year]`").
  - Route dates are `TBC` except the Bamberg anchor date (Thomann run, 15 Jan).
  - Fundraising bar shows R$31.200 of R$48.000 (65%, 214 backers, 38 days left).
<!-- CONFIRMAR COM PAULO: os números de arrecadação acima são dados reais de uma
campanha já rodando em outra lugar (que precisa ser sincronizado manualmente aqui),
ou são só valores de mockup para mostrar o layout? E: qual vai ser o destino real dos
links "Choose"/"Join" — plataforma de crowdfunding, Pix direto, outro? -->

## Stack conventions

- Single-file HTML: CSS in one `<style>` block, then markup, then one `<script>`
  block at the bottom (nav toggle + `setLang()`). No JS beyond that — no build,
  no framework, no external JS libraries.
- Bilingual pattern: every user-facing string carries `data-en="..."` and
  `data-pt="..."`; `setLang()` swaps `innerHTML` for all `[data-en]` elements and
  syncs the EN/PT toggle buttons + `<html lang>`. Default language is picked from
  `navigator.language` on load.
- Fonts: Google Fonts, loaded via `<link>` (Anton = display/headings, DM Sans = body,
  JetBrains Mono = labels/tags/mono UI bits) — not self-hosted.
- `index.html` design tokens (`:root`): `--ink/--ink-2/--concrete` (background
  layers), `--line` (borders), `--smoke` (muted text), `--bone` (primary text),
  `--glow/--glow-2` (orange accent/CTAs), `--rust` (secondary accent, fund bar
  gradient).
- `index.html` utility classes: `.mono`, `.display`, `.wrap` (max-width container),
  `.btn` / `.btn-ghost`, `.rv` + `.d1`–`.d4` (scroll-reveal animation delays).
- `index.html` sections in order: nav → hero (video/gif bg + fundraising bar) →
  `#pitch` → `#music` → `#tiers` → `#lineup` → `#route` → `#join` → footer.
- `epk/index.html` is a separate bilingual page (own tokens, see Hard rules) — the
  Electronic Press Kit, not part of the crowdfunding funnel.

## Anti-patterns (with evidence)

- **Don't assume mobile nav works without checking.** The hamburger menu + slide-down
  drawer were missing initially and had to be retrofitted (`3bed8ea`, "Fix mobile nav
  with hamburger menu and slide-down drawer") — always check the `<880px` breakpoint
  after nav changes.
- **Don't write "home.html" in commands or docs** — see Hard rules above; it was
  renamed in `1fdfb9b` and stale references cause "file not found" confusion.

## File map

- `index.html` — main crowdfunding page (HTML+CSS+JS, single file).
- `epk/index.html` — Electronic Press Kit, separate bilingual page, own design system.
- `epk/gigs.json` — show list rendered into the `#live` section (generated, see below).
- `epk/gigs/` — gig posters referenced by `gigs.json` (generated; git-tracked and deployed).
- `epk/hero.mp4`, `epk/van.png`, `epk/logo-transp-white.png`, `epk/covers/*.jpg` —
  media referenced by the EPK page (all git-tracked).
- `Mars_and_Venus_colliding.gif` — hero background image for `index.html` (the only
  root-level media file that's actually versioned).
- `vercel.json` — `{"trailingSlash": true}`, required for `/epk/` to resolve.
- `.vercel/project.json` — Vercel project link (`morningstorm`).
- `.vercelignore` — excludes `public/`, `posters-tours/`, `epk/files*`, `*.psd` from
  deploy.
- `public/`, `posters-tours/` — untracked raw asset dumps (~52 MB / ~188 KB), not
  deployed, not backed up in git — see Hard rules.

## Boundaries

- This project has no VPS/Docker footprint — don't apply the Hetzner/Doppler rules
  from the global CLAUDE.md files here, they're for other projects on that host.
- Don't touch Vercel domain/DNS settings or the `morningstorm.com.br` domain config
  without explicit instruction — this is a live public site.
- Don't rename/move `index.html` or `epk/index.html` — both are live, deployed URLs.


## Classificação (vitrine)

categoria: cliente


## Shows section (`#live`) — how it is updated

The `#live` section of `epk/index.html` is **not hand-edited**. It renders from
`epk/gigs.json` at page load (`loadGigs()` in the inline script).

**Flow:** Paulo drops a gig poster into the Circuit Mapper admin
(https://circuit.morningstorm.com.br, tab "Shows MS") -> Claude vision reads date,
city, venue and line-up -> Paulo reviews and corrects the form -> "Publicar no site"
commits `epk/gigs.json` + `epk/gigs/*` into this repo via the GitHub API -> Vercel
auto-deploys. See `~/projetos/cult-circuit/CLAUDE.md` for the admin side.

Layout rules encoded in the renderer:
- Exactly one gig may be `pinned` — it renders as the wide `.gig-pin` destaque card
  at the top, bordered in `--hot` (same orange as the hero `.stamp`; do not
  introduce a new accent colour for this section).
- The destaque is a manual choice, **not** derived from the date: RODOX stays
  featured after 29 Oct 2026 as a credential. Only the admin's pin flag moves it.
- Remaining gigs split into "Upcoming" (date >= today, soonest first) and
  "Past events" (date < today or no date, most recent first).
- Past posters are capped at 210px (`.gigs` track max) so they stay visually
  subordinate to the 290px destaque poster. Removing that cap inverts the
  hierarchy — it already happened once during the build.

**Static fallback.** The markup inside `#gigs-root` in `epk/index.html` is a
hand-written copy of the current shows. It is what a visitor sees if `gigs.json`
fails to load; `loadGigs()` replaces `#gigs-root` wholesale on success. It is a
safety net, not the source of truth — refresh it by hand only if the featured show
changes and you want the fallback to match.

**Every generated text node carries both `data-en` and `data-pt`** (see `gigEl()`),
because `setLang()` overwrites `innerHTML` of everything matching `[data-en]`. A
node with only one of the two disappears when the visitor switches language.

**Known limitation:** publishing never deletes posters from `epk/gigs/`. A gig removed
in the admin drops out of `gigs.json` (so it leaves the page), but its image file
stays in the repo. Harmless; prune by hand if it ever matters.
