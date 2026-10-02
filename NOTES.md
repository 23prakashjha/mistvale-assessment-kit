# NOTES — Mistvale Tea Co.

## 1. What I changed

**Bugs fixed (each was verified against the rules, not just "looked wrong")**

- `cart` started as `null`, so `cart.find(...)` threw on the first add. Now `[]`, and
  `loadCart()` scrubs anything unusable out of storage on the way in.
- The cart badge counted *lines* instead of *quantity* (`cart.length`).
- Prices were scraped out of the DOM with `innerText.replace('₹','')`, so a cart could
  never be trusted and `parseInt("1,299")` silently returned `1`. Every amount now comes
  from `PRODUCTS`.
- Clicking "All" emptied the grid: the filter compared `category === 'all'`. Chips also
  compared lowercase `'green'` against the real category `"Green"`, so Green teas could
  never appear at all.
- Both price sorts were inverted, and `sort()` was called on a reference to `PRODUCTS`,
  so sorting **mutated the source catalogue** and the order never recovered.
- `ids.indexOf(p.id)` returns `0` for the first hit, and `if (0)` is falsy — the first
  search result was always dropped.
- Search reset the category and sort (breaks R7) and had no debounce, so fast typing
  rendered stale results out of order. Now debounced, with a request token so a slow
  early response can never overwrite a newer one.
- `qty = input.value + 1` was string concatenation: `"1" + 1` = `11`.
- `cart.splice(i)` with one argument deleted every line from `i` to the end.
- A delegated listener was re-registered on every `renderCart()`, so handlers stacked up
  and one click added several items.
- `WELCOME10` was case-sensitive, had no ₹399 minimum, no ₹150 cap, no Gifts exclusion,
  and re-compounded on every click. It is now case- and whitespace-insensitive,
  idempotent, and idempotently capped.
- Free delivery tested the raw subtotal instead of the post-discount amount, and the
  "add ₹X more" bar never flipped to a "you have free shipping" state. Both fixed, and
  the bar is clamped to 100%.
- `checkPincode` had no `.catch()`, so an invalid pincode left the button stuck on
  "Checking…" forever. Every exit path now clears the pending state.
- The pincode was interpolated into `innerHTML` unescaped.
- Sold-out teas still had a working Add to cart button, and there was no stock or
  5-per-order cap.

**Design, UX and accessibility.** Rebuilt the page around the brand: Fraunces +
Manrope only, brand palette only, one primary button on each screen. The header originally
forced Shop / Delivery / Reviews / FAQ / Contact onto a full-width row of their own
underneath the logo, so the links sat on a *different line* from the logo, search and cart
and the sticky header was 155px tall — which also meant anchor targets landed *behind* it,
despite `scroll-margin-top` already assuming a 96px bar. The bar is now a symmetric
`1fr auto 1fr` grid: logo hard left, search and cart hard right, and the links centred
between them at exactly the middle of the bar. Centring them with `margin-inline:auto`
instead would have been wrong — the spare width gets shared between the auto margins, so
the links drift by half the difference between the logo and the search box. Below 960px
it stacks as logo + cart, then search, then links, and stops being sticky, because a
216px pinned header would eat a quarter of a phone screen.

Removed a `<marquee>`, a blinking `SALE!!!`, a self-opening newsletter pop-up, and
`* { transition: all .4s !important }` paired with `*:focus { outline: none !important }`.
Replaced the four `h1`s with one, and added a real `<label>` for the search box instead of
a placeholder. Re-rendering a grid destroys the element you were standing on, so repaints
now save and restore focus by a stable key — Tab no longer falls to `<body>` after adding
to the cart.

**Images.** Nine product images (WebP + AVIF, 800×800, ~5–12 KB each), two hero banners,
an SVG logo mark, favicons and a social card.

## 2. What the AI got wrong

The honest answer is that most of what I had to correct was **mine**, from earlier in the
same session:

- I wrote a `canTakeFocus` helper twice while fixing a focus bug and left a duplicated
  "Cart drawer" banner in the file. Harmless, but it shipped into a test run before I
  grepped for the declaration and found it.
- I "fixed" the skip link by making it `position: fixed` — correct — but only after
  measuring it and finding `top: -2210px` while the page was scrolled. I had assumed the
  CSS was fine because it *looked* fine in the source.
- I nearly reported a false pass: my first transition audit flagged every element with no
  transition at all, because `all 0s` is the computed *default*, not a bug.
- On the header I asserted for a long time that the nav links were "bottom aligned" as a
  styling detail, when the actual cause was `width:100%` forcing a flex wrap onto a second
  row. Measuring the boxes (`cy:160` for the links vs `cy:86` for everything else) settled
  it in one run; reasoning about it in the abstract had not.
- Centring those links took three attempts and I got two of them wrong. First I closed the
  new `.header-actions` wrapper *after* the nav instead of before it, which quietly put the
  nav inside it. Then I set only `grid-column` and not `grid-row`, and the auto-placer
  dropped the logo and the links onto a second row while the search stayed on the first.
  Both looked plausible in the source and were only visible by printing the computed
  `gridTemplateColumns` and each child's top edge.
- The single clearest example of the model being wrong: I twice wrote in my own summary
  that hover scale/rotation had been removed, while `transform: scale(1.04)` was still in
  the stylesheet. I only caught it by grepping the CSS against my own claim.

The fictional content in the original file was the other half: a Diwali sale, a
countdown targeting a date in the past, "Rated 4.9/5 by 10,000+ happy customers",
"AS SEEN ON SHARK TANK INDIA", "BEST TEA IN THE WORLD", and invented opening hours. All
removed, because a tea shop cannot claim what is not true.

## 3. Images

**Gap, stated plainly:** the images were generated earlier in this engagement and I
cannot reproduce the exact prompt strings — see Prompt 5 in `PROMPTS.md`. I have left that
entry open rather than filling it with something plausible. No image tool is exposed to me
in this session, so I could not regenerate them to recover the originals. This is the one
part of the brief I did not fully deliver.

What I can attest to, because I measured it: one art direction across all nine, square 1:1
framing, brand palette only, `p101`–`p108` all 800×800 with eight distinct SHA-256 hashes
(no duplicates), hero 1600×800 desktop and 800×1100 mobile, `og.jpg` at 1200×630. Every
asset is served as AVIF with a WebP fallback. After generation: resized to the boxes
above, compressed (largest product image is 12.4 KB), converted to AVIF and WebP, and an
unused `og.webp` was deleted.

**Limitation:** this model has no image input, so I could not visually review the
generated art. Everything above is verified by dimensions, formats, hashes, file size and
the browser successfully decoding all nine at their natural size. A human should eyeball
them before this ships.

## 4. How I tested it

Two throwaway harnesses, both re-run to green at the end.

- **Static audit** — `0 failures, 9 warnings`. The warnings are all known false
  positives: four decorative empty-`alt` thumbnails, `&#8377;` misparsed as the hex
  colour `#8377`, three primary buttons (one per view, by design), and image files the
  scanner thinks are unreferenced because they are referenced from JS templates and meta
  tags rather than literal HTML.
- **Headless Chrome over the DevTools protocol — 185 checks, 0 failures.** All eight
  business rules asserted against the live page: coupon arithmetic including the
  ₹150 cap and the Gifts exclusion, free delivery decided *after* discount (₹499 − ₹50
  = ₹449 → ₹49 delivery → ₹498 total), Indian digit grouping, the 5-per-order cap,
  sold-out sorted last in every one of the five sort orders, search/filter/sort
  intersecting, race conditions, `file://` storage, the checkout form contract, and the
  pincode pending/success/failure paths.

Also verified: 320 / 360 / 390 / 768 / 959 / 960 / 1024 / 1280 / 1440 px with no sideways
scrolling and equal-height cards at every width; all text ≥13px; all interactive targets
≥44px; 21 contrast pairs against WCAG AA; real Tab-key traversal with a ≥2px ring on all
ten stops; reduced-motion honoured; and a clean console. The header gets four dedicated
checks at each width above 960px — links on the logo's row, centred to within 1px of the
bar's centre, not colliding with the search, and the bar no more than 100px tall — because
I broke that layout twice while trying to improve it.

**The most useful habit:** the last seven failures were all my *test* being wrong, not the
page. Double-encoded JSON in a fixture, an off-by-one in a sold-out-last assertion,
forgetting a filter was still applied, asserting against a programmatic `.focus()` that
correctly does not trigger `:focus-visible`, and — twice — leaving a modal `<dialog>`
open, which makes the whole page inert so every later `focus()` silently does nothing.
Each was settled by dumping the real state, not by reasoning about it.

## 5. Questions for the team

1. **Share links to AI chats: there are none to give.** All of this work was done in a
   terminal-based coding agent (the opencode CLI), not a chat web UI, so there is no
   "Share" button and no public URL behind the conversation — the field should be left
   empty. I did not write plausible-looking `chatgpt.com/share/…` style links to fill it,
   because a dead link in a submitted assignment is a much bigger problem than an admitted
   gap. `PROMPTS.md` is the record of what was actually asked.
2. **Free-shipping threshold — I need a ruling.** `README.md` R4 says shipping is "free
   when the amount after the discount reaches *the free-shipping threshold*" and never
   gives the number. The old code tested against **₹599**, but the old announcement strip
   told customers **₹499**. Those disagree. I used **₹499**, because that is the figure
   that was published to customers and honouring it cannot break a promise already made.
   If the threshold is really ₹599, it is a one-line change to
   `FREE_SHIPPING_THRESHOLD`.
3. **The footer wordmark reads "MistVale"**, which contradicts the BRAND.md spelling
   rule, but it sits inside the do-not-edit legal block. I left it exactly as instructed
   rather than silently changing it. Should the legal block be amended by you?
4. **The exact image prompts** — see section 3. If you need them for the form, they have
   to come from the earlier session.
5. **Was `index.html` meant to ship with the legal line `© 2020 MistVale`?** It reads as
   stale, but I treated it as protected.

## 6. Time spent

About 5 hours. Roughly: 1h reading and inventorying, 2h rewriting the store, 1.5h
verification and chasing the failures above, 0.5h on images and documentation.

## 7. Extra features I added

- Wishlist that persists to `localStorage` and renders a rail.
- Recently-viewed rail, fed by opening the quick view.
- Shareable state: category, sort and search are mirrored into the URL, so a shared
  `?q=masala` link filters on arrival.
- A free-delivery progress bar that names the exact shortfall.
- Toast messages, a visible skip link, `<noscript>` fallback with prices in plain text,
  JSON-LD `Product` markup, Open Graph and Twitter cards, and a copy-to-clipboard
  pincode sanitizer.

## 8. With more time I would…

Visually review the generated imagery with a model or a human that can actually see it —
that is the largest untested assumption in this submission. Then run Lighthouse and axe
properly rather than my own approximations of them, add unit tests for `totals()` so the
rule arithmetic is pinned down in CI rather than in a throwaway browser script, add a
service worker for offline browsing, and build a real product detail page instead of
relying on the quick-view dialog.
