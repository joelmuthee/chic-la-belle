# Chic La Belle — build notes

Brand-new **women's outfits** catalog, Nairobi. Forked from **Ryker Luxury** (new-stock data model) per CATALOG-STANDARDS "Onboarding a new client" (NOT `new-client.sh`, which is the deprecated thrift/GitHub-Pages downgrade path). Built 2026-06-02.

## Live
- Public site: https://chic-la-belle.pages.dev  (CF Pages project `chic-la-belle`, prod branch `main`)
- Admin: https://chic-la-belle.pages.dev/admin  (password is in `admin.js`)
- Worker API: https://chiclabelle-api.stawisystems.workers.dev  (`chiclabelle-api`)
- CF account: Stawisystems (`58685495706b973821d77208248c66fc`)
- KV namespace: `chiclabelle-bags` = `444775e60c804ddd8ddff209d3f9f801` (binding `BAGS`)
- Worker secrets: `ADMIN_TOKEN` (fresh), `MASTER_TOKEN` (fleet kill-switch value, shared across all catalogs — see `worker/.master-token`)

## Client facts
- IG: @chic_labelle_254 — numeric user id **68768998504** (hardcoded in `admin.js` IG_USER_ID + worker `/api/ig-discover` fallback)
- WhatsApp: 254112199913 (settings.whatsappNumber + main.js fallback)
- Shop: Mithoo Biashara Center, 2nd floor, Shop S-05, Moi Avenue, Nairobi
- Sells: brand new women's outfits → **new-stock** data model (`stock:{size:qty}` + `sales:[]`)

## Re-verticalisation (Ryker menswear → women's apparel)
- Worker: `WOMENSWEAR_KEYWORDS` map, `CLB_CATEGORIES` set + `coerceCategory`, both AI prompts (vision + text), women's size parser (letter XS-3XL + UK numeric 4-22 + Free Size; dropped jeans-waist 28-44 and menswear UK shoes), default category `Dresses`. Removed the `deriveBrand` leading-handle strip (latent feed-API bug, CATALOG-STANDARDS §4).
- Admin: form category `<select>` (12 women's cats), `APPAREL_CATEGORIES` (IG-sync dropdown), `genDescription` catMap + ungendered/dash-free copy, size grids (UK 4-22 + women's UK shoe 3-9).
- Categories: Dresses, Tops, Two Piece Sets, Jumpsuits, Skirts, Trousers, Jeans, Shorts, Outerwear, Knitwear, Activewear, Accessories.

## Brand palette (from Logo.jpg)
Blush pink `#FFC8DD` · black `#0A0A0A` · Louboutin red `#AB262B`. Recoloured Ryker's chocolate/champagne `:root` vars → plum-black ink `#1E1014`, rose accent `--gold:#C2557A`, blush `--gold-light:#F0C4D6`, blush-cream bg `#FFF4F8`. Hero is a plum gradient with rose glow (no image file).

## Privacy / scrub (done)
- `/api/buyer` **neutralised** (acks without forwarding) so buyer PII never lands in Ryker's GHL until Chic La Belle has its own form. Admin still records sales to KV.
- Scrubbed all Ryker SITE/OG/GA(`G-WPHV51W4TM`)/GHL-external-tracking/IG-handle/address strings from index.html, main.js, worker share page. Removed the rykerluxury domain-migration redirect from index.html + admin.html.

## Seed
- 127 products seeded from the IG **feed API** `/api/v1/feed/user/68768998504/` (NOT the grid), top-150 cap, parsed captions client-side (Python mirror of worker heuristics), pushed via `/api/ig-sync` in chunks of 4 (oldest-first → newest-first catalog).
- Skipped: 1 sold (`\bsold(out)?\b`), 22 no-name/address-footer/info posts. **Key gotcha:** the shop's standard caption footer "Shop S-05" was faking size "S" on every item — fixed by stripping the address/contact footer before name+size parsing.

## Pending / follow-ups
- **GitHub repo:** pushed to https://github.com/joelmuthee/chic-la-belle (branch `main`). Auto-deploy via `.github/workflows/deploy.yml` will run on push but **fails until the reusable `CLOUDFLARE_API_TOKEN` repo secret is added** (Settings → Secrets and variables → Actions; account-scoped token, paste with NO trailing newline). Until then deploy manually: worker `cd worker && npx wrangler deploy`; site `CLOUDFLARE_ACCOUNT_ID=58685495706b973821d77208248c66fc npx wrangler pages deploy . --project-name=chic-la-belle --branch=main --commit-dirty=true`.
- ~~Custom domain~~ **DONE** — `chiclabelle.essenceautomations.com` live (added via CF Pages dashboard, auto-CNAME). All public URLs swapped from `chic-la-belle.pages.dev` (worker SITE, index OG/canonical, admin broadcast lookUrl). `API_BASE` stays on the worker domain. Item-share `/p/<id>` previews ride the worker domain (no zone bot-protection); homepage OG verified 200 to the FB crawler.
- **GHL buyer capture**: provision a Chic La Belle GHL form, then re-wire `/api/buyer` (formId/locationId/notes-field-key) per `workflows/new_client_onboarding.md`.
- **Billing**: add to clients-dashboard with `catalog_api_base = https://chiclabelle-api.stawisystems.workers.dev` for the kill-switch.
