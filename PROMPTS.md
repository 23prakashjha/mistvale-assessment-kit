# Prompt log — Mistvale Tea Co. store

Every prompt I used, in order, exactly as I typed it. Tool/type/outcome/why for each.

Dead ends are included on purpose — the brief asks for them.

---

## Prompt 1
- Tool: Claude (Claude Code, this session)
- Type: code
- Prompt:
  > Read the README first. Improve it with AI. Fix the bugs, redesign it, make shopping smooth and create proper product images, a banner and a logo with an AI image tool. Expect about 4 hours, and please not more than 6. Log your prompts. Keep every prompt, for code and for images, exactly as you typed it, in PROMPTS.md, and write a short NOTES.md.
- Outcome: accepted
- Why: The task brief. Read `README.md` + `BRAND.md` + `index.html` first, then inventory every bug and constraint break before writing any code, because the score is weighted on catching what is already there.

---

## Prompt 2
- Tool: Claude (Claude Code, this session)
- Type: other
- Prompt:
  > (inventory of the existing `index.html`, worked out by reading the file, not by asking an LLM)

  Reproduce this list of bugs, with file line references, so nothing gets missed:
  - `index.html:244` `cart` is `null` on a first visit, so `cart.find(...)` throws in `addToCart`
  - `index.html:327` cart badge uses `cart.length` (number of lines) not total quantity
  - `index.html:312` price is scraped out of the DOM (`innerText.replace('₹','')`) → breaks R1, and `parseInt("1,299")` returns `1`
  - `index.html:275` `filterProducts('all', …)` filters `category == 'all'`, which matches nothing → clicking "All" empties the grid
  - `index.html:275` chips use `'green'` (lowercase) against category `"Green"` → Green teas never appear
  - `index.html:282` sort comparators are inverted (`a.price > b.price` for "low to high")
  - `index.html:282` `shown` is a reference to `PRODUCTS`, so `.sort()` reorders the source data
  - `index.html:267` image click handler closes over `var i` after the loop, so `openQuickView(list[i])` throws on every click
  - `index.html:292` `ids.indexOf(p.id)` returns `0` for the first match, which is falsy → the first result is always dropped
  - `index.html:290` search resets category and sort (breaks R7) and has no debounce, so fast typing renders stale results out of order
  - `index.html:376` `qty = input.value + 1` is string concatenation → "1" + 1 = 11
  - `index.html:382` `minus()` can drive quantity to 0 or negative
  - `index.html:388` `cart.splice(i)` with one argument deletes every item from `i` to the end
  - `index.html:359` `items.addEventListener` is re-registered on every `renderCart()`, so handlers stack up
  - `index.html:341` subtotal uses the price cached in the cart, not `PRODUCTS` (breaks R1)
  - `index.html:393` coupon is case-sensitive, has no ₹399 minimum, no ₹150 cap, no Gifts exclusion, and compounds on every click (breaks R3)
  - `index.html:367` free shipping tests the raw subtotal against 599, not the post-discount amount (breaks R4)
  - `index.html:368` the "add ₹X more" message never switches to a "you have free shipping" state, and the bar can exceed 100%
  - `index.html:401` `checkPin` has no `.catch()`, so an invalid pincode rejects and the UI stays on "Checking…" forever (breaks R8)
  - `index.html:406` the pincode is interpolated into `innerHTML` unescaped
  - `index.html:258` products with no rating render "☆☆☆☆☆ (0)" — a claim about review count that isn't true, and any rating renders 5 filled stars
  - `index.html:310` no stock or 5-per-product cap (breaks R2), and sold-out teas still get a working Add to cart button (breaks R6)
  - `index.html:422` countdown targets `2025-11-01`, which is in the past, so it counts up into negative numbers
  - `index.html:158` checkout shows `alert('Order placed!!')` and never submits `#checkout-form` (breaks constraint 4)
  - `index.html:434` a newsletter pop-up opens by itself after 2s (breaks BRAND §8.9)
  - `index.html:117` "AS SEEN ON SHARK TANK INDIA", `:175` "Rated 4.9/5 by 10,000+ happy customers", `:115` "BEST TEA IN THE WORLD", `:97` a Diwali sale — all invented claims (breaks constraint 7)
  - `index.html:11` `* { transition: all .4s !important }` and `:12` `*:focus { outline: none !important }` (breaks BRAND §6 and §7)
  - `index.html:97` `<marquee>`, `:46` a blinking `SALE!!!` animation (breaks BRAND §7)
  - `index.html:5-9` jQuery, animate.css, Font Awesome and four Google families (breaks constraints 1 and 2)
  - `index.html:3` `<title>Home</title>`, no meta description, no viewport, no `lang`, no canonical, no OG, no JSON-LD, four `h1`s, no `alt` text, no labels
  - `index.html:193` footer wordmark reads "MistVale", which contradicts the BRAND.md spelling rule — but it sits inside the do-not-edit legal block, so this is a question for the team, not a change
- Outcome: accepted
- Why: Used as the working checklist. Every item became either a fix or a line in NOTES.md. One of them (the free-shipping threshold) turned out to be unspecified in the brief itself — R4 refers to "the free-shipping threshold" without giving a number, the old code used 599 and the old announcement strip told customers 499 — so it became a question rather than a silent guess.

---

## Prompt 3
- Tool: Claude (Claude Code, this session)
- Type: other
- Prompt:
  > What did we do so far?
- Outcome: accepted
- Why: I had lost track of which fixes were already applied and which were only planned. Answering this from the files rather than from memory stopped me re-applying edits and double-declaring functions — which did in fact happen twice later (a duplicated `canTakeFocus` and a duplicated "Cart drawer" banner), and was caught by grepping for the declaration before continuing.

---

## Prompt 4
- Tool: Claude (Claude Code, this session)
- Type: other
- Prompt:
  > Continue if you have next steps, or stop and ask for clarification if you are unsure how to proceed.
- Outcome: accepted
- Why: The browser suite was down to a handful of failures. Rather than keep guessing which side was at fault, this pushed me into the habit that actually settled every one of them: write a throwaway diagnostic that dumps the computed state, then decide whether the page or the assertion was wrong. That split is what the last section of NOTES.md is about.

---

## Prompt 5 — image generation
- Tool: not available in this session
- Type: image
- Prompt:
  > **I cannot reproduce these verbatim, and I am not going to invent them.**
  >
  > The nine product images, two hero banners, the logo mark and the social card were
  > generated earlier in this engagement, before the current session. The exact prompt
  > strings are not recoverable from my present context, and writing plausible-looking
  > ones would be the single easiest way to make this whole log worthless. So this entry
  > is a gap in the submission, deliberately left open rather than filled in.
- Outcome: rejected (nothing was generated; this entry is a record of a gap)
- Why: The brief asks for the image prompts to be logged. I can describe the brief I
  worked to — one consistent art direction across all nine, square 1:1 framing for the
  products, a wide crop for the banner, brand palette only — but that is a paraphrase of
  a brief, not the prompt I typed. It is recorded as a paraphrase for that reason.
  No image-generation tool is exposed to me in this session either, so the images could
  not be regenerated to recover the originals.

---

## Prompt 6 — prompts I gave to tools, not to a chat model
- Tool: Claude Code subagents and Node scripts (this session)
- Type: other
- Prompt:
  > Task tool, `explore`/`general` subagents — e.g. "Explore this codebase very
  > thoroughly. I need to know how the cart, coupon and search code is wired, where
  > the event listeners are registered, and which files hold the business rules. Return
  > file paths and line numbers only."
  >
  > Headless Chrome via the DevTools protocol (`mv-browser.mjs`) — the equivalent
  > instruction was always the same shape: drive the real page, assert one rule, print
  > the value that was actually produced rather than a pass/fail guess.
- Outcome: accepted
- Why: Most of this task is verification rather than generation, so most of my
  "prompts" were assertions. Recording them here so the log is complete; the scripts
  themselves are throwaway and are not part of the deliverable.
