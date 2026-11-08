# User journeys

Four journeys cover the whole product. Each diagram follows what the screens actually do.

## 1. Buyer to weaver: the main flow

```mermaid
sequenceDiagram
    autonumber
    actor B as Buyer
    participant R as ReWeave
    actor W as Weaver
    B->>R: Posts an order (fabric type, specification, quantity, notes)
    R-->>W: Order appears in the Buyer Orders feed
    W->>R: Opens "Reply to Order", writes an optional note
    R-->>B: Notification: New Weaver Response
    B->>R: Views replies, presses Request WhatsApp
    Note over B,R: The dialog says the request is logged and the buyer's contact is shared too if accepted
    R-->>W: Notification: New Contact Request
    W->>R: Opens My Connections and chooses Accept or Decline
    alt Accept
        Note over W,R: "Share your contact?" This is permanent and will be logged
        R-->>B: Request Accepted: weaver's contact revealed
        R-->>W: Buyer's contact shown
    else Decline
        R-->>B: Request Declined: nothing is shared
    end
    Note over B,W: Contact is on WhatsApp from here. Price, samples and payment happen outside ReWeave
```

Points worth noticing:
- **A weaver needs a WhatsApp number or group link on their profile before they can accept.** Otherwise the server asks them to add one.
- **One reply per order.** A second attempt is refused, and replies to closed orders are refused.
- The order card shows an **unread badge** ("2 New") that stays in sync across the buyer's devices.

## 2. Weaver to supplier

```mermaid
sequenceDiagram
    autonumber
    actor S as Supplier
    participant R as ReWeave
    actor W as Weaver
    S->>R: Publishes a material (name, category, availability, price indication, 1 to 3 photos)
    W->>R: Browses Raw Materials, opens a listing
    W->>R: Presses Request Contact
    Note over W,R: Only one active request per weaver and listing. After a decline they can ask again
    R-->>S: Notification: New Contact Request
    S->>R: Accepts or declines, from My Listings or My Connections
    alt Accept
        R-->>W: Supplier's contact revealed
        R-->>S: Weaver's contact shown
    else Decline
        R-->>W: Request Declined
    end
```

Sellers never start a conversation and buyers never receive requests. Direction is fixed so nobody can be cold-contacted.

## 3. A shared link, from WhatsApp to a reply

```mermaid
sequenceDiagram
    autonumber
    actor O as Owner
    participant C as Chat app
    actor V as Visitor
    participant PG as Public page
    participant J as Join screen
    participant PT as Weaver portal
    O->>C: Shares the order link (phone share sheet or desktop window)
    C->>PG: Fetches the page to build a preview
    PG-->>C: Title, description and a generated 1200x630 card
    V->>C: Sees the preview card and taps it
    V->>PG: Opens the public order page (no sign-in, no contact details)
    alt Not signed in
        PG-->>V: Join ReWeave to connect
        V->>J: Registers as a weaver and confirms the emailed code
        J->>PT: Redirected straight to that order
    else Signed-in weaver
        PG-->>V: Respond in Portal
        V->>PT: Opens that order
    end
    PT-->>V: The Reply to Order dialog opens by itself (recent items)
    V->>PT: Sends a reply
    PT-->>O: Notification: new reply
```

What each visitor sees on the public page:

| Visitor | Button |
|---|---|
| Not signed in | Join ReWeave to connect |
| Signed in as the owner | Manage in Dashboard |
| Signed in as a weaver | Respond in Portal |
| Signed in as another buyer or a supplier | Go to My Dashboard |

A closed order shows an "Order Closed" overlay, and an inactive listing shows "Listing Unavailable". The public pages never show WhatsApp numbers, email addresses, replies, or who contacted whom.

<table>
<tr>
<td width="33%"><img src="../assets/screenshots/public/23-public-order-link-preview.webp" alt="Generated link preview card for an order"><br><sub>The preview card in the chat.</sub></td>
<td width="33%"><img src="../assets/screenshots/public/21-public-order-page.webp" alt="Public order page for signed-out visitors"><br><sub>The public order page.</sub></td>
<td width="33%"><img src="../assets/screenshots/public/22-public-listing-page.webp" alt="Public material listing page"><br><sub>The public listing page.</sub></td>
</tr>
</table>

## 4. Getting verified

```mermaid
sequenceDiagram
    autonumber
    actor M as Member
    participant R as ReWeave
    participant ST as Private storage
    actor A as Admin
    M->>R: Uploads at least two of Aadhaar, PAN, GST
    R->>R: Checks type, size and the file's real bytes
    R->>ST: Stores the documents in a private bucket
    R-->>M: Status moves to Pending
    R-->>A: Notification: new verification document
    A->>R: Opens the queue, views a document through a one-hour signed link
    alt Approve
        R-->>M: Verified badge and an approval email
    else Reject
        R-->>M: Back to Unverified, can resubmit, rejection email
    else Request more information
        R->>R: Note saved, submission stays pending
    end
```

Being verified adds a green tick on profile cards, replies and public pages, and it powers the weaver feed's **Verified buyers only** filter. It does not unlock features. Anyone can register and use the platform, and verification is about trust.

Related: [architecture](architecture.md), [data model](data-model.md) and the [screens gallery](screens.md).
