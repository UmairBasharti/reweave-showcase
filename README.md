# ReWeave Malegaon

> **Connection, Not Commerce.** A verified discovery registry that connects fabric buyers, powerloom weavers and raw-material suppliers in Malegaon, India. Contact details are shared only when the other person says yes.

**[Live demo](https://reweave-red.vercel.app)** · [Architecture](docs/architecture.md) · [Data model](docs/data-model.md) · [User journeys](docs/user-journeys.md) · [Engineering notes](docs/engineering-notes.md) · [All screens](docs/screens.md)

<p align="center">
  <img src="assets/screenshots/public/01-landing-hero.webp" alt="ReWeave landing page hero with the tagline Connection, Not Commerce" width="860">
</p>

> **About this repository.** It holds documentation, diagrams and screenshots only. The application's source code is in a private repository, and I'm happy to walk through it on a call.

| At a glance | |
|---|---|
| **What** | A verified discovery registry with consent-based contact exchange |
| **For** | Fabric buyers, powerloom weavers and raw-material suppliers in Malegaon, India |
| **Status** | Live demo and pilot (not a commercial service) |
| **My scope** | Sole developer: product design, front end, API, database, deployment and testing |
| **Stack** | Next.js 16, React 19, Supabase, Clerk, Vercel |
| **Size** | 40 pages, 43 API routes, 16 tables, 56 unit tests |

---

## The problem

Malegaon is one of India's large powerloom clusters.

**Scale**

- About **6,000 production units** (my research summary on the [live Research page](https://reweave-red.vercel.app/research)).
- **3.5 lakh powerlooms and 2 lakh workers**, as stated by Union Textile Minister Giriraj Singh ([post on X, 25 Oct 2025](https://x.com/girirajsinghbjp/status/1981973931805626818)).
- About **1.2 crore metres of cloth a day** (Research page). The minister's post puts the 2025 festive peak at **3.5 crore metres a day**.
- **₹3,974 Cr a year** of turnover (Research page). The minister's post cites **₹1,500 Cr of grey fabric in a single festive month**.
- **83% of output** leaves the city for dyeing and finishing in Surat, Rajasthan and Bhiwandi, so Malegaon earns raw-cloth prices (Research page).

**Structure (my own field study)**

- **80% of looms are plain powerlooms**, and output is mostly grey cotton (45%) and grey polyester (38%).
- **78% of buyers are local traders and brokers**, 14% are garment manufacturers buying directly, and 8% come through digital or B2B platforms.
- **46% of loom owners** are interested in modernising but hesitant because of past failures.

Weavers depend on one channel, and real buyers have no trusted way to reach them directly. **ReWeave is a single direct window**: one place where direct buyers reach Malegaon's weavers and suppliers, with every contact consent-based.

*The scale figures are quoted from the sources named. The structure percentages come from my own field study of the cluster (interviews with loom owners, weavers, traders and others, chosen by judgmental and convenience sampling), so treat them as indicative rather than statistically representative. None of these are results of ReWeave's own usage.*

## The idea: consent before contact

ReWeave is **not a marketplace**. There are no payments, no negotiation and no chat. People find each other, and a **WhatsApp contact is shared only after the other person explicitly accepts**. Every request is logged. Price, samples and payment then continue on WhatsApp, outside the platform.

```mermaid
sequenceDiagram
    autonumber
    actor B as Buyer
    participant R as ReWeave
    actor W as Weaver
    B->>R: Posts an order (fabric, specification, quantity)
    W->>R: Browses open orders and replies with a note
    R-->>B: Notification: new reply
    B->>R: Requests the weaver's contact (logged)
    R-->>W: Notification: contact request
    W->>R: Accepts ("Share your contact?" dialog)
    R-->>B: Weaver's WhatsApp revealed
    R-->>W: Buyer's WhatsApp revealed
    Note over B,W: From here, price, samples and payment continue on WhatsApp
```

The same pattern connects **weavers to material suppliers**. Buyers ask weavers, weavers ask sellers, and nobody can be contacted without agreeing.

<table>
<tr>
<td width="33%"><img src="assets/screenshots/public/04-landing-hub-buyer.webp" alt="Role panel for buyers: Direct Sourcing Network"><br><sub><b>Buyers</b> post what fabric they need and review replies from verified weavers.</sub></td>
<td width="33%"><img src="assets/screenshots/public/05-landing-hub-weaver.webp" alt="Role panel for weavers: Verified Market Access"><br><sub><b>Weavers</b> answer orders, list their looms and source raw materials.</sub></td>
<td width="33%"><img src="assets/screenshots/public/06-landing-hub-seller.webp" alt="Role panel for suppliers: Targeted Material Distribution"><br><sub><b>Suppliers</b> list yarn and materials where weavers are actively looking.</sub></td>
</tr>
</table>

## How it works

<table>
<tr>
<td width="50%"><img src="assets/screenshots/buyer/buyer-05-post-order.webp" alt="Buyer's form for posting a fabric order"><br><sub><b>1. A buyer posts an order.</b> Fabric type, specification, quantity and notes. Unfinished forms are saved as drafts.</sub></td>
<td width="50%"><img src="assets/screenshots/weaver/weaver-02-buyer-orders.webp" alt="Weaver's feed of open buyer orders"><br><sub><b>2. Weavers browse the live feed</b> (search, a Verified-only filter) and reply with a note.</sub></td>
</tr>
<tr>
<td width="50%"><img src="assets/screenshots/buyer/buyer-03-order-replies.webp" alt="Replies panel showing two weavers, both with contact revealed"><br><sub><b>3. The buyer reviews replies</b> and, when ready, requests a weaver's contact.</sub></td>
<td width="50%"><img src="assets/screenshots/weaver/weaver-08-accept-dialog.webp" alt="Dialog asking the weaver to confirm sharing their contact"><br><sub><b>4. The weaver decides.</b> "Your WhatsApp number will be shared… This is permanent and will be logged."</sub></td>
</tr>
<tr>
<td width="50%"><img src="assets/screenshots/weaver/weaver-07-connections-from-buyers.webp" alt="Weaver's connections page with a pending contact request"><br><sub>Pending requests wait for the weaver's decision. Nothing is revealed until they accept.</sub></td>
<td width="50%"><img src="assets/screenshots/buyer/buyer-04-connections.webp" alt="Buyer's connections page showing contact shared"><br><sub><b>5. Both sides see each other's contact</b> and continue off-platform.</sub></td>
</tr>
</table>

Weavers find suppliers the same way: they browse raw materials and request a supplier's contact, and the supplier accepts or declines.

<table>
<tr>
<td width="33%"><img src="assets/screenshots/weaver/weaver-04-raw-materials.webp" alt="Weaver browsing raw material listings with photos"><br><sub>Weavers browse supplier listings.</sub></td>
<td width="33%"><img src="assets/screenshots/seller/seller-05-add-material-form.webp" alt="Supplier's form for adding a material listing"><br><sub>Suppliers publish listings with up to three photos.</sub></td>
<td width="33%"><img src="assets/screenshots/weaver/weaver-09-connections-to-suppliers.webp" alt="Weaver's supplier requests: shared, awaiting, declined"><br><sub>Every request has a visible state: shared, awaiting or declined.</sub></td>
</tr>
</table>

## Trust: identity is checked by a person

Members upload **Aadhaar, PAN and GST documents** (at least two). An admin reviews them and approves or rejects, and approved members get a green **Verified** badge that shows on profile cards, replies and public pages. Weavers can filter the feed to verified buyers only. ID documents sit in a **private storage bucket** and are only ever opened through **one-hour signed links**. An admin portal also gives the review queue, a member registry, a contact ledger, announcements and platform settings.

<table>
<tr>
<td width="50%"><img src="assets/screenshots/buyer/buyer-07-get-verified.webp" alt="Verification screen with Aadhaar, PAN and GST upload slots"><br><sub>Upload, human review, Verified badge.</sub></td>
<td width="50%"><img src="assets/screenshots/public/16-about-how-contact-is-disclosed.webp" alt="About page listing how contact is disclosed, step by step"><br><sub>The disclosure rules are stated up front.</sub></td>
</tr>
</table>

## Sharing that works outside the app

Orders and listings can be shared to WhatsApp (native share sheet on phones, a share window on desktop). The link opens a **public page with no contact details**, and chat apps show a **generated preview card**: a fabric swatch for an order, and the seller's own photo for a listing. The cards show only what the public page already shows, never a contact detail. A visitor who joins from the link lands with the reply dialog already open (for recent items).

<table>
<tr>
<td width="33%"><img src="assets/screenshots/public/23-link-preview-order.webp" alt="Preview card for a buyer order: a cotton swatch, the fabric name and the quantity in metres"><br><sub><b>An order.</b> A swatch in the fabric's colour, the fabric and the quantity.</sub></td>
<td width="33%"><img src="assets/screenshots/public/24-link-preview-listing.webp" alt="Preview card for a material listing: the seller's photo, the material name and the price"><br><sub><b>A listing.</b> The seller's first photo, the material and the price.</sub></td>
<td width="33%"><img src="assets/screenshots/public/25-link-preview-site.webp" alt="Preview card for the site link, with the tagline Connection, Not Commerce"><br><sub><b>The site.</b> A fixed card, so the link previews instantly.</sub></td>
</tr>
</table>

<table>
<tr>
<td width="40%"><img src="assets/screenshots/public/21-public-order-page.webp" alt="Public order page, signed out, showing no contact details"><br><sub>The public page: no sign-in, no contact details.</sub></td>
<td width="40%"><img src="assets/screenshots/public/22-public-listing-page.webp" alt="Public material listing page, signed out"><br><sub>A listing's public page.</sub></td>
<td width="20%"><img src="assets/screenshots/public/m06-public-order-page.webp" alt="Public order page on a phone"><br><sub>On a phone.</sub></td>
</tr>
</table>

## Built for phones

Every screen is responsive, so the same flows work on a phone.

<table>
<tr>
<td width="25%"><img src="assets/screenshots/weaver/weaver-m01-dashboard.webp" alt="Weaver dashboard on a phone"></td>
<td width="25%"><img src="assets/screenshots/weaver/weaver-m02-buyer-orders.webp" alt="Buyer orders on a phone"></td>
<td width="25%"><img src="assets/screenshots/weaver/weaver-m04-accept-dialog.webp" alt="Accept dialog on a phone"></td>
<td width="25%"><img src="assets/screenshots/seller/seller-m02-my-listings.webp" alt="Supplier listings on a phone"></td>
</tr>
</table>

More screens, by role, are in [docs/screens.md](docs/screens.md).

---

## Architecture

```mermaid
flowchart LR
    U["Browser<br/>phone or desktop"] --> P
    subgraph V["Vercel"]
        P["Proxy<br/>maintenance, sign-in, roles, account status"] --> A["Next.js App Router<br/>pages and API routes"]
        C["Daily cron<br/>08:00 UTC digest"] --> A
    end
    A --> CL["Clerk<br/>sign-in and sessions"]
    A --> SB[("Supabase<br/>Postgres and Storage")]
    A --> UP["Upstash Redis<br/>rate limiting"]
    A --> RS["Resend<br/>email"]
    U -.-> OB["Sentry, PostHog, Clarity<br/>analytics only after consent"]
```

| Layer | Technology |
|---|---|
| Front end and API | Next.js 16 (App Router), React 19, JavaScript, Tailwind CSS 4, shadcn/ui, GSAP |
| Authentication | Clerk |
| Data and files | Supabase (PostgreSQL and Storage), 16 tables, 3 storage buckets |
| Validation | Zod |
| Email | Resend and React Email |
| Rate limiting | Upstash Redis |
| Monitoring | Sentry, PostHog, Microsoft Clarity (analytics are consent-gated) |
| Hosting | Vercel (auto-deploy on every push, daily cron) |

Roughly 25K lines of JavaScript across about 150 files, with 40 pages and 43 API routes. Details: [docs/architecture.md](docs/architecture.md).

## Engineering highlights

- **A consent model enforced by the database.** Contact requests have an explicit lifecycle, and unique rules stop duplicates: one reply per weaver per order, one active request per weaver, supplier and listing. See [data model](docs/data-model.md).
- **Role-based routing with no database call.** Admin status is carried in the signed session token, so the request pipeline can keep admins and members apart cheaply.
- **Members are created on first use** instead of by webhook, with the profile cached for five minutes and refreshed the moment it changes.
- **Defence in depth.** Every write checks identity, role and ownership. Row-level security is on for all 16 tables as a backstop. The browser never receives a database key.
- **Careful uploads.** File type and size checks, plus a check of the file's real leading bytes, so a renamed file is rejected. IDs go in a private bucket.
- **Security headers and rate limits.** A content security policy, HSTS, frame denial, and three rate-limit tiers (public, signed-in, admin).
- **Link previews for every shared post.** A 1200×630 JPEG card per order and listing (a fabric swatch, or the seller's photo), built only from fields the public page already shows. The site link uses a fixed card.
- **Notifications that respect people's time.** Instant in-app alerts, and a once-a-day email digest for members who have been away.
- **Privacy by default.** Analytics start only after a visitor accepts cookies.

## Quality and process

- **56 unit tests** cover input validation, escaping, search sanitising, upload checks and the public and admin route lists. They also **apply all database migrations to an empty throwaway Postgres** and check the result: 16 tables, no seed data, the expected storage buckets.
- **CI** runs lint, tests and a production build on every push and pull request.
- **Lint** has no errors.
- Found and fixed along the way: migrations that could not rebuild a database in order (one even began by dropping the schema), and a missing migration that made ID uploads fail. The story is in [engineering notes](docs/engineering-notes.md).

## Known limitations and what's next

I'd rather be upfront about what is unfinished:

- Contact requests carry an expiry date, but nothing enforces it yet.
- Account deletion is partial (uploaded ID documents are kept).
- Yarn prices are entered by hand, with no automatic feed.
- The interface is English only, and there are no push notifications.
- There are no browser (end-to-end) tests yet.

Next: expiring requests, complete account deletion, a Hindi/Urdu interface, WhatsApp notifications and a live price feed.

## About

Designed and built by **[Umair Basharti](https://github.com/UmairBasharti)**, as the idea, the research and the code. It grew out of a semester of field research on the Malegaon textile cluster (my MCA research project at Dr. B.V. Hiray College of Management & Research Centre, Malegaon), which I then turned into a working product. The research was published in the Akshardhara Research Journal (E-ISSN 3048-8095), presented at an Interdisciplinary National Conference at Bhusawal Arts College in September 2025, and is summarised on the live site's Research page.

**Connect:** [LinkedIn](https://www.linkedin.com/in/umair-basharti) · [GitHub](https://github.com/UmairBasharti)

**Try it:** [reweave-red.vercel.app](https://reweave-red.vercel.app). Create an account (an email code confirms it) and pick any role. Sign-in currently runs on development-mode auth keys.

---

© 2026 Umair Basharti. All rights reserved. This repository contains documentation and screenshots only; the source code is private. All screenshots use demo accounts and sample data with fictional names.
