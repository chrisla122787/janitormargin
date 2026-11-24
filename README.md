# JanitorMargin

A bid & profitability calculator for small commercial cleaning / janitorial
companies. Tagline: **"Price cleaning contracts that actually make money."**

JanitorMargin is v2 of the tool originally built as "CleanBid Pro" — renamed
because two existing products already use the CleanBid Pro name.

Pricing: **$19/month founding-member pricing** for the first 25 customers,
locked in while the subscription stays active. Standard price is **$29/month**
after the founding spots are gone. The calculator and a watermarked proposal
preview are free; Pro unlocks the clean PDF, saved bids, the bid dashboard,
and backup export.

Plain static HTML/CSS/JS — no build step, no backend.

## Files

| File | What it is |
|---|---|
| `index.html` | Landing page: promise, how it works, free/Pro features, a static sample proposal, founding pricing, privacy + billing/cancellation notes. |
| `app.html` | The tool: bid calculator, watermarked proposal preview, Pro PDF proposal (jsPDF via CDN), bid dashboard with statuses and totals, backup export/import + CSV export. |
| `README.md` | This file. |

## Before launch

- In `index.html` and `app.html`, replace the `STRIPE_LINK` placeholder hrefs
  with the real Stripe subscription payment link for JanitorMargin
  ($19/month founding price). There are two in `index.html` (hero + pricing)
  and one in `app.html` (Pro gate card).
- Contact email in the footer is `chrisla122787@gmail.com`.

## Free vs Pro

**Free (no payment, no account):**

- Full calculator — every input, live bid math, low-margin warning.
- Watermarked proposal preview rendered on screen (diagonal
  "PREVIEW — JanitorMargin" watermark), showing company name, client name,
  scope, and price.

**Pro (subscription):**

- Clean branded PDF download: company logo + contact info, client and
  facility names, editable scope-of-work checklist, exclusions line, service
  frequency, monthly investment, per-visit price, annual value, validity
  period, payment terms, and a client signature block.
- Saved bids + bid dashboard (statuses, pipeline/won totals, duplicate).
- Backup export (JSON) / import-restore, and CSV export of bids.

**The customer PDF and preview never show internal numbers.** Labor costs,
payroll burden, overhead, and margin appear only in the private "Your numbers"
panel — the proposal carries prices and scope only.

## ⚠️ MVP access note (read before launch)

Pro unlocking is **client-side only** for launch validation:

- The "I already subscribed — unlock Pro" button in `app.html` simply sets a
  `localStorage` flag (`janitormargin.pro.v1 = "1"`) in the visitor's own
  browser. There is **no real payment verification** — no server checks the
  Stripe subscription, and anyone who finds the button can unlock Pro without
  paying.
- This is a deliberate MVP trade-off to validate demand cheaply on a $0
  static stack (the landing page says so in plain words: unlocking is on the
  honor system during opening validation).
- Before scaling past validation, replace this with a real entitlement check
  (e.g. a small serverless function that verifies the Stripe subscription by
  email before issuing an unlock token).

## The bid formula

All math is in `calculate()` in `app.html`:

```
cleansPerMonth  = cleansPerWeek × 52 ÷ 12            (≈ 4.333 weeks/month)
laborBase       = hoursPerClean × laborRate
laborPerClean   = laborBase × (1 + payrollBurden% / 100)
directPerClean  = laborPerClean + suppliesPerClean
costPerClean    = directPerClean × (1 + overhead% / 100)
monthlyCost     = costPerClean × cleansPerMonth
monthlyBid      = monthlyCost ÷ (1 − margin% / 100)   ← margin is a % of the PRICE
perVisitPrice   = monthlyBid ÷ cleansPerMonth
annualValue     = monthlyBid × 12
profitPerMonth  = monthlyBid − monthlyCost
pricePerSqFt    = monthlyBid ÷ squareFootage
```

Payroll burden is optional (default 0%) and covers employer taxes, workers
comp, and benefits — typically 12–20%. It applies to labor cost only.

Why divide by `(1 − margin)` instead of just adding the margin on top of cost:
a profit margin is a share of the final price, not a markup on cost. Pricing
this way means the margin the user picks (say 25%) is the share of the bid
price that is actually profit. Marking cost up by 25% would only deliver a
20% true margin.

Worked example — defaults with burden 0% (10,000 sq ft, 3 cleans/week,
3 hrs/clean, $22/hr, $15 supplies, 10% overhead, 25% margin):

- Cost per clean: ($66.00 labor + $15.00 supplies) × 1.10 = **$89.10**
- Monthly cost: $89.10 × 13.0 cleans = **$1,158.30**
- Recommended monthly bid: $1,158.30 ÷ 0.75 = **$1,544.40**
- Per-visit: **$118.80** · Annual: **$18,532.80** · Profit/month: **$386.10**

Same defaults with payroll burden at 15%:

- Labor per clean: $66.00 × 1.15 = **$75.90**
- Cost per clean: ($75.90 + $15.00) × 1.10 = **$99.99**
- Monthly cost: **$1,299.87** · Monthly bid: **$1,733.16**
- Per-visit: **$133.32** · Annual: **$20,797.92** · Profit/month: **$433.29**

Warnings: margin under 10% shows a plain-language caution; margin ≥ 100% is
rejected as unpriceable.

## Data & privacy

Everything lives only in the visitor's own browser (`localStorage`):

| Key | Contents |
|---|---|
| `janitormargin.bids.v1` | Saved bids (inputs, name, status, date). |
| `janitormargin.settings.v1` | Company name/phone/email + logo (data URL). |
| `janitormargin.pro.v1` | Pro unlock flag (see MVP note above). |

Nothing is sent to a server. The app tells users plainly: "Your data is stored
only in this browser on this device — export a backup regularly." Backup
export downloads a JSON file with bids + settings; import restores it. CSV
export lists bids with their computed prices.

Bid statuses: Draft, Sent, Follow-up, Won, Lost. Dashboard totals:
active pipeline = Draft + Sent + Follow-up monthly bids; won = Won monthly
bids.

## Deploy

Any static host works — copy the three files (or at least the two HTML files)
to the host as-is:

- **Netlify / Vercel:** drag-and-drop the folder, or connect the repo; no
  build command, publish directory = this folder.
- **Cloudflare Pages:** same — framework preset "None", no build command.
- **GitHub Pages:** fine for demos/preview only — GitHub Pages terms prohibit
  primarily commercial SaaS use, so don't run the paid product from there.

The only external request the app makes is the jsPDF CDN script in `app.html`
(cdnjs). If it fails to load, everything except the PDF button still works
and the user gets a clear message.
