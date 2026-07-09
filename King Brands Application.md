# King Brands — Production Tool · Session Handover
_Last updated: 2026-07-09_

## What this is
A staff-only web tool that turns a **client order (in boxes)** into an exact **raw-material shopping list grouped by supplier**, with a ready-to-send **WhatsApp order** for each supplier. Covers two brands: **Gold Joy** (syrups) and **Nectarly** (pure honey).

---

## 🔗 Live test link (deployed, isolated from public site)
- **URL:** https://kb-ordertool.vercel.app
- **Password:** `king2024`
- Vercel project: `2linket/kb-ordertool` (separate project — the public King Brands site is untouched)
- Deploy source folder: `Desktop\kb-ordertool\` (a copy of the source file as `index.html` + `assets\kb-logo.jpeg`)

## 📁 Files
| File | Purpose |
|---|---|
| `Desktop\KingBrands\ordertool-test.html` | **SOURCE** (single file, ~800 lines, matches site navy/gold UI). Edit this. |
| `Desktop\kb-ordertool\index.html` | Deploy copy of the source (copied before each `vercel --prod`) |
| `Desktop\kb-ordertool\assets\kb-logo.jpeg` | Logo used by the tool |
| `Desktop\KingBrands\.git` | Source committed here (commit `5556a3e`) |

### To redeploy after editing source
```
cp "Desktop/KingBrands/ordertool-test.html" "Desktop/kb-ordertool/index.html"
cd "Desktop/kb-ordertool" && vercel deploy --prod --yes
```
Clean domain `kb-ordertool.vercel.app` auto-points to newest prod deploy.

---

## How the tool works (current state)
**Flow:** Login (`king2024`) → **Brand picker** → select boxes on Gold Joy and/or Nectarly → **View order summary** (combined) → tap WhatsApp per supplier.

### Product model
- 1 box = **24 units** = 24 bottles + 24 caps + 24 front labels + 24 back labels + 1 physical box.
- **Gold Joy:** 4 flavours × 2 sizes (500g / 375g). Cap colour = flavour: Original=**white**, Cinnamon=**orange**, Lemon=**green**, Ginger=**pink**.
- **Nectarly:** Pure Honey, **white cap**, both sizes. Same packaging as Gold Joy; only the fill differs.

### Suppliers & rules (round UP = ceil, no +1 extra)
| Supplier | Contact | WhatsApp | Supplies | Pack rule |
|---|---|---|---|---|
| **Ankot Plastics** | Vule | 071 983 1321 | Bottles + caps | 500g bottles = **348/bailer**, 375g = **170/bailer**, caps **1000/bag per colour** |
| **Box Brothers** | Coenraad | 083 570 4271 | Boxes | **25/bailer** (round to nearest 25) |
| **Jutagz** | Kim | 072 293 6319 | Labels | bulk **min 14 000** front + 14 000 back; not combined |
| **Marico River Honey** | Marico | 076 111 1145 | Pure honey (Nectarly) | **10 boxes = 1 bucket**; whole buckets, round up, min 1 |
| **Sugar on Tap** | Kobus | 066 472 0969 | Glucose (Nectarly) | **15 boxes = 1 bucket** (⅓ per 5 boxes); whole buckets, min 1 |
| _Syrup_ | — bulk stock, no supplier | — | Syrup fill (Nectarly) | **5 boxes = 1 bucket**; info only, no order |

### Combine rule (important)
When ordering **both** brands, **Ankot** (bottles+caps) and **Box Brothers** (boxes) totals are **added together across both brands, then rounded once**.
- White caps merge = Gold Joy Original caps + all Nectarly caps.
- Other cap colours = Gold Joy only.
- **Jutagz labels do NOT combine** (different artwork) — shown per brand.
- Marico / Sugar are Nectarly-only.

### Features
- **− / +** steppers on every bailer/bag/box/bucket count → supplied total + WhatsApp message update live (min 0).
- **×** on each cap line = "we have these in stock" → removes from that supplier's WhatsApp message (toggle back with **+**).
- Prefilled **wa.me** WhatsApp links (staff taps send; no paid Business API).
- **Save / Load / Delete** orders (localStorage, this device only).
- Fully **mobile-responsive** — product cards + order lines stack 1-per-row on phones.

### Verified (Playwright)
GJ 25×Original500 + 10×Ginger375 + Nectarly 10×Honey500 →
Ankot 500g **3 bailers**, 375g **2 bailers**, white caps merged 840→**1 bag**, pink **1 bag**; Box 500g 35→**2 bailers (50)**, 375g **1 bailer**; Marico **1 bucket**, Sugar **1 bucket**; Jutagz GJ+Nectarly labels separate. No mobile horizontal scroll at 390px.

---

## Architecture notes (for next dev / next session)
- Single self-contained HTML file. No build step, no dependencies (Google Fonts only).
- Views: `#brandPick`, `#view-goldjoy` (`#prodGrid`), `#view-nectarly` (`#necGrid`), `#view-summary` (`#summaryResults`), `#view-history`. Routed by `openBrand('goldjoy'|'nectarly'|'summary'|'history')` + `backToBrands()`.
- Config near top: `FLAVOURS`, `SIZES`, `HONEY`, `SUPPLIERS`, `BUCKET`, `CAPS_BAG`, `BOX_BAILER`, `LABEL_MIN`, `PASSWORD`.
- Calc: `readGoldJoy()` + `readNectarly()` → `computeCombined()` → `renderSummary()`.
- Order lines: `orderLine({...})` renders a stepped line carrying `data-count/pack/unit/need/needword/fraglabel/mode`; `updateLine()` recomputes text + `data-wa`; `rebuildWA(supplierEl)` builds the wa.me href from included lines. `mode:'bucket'` variant for honey/glucose.
- **Password is client-side** (`const PASSWORD`) → visible in source. Fine for a test; NOT real security.
- **All data is localStorage** → per-device, does not sync between phones.

---

## ✅ Decisions locked this session
- Storage: **local test now**, Supabase backend = phase 2.
- Login: shared password now (`king2024`).
- Rounding: **ceil** (round up on remainder; exact stays exact).
- WhatsApp: **prefilled wa.me link**, staff taps send.
- Both bottle sizes = 24/box. Nectarly honey/glucose/syrup fill = **same per box** regardless of size.
- Buckets: **whole buckets only**, round up, min 1.
- Architecture going forward: **public website stays separate**; build ONE **internal staff portal** (order + stock + timesheets, role-based) on its own URL.

---

## 🔜 Next steps / open ideas (discussed, NOT yet built)
1. **Two roles / logins:**
   - **Owner** (PW + Andru) → full: ordering, stock, timesheets.
   - **Staff** (workers) → only **stock update** + **timesheet clock-in**; ordering fully hidden.
2. **Stock module:** staff update current warehouse quantities; later feed into order tool ("only order the difference").
3. **Timesheets:** clock in/out, auto hours, weekly totals per worker; owner review view.
4. **Turn `kb-ordertool` into the internal PORTAL** (rename, add roles + stock + timesheets).
5. **Backend (Supabase, free tier)** — REQUIRED for staff features to be real: multi-device sync, secure logins, know who clocked in. Owner sees staff stock + hours.
6. Eventually: integrate/link from public site via a small "Staff Login" button (optional).

### Info needed before building the portal/backend
- Staff names (for timesheets / who-clocked-in).
- Exact warehouse stock items to track.
- Owner password(s) vs staff password — shared staff login, or per-worker logins?
- Prototype-first (local, to approve UX) **or** straight to Supabase backend?

---

## Related memory
See `~/.claude/projects/C--Users-pwdeb/memory/project_king_brands_website.md` (Production/Supplier Order Tool section) for the persistent record.
