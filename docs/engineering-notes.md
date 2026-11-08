# Engineering notes

Decisions I would defend, problems I found in my own system, and what I would do next.

## Scope: what I left out on purpose

The tagline is "Connection, Not Commerce", and it shaped the scope. ReWeave has **no payments, no in-app negotiation, no chat, no ratings and no automatic matching**. The product's job ends at the moment two people agree to connect. That kept the system small enough to build properly: a consent flow, a trust layer and a notification layer.

## Decisions

### 1. Consent as a database object
A contact request is a row with a status, a direction and a reference to what it was about. Because it is data, not just a button, it can be logged, audited by an admin, and protected by database rules: one reply per weaver per order, and one *active* request per weaver, supplier and listing. The rule for the weaver-to-supplier case was refined later so that a declined request can be asked again but an active one cannot be duplicated.

### 2. Roles from the signed session
Admin status is signed metadata on the Clerk account, carried in the session token. The request gate can route admins and members without touching the database. The cost is a one-time identity-provider setting (the token must include that metadata). The benefit is one fewer database query on every request, and no way for the browser to claim to be an admin.

### 3. Privileged database key on the server, with row-level security underneath
The API checks identity, role and ownership itself. Row-level security is on for all 16 tables as a backstop, not the main control. I chose this because the permission rules (who can ask whom, in which direction, about what) are business logic that is clearer in application code than in policies, and because it keeps the browser from ever holding a database key.

### 4. Create members on first use, not by webhook
No webhook means no webhook failures to reconcile. The first portal load creates the record. The trade-off is that a person who signs up but never opens the portal has no record, which is acceptable here.

### 5. Fail open for the safety systems
If the rate limiter or the maintenance lookup is unreachable, requests are allowed. For a small community platform, being available matters more than being strict when a helper service blips.

## Problems I found in my own system

After the app was feature complete, I audited it against the live database and the code, then fixed what mattered. Four examples:

| Problem | What was wrong | Fix |
|---|---|---|
| **ID uploads failed for every member** | The verification screens uploaded Aadhaar and PAN, but the database rule only allowed older document types. The code had changed without a migration. | Found by reading the live database's rules. Added a migration, then confirmed it on the live schema. |
| **Migrations could not rebuild the database** | The security-policy file ran before the tables it secured. One file used a function that only exists with an extension installed. The first file began by dropping the whole schema and ended by inserting fake members. | Reordered, removed the destructive and seed parts, and added a storage-bucket migration. A test now applies every file to an empty database and checks the result. |
| **Accepting needed a phone number** | A weaver who had only a WhatsApp *group link* could not accept a request, although the form allowed a link. | Accept now requires a number **or** a group link. |
| **Shared links looked unfinished** | Order previews showed raw database codes (`cotton_grey`), and a listing link lost the item for new visitors. | Previews now show readable labels, and the sign-up redirect carries the real listing id. |

A fifth, smaller one: a user-supplied name was inserted into an HTML dialog without escaping. It is now escaped.

## Testing and delivery

- **56 unit tests** (Node's built-in runner) cover the Zod validation schemas, the escaping, search-sanitising, client-IP and upload-signature helpers, and the lists of public and admin routes.
- **Migration tests** apply all migration files, in order, to an empty in-memory Postgres (PGlite, with small stand-ins for Supabase's own schemas). They check that each file applies, that file numbers have no gaps, that no file drops a schema or inserts members, and that the result has the 16 tables, the verification types and the three buckets. I checked the test can fail by restoring the old broken file name.
- **CI** runs lint, the tests and a production build on every push and pull request. **Lint** has no errors.
- **Not done yet:** browser (end-to-end) tests.

## Honest limitations

- Contact requests have an expiry date that nothing enforces.
- Account deletion is partial: uploaded ID documents are kept.
- The admin action log has a table and is read by the analytics page, but actions are not yet written to it.
- A weaver's reply can be withdrawn through the API, but no screen offers it.
- Yarn prices are entered by hand by an admin.
- The weaver's loom listings are visible only to the weaver, and the feature that would match them to orders is switched off.
- The interface is English only, with no push notifications and no real-time updates (the bell polls every 60 seconds).

## What I would do next

1. Enforce contact-request expiry and write the admin action log.
2. Finish account deletion, including stored documents.
3. Add end-to-end tests for the consent flow.
4. Offer a Hindi and Urdu interface.
5. Add WhatsApp notifications, since that is where the users already are.
6. Replace the hand-entered yarn prices with a live feed.

Related: [architecture](architecture.md), [data model](data-model.md) and [user journeys](user-journeys.md).
