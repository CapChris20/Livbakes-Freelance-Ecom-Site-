# Architecture Decision Records — LivBakes (Freelance E-commerce Site)

LivBakes is a storefront for a home bakery selling macaron boxes (5 flavors at $14.99–$15.99) with pickup or delivery. It's a freelance build for a small business owner: plain HTML/CSS with **Snipcart** for cart and checkout, hosted on **Netlify**.

These records were reconstructed from the files in this repo (`product.html`, `contactus.html`, `netlify.toml`).

---

# ADR-001: Static HTML/CSS + Snipcart instead of Shopify or a custom backend

## Context
The client is a solo baker with a handful of products, no technical staff and a tiny budget. They needed online ordering with card payments and a pickup/delivery choice, and they need to be able to keep it running without a developer.

## Alternatives Considered
- **Shopify**: Full-featured, but a monthly subscription (~$39/mo Basic) is a big fixed cost for a home bakery, and the custom look would mean theme work.
- **Square Online (free plan)**: Free and simple, but limited design control and Square branding.
- **Custom backend (Node + Stripe + a database)**: Total control, but the client would need a developer for every change, plus hosting, security and PCI work.
- **Instagram DMs + Venmo**: Free, but manual, error-prone and unprofessional.

## Decision
Build static pages (`index`, `product`, `aboutus`, `contactus`) styled by hand. **Snipcart v3** handles the cart: products are declared inline with `data-item-*` attributes on "Add to Cart" buttons, and a custom field `Fulfillment Method: Pickup|Delivery` captures fulfillment. Snipcart crawls `data-item-url` to validate prices, and its hosted checkout handles payment through the connected gateway (Stripe/PayPal).

## Rationale
- **Cost:** no monthly platform subscription; Snipcart charges per transaction.
- **Maintainability:** adding a flavor means copying one HTML block.
- **Security:** card data never touches the site (PCI is handled by Snipcart and the gateway). Snipcart's server-side crawl of `data-item-url` stops anyone editing the price in the browser.

## Tradeoffs
- **Gained:** a working store with zero backend, quick to build, with a custom look.
- **Cost:** the product catalog is hard-coded HTML, so price changes need an HTML edit and redeploy. Snipcart fees apply on top of the payment processor's fees, and inventory and order management happen in Snipcart's dashboard.

## What I'd Change
Move products into a small JSON file (or a headless CMS like Decap) that renders the buttons, so the owner can edit prices without touching HTML. I'd also compare total fees against Square Online's free plan once order volume is known.

## Measurement
- Catalog: **5 products** (2 × $14.99, 3 × $15.99).
- Fees per $15.99 box: Snipcart's transaction fee (**2%** at its published standard rate ≈ $0.32) plus Stripe (2.9% + $0.30 ≈ $0.76), so roughly **$1.08, about 6.7% per order**. Check current Snipcart pricing, which also includes a monthly minimum for low-volume stores.
- **Not yet tracked:** orders per month and cart abandonment (available in the Snipcart dashboard).

---

# ADR-002: Netlify static hosting

## Context
The site needs free HTTPS hosting, and Snipcart needs a public URL it can crawl to validate products.

## Alternatives Considered
- **GitHub Pages**: Also free, but Netlify's drag-and-drop deploy suits a non-technical owner better.
- **Shared hosting (GoDaddy etc.)**: A monthly cost and manual FTP uploads.

## Decision
Deploy to **Netlify** (`livbakes-ecom.netlify.app`). `data-item-url` points at the production product page.

## Rationale
Free HTTPS and a CDN, with no build step needed.

## Tradeoffs
- **Gained:** $0 hosting.
- **Cost:** `netlify.toml` sets `publish = "/Users/captainchris20/Desktop/LivBakes"`, an absolute path on a local Mac, so **Git-based deploys from this repo will fail**. It only works with manual drag-and-drop or CLI deploys from that machine.

## What I'd Change
Set `publish = "."` so Netlify can build from the repo. Fix filename casing: the HTML refers to `fruity.JPG`, `chocolate-1.JPG`, `cpb.JPG`, `olivia.JPG` and others, but the files in the repo are lowercase `.jpg`. Netlify's servers are case-sensitive, so **those images will 404 in production** (macOS hides the problem locally).

## Measurement
- Image weight: the largest assets are `olivia.jpg` (1.09 MB), `contactbg.png` (0.81 MB), `baking.png` (0.56 MB) and `aboutbg.png` (0.52 MB). Converting to WebP at display size would cut total page weight substantially.

---

# ADR-003: `mailto:` contact form instead of a form backend

## Context
Customers need a way to ask about custom orders.

## Alternatives Considered
- **Netlify Forms**: Free tier, and submissions show up in the dashboard and get emailed. It only needs a `data-netlify="true"` attribute.
- **Formspree / EmailJS**: Free tiers, with spam filtering.

## Decision
`<form action="mailto:…">` plus a visible email link.

## Rationale
No setup and no third party.

## Tradeoffs
- **Gained:** simplicity.
- **Cost:** `mailto:` forms open the visitor's mail app, if one is configured, and often fail silently on mobile and in webmail. The owner's personal email is exposed to scrapers.

## What I'd Change
Switch to **Netlify Forms** (it's already on Netlify, so it's a one-attribute change) with honeypot spam protection.

## Measurement
Not measured yet. Track inquiries per month after switching to Netlify Forms.

---

# ADR-004: No authentication, database or data isolation layer

## Context
Customers buy as guests, and the owner manages orders in Snipcart.

## Decision
There are no user accounts on the site. Snipcart stores customer and order data, and its optional customer accounts can be turned on in its dashboard.

## Rationale
Guest checkout converts best for small, one-off purchases, and it avoids storing any customer data on the site.

## Tradeoffs
There's no loyalty or reorder history on the site itself.

## What I'd Change
Nothing at this scale.

## Measurement
Not applicable.
