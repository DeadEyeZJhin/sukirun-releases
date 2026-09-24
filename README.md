# SukiRun

An offline-first order-taking app for field sales agents, built for a noodle manufacturer and
distributor in Bacolod City, Philippines.

**Android APK · 4.7 MB · one HTML file · works with no signal · 55 releases**

[**Download the latest release →**](https://github.com/DeadEyeZJhin/sukirun-releases/releases/latest)

---

## What it does

A sales agent walks into a sari-sari store, adds the order up on the phone in front of the
shopkeeper, and sends it to the office. The phone is often handed across the counter, so the
form is blank every time and nothing on it is a surprise. There is no signal in half the
places they go, so everything works offline and syncs later.

The whole app is one HTML file. The Android build is that file inside a WebView wrapper; the
same file opens in a browser for the office.

---

## The systems inside it

### 1. Taking an order
- Product grid with typed quantities and `−`/`+` steppers, live running total, sticky cart bar.
- Cart sheet listing every line, each one editable without leaving it.
- Per-line **sold-as** unit — the same product ordered by piece, pack, box or case, with the
  unit and the price frozen onto the line so a price change next month cannot rewrite what
  was agreed today.
- Shop details, notes, optional GPS pin and optional photo.
- **Copy summary** — the order as plain text, formatted for pasting into a Messenger group
  chat, with an optional paste number in front of the shop name.
- Share sheet, and a day report of every order the agent booked today.
- **The unfinished order**: leave the form half-done and an amber card follows you onto every
  screen naming the shop, with a badge on the tab. Send, Clear or Throw it away ends it.

### 2. The catalogue
- 200+ products on a live phone; 76 ship inside the APK with photographs.
- Price, unit, pack size, pieces-per-unit, hidden flag, product groups and folders.
- Prices published from the office and pulled by every phone; a price that changes under an
  open order form updates the card **and** says so at the top of the form, and typed
  quantities survive.
- Product search, "this shop bought last time", best sellers, grouped and default views.
- Delete and bring back, with removals travelling to other phones as tombstones.

### 3. Shops
- Saved stores with owner, contact, address, landmark, GPS pin and photo.
- Order history and running balance per shop.
- **Due for a visit** — a shop nobody has ordered from, or marked visited, in N days, with the
  threshold set in Settings. Blue strip on the agent's screen, a count for the office.
- Stores are shared through the office, so a shop added on one phone appears on the others.

### 4. Roles and access
- Five entry cards: Place an Order · Field Agent · Admin/Secretary · Warehouse · Delivery.
- Four account roles enforced in the database, not the screen: `customer`, `staff`,
  `warehouse`, `admin`.
- Postgres row-level security on every table, plus `security definer` RPCs for every write.
- Agents see their own orders; the office sees everything; a customer sees only their own.
- Optional 4-digit PIN on the Admin door for a phone left on a table.
- Admin → Team: who has signed up, their role, their monthly target, and an on/off switch
  that stops a lost phone reaching the shared list.

### 5. The shared list (sync)
- Offline-first. Every write lands on the phone first and is queued.
- Push and pull over Supabase RPCs, with per-row merge rather than last-write-wins.
- Conflict rules written down and versioned by a `REPAIR_ID`, so a phone that already lost
  rows can be told to re-ask once.
- Tombstones for removed orders, stores and products, so a deletion actually propagates.
- "Take the office's copy" for a phone whose copy has drifted.
- Swipe down to refresh, a status dot, last-checked time, and an honest count of what is
  genuinely waiting to send.

### 6. Pictures
- Product and shop photos in Supabase Storage, with the bucket policies to match.
- Photos taken on a phone stay on that phone; a shop's own picture travels so every phone
  can see the shop.
- IndexedDB on the device rather than localStorage, so the catalogue's images do not fight
  the 5 MB quota.
- Full-screen viewer, add/replace/remove, and orphan cleanup.

### 7. Maps, offline
- MapLibre GL with OpenFreeMap tiles.
- Tiles cached into IndexedDB, so the map still draws where there is no signal.
- Pin a shop from the Stores tab or while taking an order; correct a pin later.
- Nearest-shops list computed from GPS alone, works with the map offline.

### 8. Money
- Payments recorded against an order, with balance (*utang*) carried per shop.
- Server-side payment merge, so two phones recording against the same order cannot lose one.
- Monthly target per agent, set by the office, with progress on My Profile.

### 9. Stock
- Optional per-product stock tracking — off by default, turned on for one product at a time.
- `move_stock` RPC so two phones cannot double-spend the same count.
- A product with no stock can be hidden from the order screen.

### 10. The office
- Orders list with search, date window, sort, and "whose orders" — applied before the counts,
  so the four figures at the top always describe what is on screen.
- Status ladder Sent → Confirmed → Packed → Delivered, plus cancelled and removed.
- Mark as Packed and Mark as Delivered live on Admin and on the delivery phone through one
  shared code path; the account is the record, not the phone.
- **Undo is not cancel** — it reads the step to go back to out of the recorded stamps and
  writes itself into the timeline.
- Edit an order and the app records *what* changed, not just that somebody changed it.
- Timeline per order: who, what, when.

### 11. Delivery and packing
- Assign a driver, pack, deliver, record the payment on the doorstep.
- Works offline and syncs on the way back.

### 12. Data, settings and recovery
- Storage stats, and a one-file JSON backup of every order, shop and price on the phone.
- An error log — "What went wrong" — with the last failures, for a phone you cannot reach.
- Maps, catalogue, visit threshold, name and appearance settings.
- An in-app roadmap listing what is planned, with the reason for each.

### 13. Shipping
- Python build tooling: one script inlines everything into a single HTML file, another builds
  and signs the APK, a third cuts the release.
- Updates over GitHub Releases — the app reads `update.json` from `/releases/latest/`,
  compares the version code, and offers the download.
- Catalogue and photos published to this repository from the same tooling.

---

## Not in it, on purpose

Invoices · money owed as a screen · reports for the boss · anything after "packed" ·
undo-send · discounts and free items · push notifications.

Each one was measured against the real data first and would have drawn an empty screen. They
are parked with the number that parked them, not with a shrug.

---

## Built with

Vanilla JavaScript — no framework, no build step for the app itself · Supabase (Postgres,
Auth, Row Level Security, Storage) · MapLibre GL · IndexedDB · Android WebView wrapper built
with `aapt2`/`d8`/`apksigner` · Python build and release tooling.

## By the numbers

| | |
|---|---|
| JavaScript | ~13,500 lines, 513 functions |
| CSS | ~1,700 lines |
| PostgreSQL | ~3,900 lines across 24 migrations |
| Database | 8 tables, 29 functions, row-level security on all of them |
| Catalogue | 200+ products, 76 with photographs in the APK |
| Releases | 55 |
| APK | 4.7 MB |

---

## What is in this repository

This is the **release** repository — the public address every phone checks. The app's source
is kept private.

| | |
|---|---|
| [Releases](https://github.com/DeadEyeZJhin/sukirun-releases/releases) | the signed APK and `update.json` for each version |
| `catalog.json` | the office's published price list |
| `catalog-seed.csv` | the same catalogue in a form a spreadsheet can open |
| `photos/` | product photographs served to the app |

The app reads `update.json` from `/releases/latest/download/update.json`, compares the version
code against its own, and offers the download. Phones already running SukiRun update from
**Settings → About → Check for updates**.

## Author

Guile Pitogo — Bacolod City, Negros Occidental.
[deadeyezjhin.github.io](https://deadeyezjhin.github.io/)
