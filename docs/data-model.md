# Data model

ReWeave stores everything in a Supabase project: **PostgreSQL** for data and **Storage** for files. There are **16 tables**, all with row-level security switched on, and **3 storage buckets**.

These diagrams were generated from the project's migrations: every migration file was applied, in order, to an empty database, and the tables, columns and relationships were read back. Only the key columns are shown.

## Core: people, orders and consent

```mermaid
erDiagram
    users ||--o{ requirements : "buyer posts"
    users ||--o{ weaver_responses : "weaver writes"
    requirements ||--o{ weaver_responses : "receives"
    users ||--o{ capacity_listings : "weaver lists looms"
    users ||--o{ material_listings : "seller lists"
    users ||--o{ contact_disclosures : "initiates"
    users ||--o{ contact_disclosures : "receives"
    requirements |o--o{ contact_disclosures : "about"
    weaver_responses |o--o{ contact_disclosures : "from"
    material_listings |o--o{ contact_disclosures : "about"
    users ||--o{ verification_submissions : "uploads"

    users {
        uuid id PK
        text clerk_user_id UK
        text role "buyer, weaver or seller"
        text account_status "active, restricted, suspended"
        text verification_status "unverified, pending, verified"
        text display_name
        text whatsapp_number
    }
    requirements {
        uuid id PK
        uuid buyer_id FK
        text fabric_type "five fixed codes"
        text specification
        int quantity_metres
        text status "open or closed"
    }
    weaver_responses {
        uuid id PK
        uuid requirement_id FK
        uuid weaver_id FK
        text status "pending, withdrawn, contact_revealed, rejected"
    }
    contact_disclosures {
        uuid id PK
        text disclosure_type "buyer_weaver or weaver_seller"
        uuid initiator_id FK
        uuid recipient_id FK
        uuid requirement_id FK
        uuid response_id FK
        uuid material_listing_id FK
        text status "pending, accepted, declined, expired"
    }
    capacity_listings {
        uuid id PK
        uuid weaver_id FK
        text loom_type
        int available_looms
    }
    material_listings {
        uuid id PK
        uuid seller_id FK
        text material_name
        text category "yarn, accessories, dyes, machinery"
        text availability "continuous, seasonal, limited"
    }
    verification_submissions {
        uuid id PK
        uuid user_id FK
        text document_type
        text storage_path "private bucket"
        text status "pending, approved, rejected, info_requested"
    }
```

## Supporting tables attached to a member

```mermaid
erDiagram
    users ||--o{ notifications : "receives"
    users ||--o{ digest_logs : "digest sent to"
    users ||--o{ unit_photos : "has"
    users ||--o{ admin_actions : "performs"

    users {
        uuid id PK
    }
    notifications {
        uuid id PK
        uuid user_id FK
        text type
        bool is_read
        bool email_sent
        text link
    }
    digest_logs {
        uuid id PK
        uuid user_id FK
        int notification_count
        timestamptz sent_at
    }
    unit_photos {
        uuid id PK
        uuid user_id FK
        text storage_path
        int photo_order
    }
    admin_actions {
        uuid id PK
        uuid admin_id FK
        text action_type
        uuid target_user_id FK
    }
```

## Standalone tables

| Table | Purpose |
|---|---|
| `drafts` | Saved, unfinished forms (one per member and form type) |
| `yarn_prices` | Entries for the yarn-price ticker, maintained by admins |
| `system_config` | Platform switches such as maintenance mode and whether registration is open |
| `broadcasts` | History of announcements sent by admins |
| `platform_alerts` | Alerts shown on the admin threat monitor |

## The core table: `contact_disclosures`

Every contact request, in both directions, is a row here. It is the platform's audit ledger, and an admin can read all of it.

- **Direction is fixed.** `buyer_weaver` means a buyer asked a weaver. `weaver_seller` means a weaver asked a seller. Sellers never initiate and buyers never receive.
- **It records what the request was about**: the order, the reply, or the material listing.
- **It has a lifecycle**, shown below. Timestamps record when it was requested and answered, and a default expiry date is stored.

```mermaid
stateDiagram-v2
    [*] --> pending : requester asks, request is logged
    pending --> accepted : recipient consents, both contacts revealed
    pending --> declined : recipient says no, nothing is shared
    pending --> expired : reserved (not yet enforced)
    accepted --> [*]
    declined --> [*]
```

## Other lifecycles

```mermaid
stateDiagram-v2
    direction LR
    state "Order" as O {
        [*] --> open
        open --> closed
        closed --> open
    }
    state "Verification" as V {
        [*] --> unverified
        unverified --> pending : first document uploaded
        pending --> verified : admin approves
        pending --> unverified : admin rejects, member can resubmit
    }
```

- **Reply:** `pending`, then `contact_revealed` when the buyer's request is accepted, or `rejected` when it is declined.
- **Account:** `active`, `restricted` (can browse, but cannot post, reply, request or add a material) or `suspended` (blocked everywhere). Admins set it.

## Rules the database enforces

| Rule | Why it matters |
|---|---|
| One reply per weaver per order | No duplicate replies |
| One contact request per requester, recipient and order | No repeated asking |
| For weaver to seller: one *active* (pending or accepted) request per weaver, seller and listing | Stops request spam, but allows asking again after a decline |
| Fixed allowed values for roles, statuses, fabric types, categories and document types | Bad values are rejected at the source |

## Files

| Bucket | Visibility | Contents |
|---|---|---|
| `profile-photos` | public | Profile photos, cover and gallery photos |
| `material-images` | public | Supplier listing photos |
| `verification-docs` | **private** | ID documents. Only reachable through one-hour signed links an admin opens |

Related reading: [architecture](architecture.md), [user journeys](user-journeys.md) and the [screens gallery](screens.md).
