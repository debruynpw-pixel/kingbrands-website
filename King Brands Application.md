# King Brands — Staff Application · Session Handover
_Last updated: 2026-07-09_

## What this is
An internal, staff-only web app for King Brands (**Gold Joy** syrups + **Nectarly** honey). One login page, role-based. Three tools inside: **Order Tool**, **Daily Stock Sheet**, **Timesheets & Pay**. Runs on any phone. Backed by **Supabase** so every phone syncs to the owner dashboard live.

---

## 🔗 LIVE
- **URL:** https://kb-ordertool.vercel.app  ← one URL for everything
- **Order Tool** lives at `/ordertool-test.html?fromApp=1` (opens auto-unlocked from inside the app)
- Vercel project: `2linket/kb-ordertool` (separate — the **public King Brands website is untouched**)

## 🔑 Logins
| Who | Username | Password | Can see |
|---|---|---|---|
| **Owner** (you) | `PW` | `king2024` | Order Tool + Stock Sheets + Timesheets |
| **Owner** (Andru) | `Andru` | `king2024` | same as above |
| **Staff** | `Cindy` `Wendy` `Thabi` `Alex` | `ginger2026` | Stock Sheet + own Timesheet only |

Usernames are **case-insensitive**. Staff logins are **locked** to those 4 names. Staff can't see each other's or the owner's data.

---

## 📁 Files
| File | Purpose |
|---|---|
| `Desktop\KingBrands\staff-app.html` | **SOURCE — edit this.** The whole Staff Application (single file). |
| `Desktop\KingBrands\ordertool-test.html` | Order Tool source (opened as a section from the app) |
| `Desktop\kb-ordertool\index.html` | Deploy copy of staff-app (staff app = the site index) |
| `Desktop\kb-ordertool\ordertool-test.html` | Deploy copy of order tool |
| `Desktop\kb-ordertool\assets\kb-logo.jpeg` | Logo |
| `Desktop\KingBrands\.git` | Source repo → github.com/debruynpw-pixel/kingbrands-website |

### Redeploy after editing source
```bash
cp "Desktop/KingBrands/staff-app.html"     "Desktop/kb-ordertool/index.html"
cp "Desktop/KingBrands/ordertool-test.html" "Desktop/kb-ordertool/ordertool-test.html"
cd "Desktop/kb-ordertool" && vercel deploy --prod --yes
# then (backup): cd Desktop/KingBrands && git add -A && git commit -m "..." && git push
```
`kb-ordertool.vercel.app` auto-points to the newest prod deploy.

---

## ☁️ Supabase (cloud database — real cross-device sync)
- **Project ref:** `llsmsvhqnliarxrihtzk`
- **URL:** `https://llsmsvhqnliarxrihtzk.supabase.co`
- **Anon key + URL** hardcoded in `staff-app.html` `<script>` config (`SB_URL`, `SB_KEY`, `const sb = window.supabase.createClient(...)`). Loaded via CDN `@supabase/supabase-js@2`.
- **Tables:**
  - `stock_sheets` (id, submitter, date_str, time_str, created_at, **data** jsonb = whole sheet)
  - `timesheets` (id, name, work_date, clock_in, clock_out, net_hours)
- **RLS:** enabled, permissive `for all using(true) with check(true)` on both.
- ⚠️ **GOTCHA:** Supabase SQL-editor's green **"Run and enable RLS"** button enables RLS but **drops the policies** → re-run the `create policy ... for all` block after using it, or inserts fail with error `42501`.

**How data flows:** staff submits/clocks on their phone → writes to Supabase → owner opens dashboard → reads live from Supabase. No per-device silo anymore. Verified live end-to-end.

---

## How each tool works

### Order Tool (owner)
Boxes in → raw-material shopping list per supplier + prefilled WhatsApp orders (Gold Joy + Nectarly). Password `king2024` internally, but opening it from the app skips that (auto-unlock via `?fromApp=1`). Unchanged from before — see git history.

### Daily Stock Sheet (staff submit)
Fresh empty sheet each time. Sections:
1. **Bottles** — 500g & 375g, each = *bailers + loose bottles* (e.g. 4 bailers + 56 loose)
2. **Boxes** — 500g & 375g, each = *bailers + loose boxes*
3. **Caps** — white / green / orange / pink, each = *whole bags + partial dropdown* (—, ¼, ½, ¾)
4. **Syrup** (amount + Litres/Flowbins) & **Barcode back labels** (pure honey, rolls left)
5. **Labels Running Short** — add-a-row (label + note)
6. **Warehouse Needs** — add-a-row (coffee, toilet paper, etc.)

On submit → saves to Supabase + **emails** `pw@kingbrandsgroup.co.za` (FormSubmit) + **WhatsApp** button to Andru (`27658942768`). Owner **Stock Sheets** view = last 7, shows who submitted + date/time, expandable.

### Timesheets & Pay
- Staff: live **Clock In** / **Clock Out** (one in + one out per day, locks after out). Sees own week hours + pay so far (private).
- **Pay rule:** full day = R250 = 8h → **R31.25/hr**. Hours: gross ≤8h = paid as-is (no lunch); gross 8–9h = capped 8h (1h lunch removed); gross >9h = gross − 1h. Weekly pay = total hours × 31.25, **rounded to nearest rand** (576.66 → R577).
- Week starts **Monday**, auto-resets. Owner **Timesheets** = each employee's hours + **owed this week**, expand to Mon–Sun grid; **Past weeks** history (last 3 weeks).
- Verified: Cindy 4h + 5h + 8h = 17h = **R531**. Full day = R250.

---

## ✅ Done this session
- Built the Staff Application shell (username+password, owner/staff roles)
- Stock Sheet module (full) → email + WhatsApp + owner view
- Timesheets module → live clock, weekly pay, owner dashboard
- Fully **mobile-responsive** (verified 390px, no horizontal scroll on any screen)
- Order Tool wired in with **auto-unlock** (no second login)
- **Supabase** backend → real cross-device sync (verified live)
- Owner logins changed to **PW** + **Andru**
- Deployed live + pushed to GitHub

## ⚠️ One-time to-do (you)
- **Activate email:** first stock-sheet submit triggers a FormSubmit confirmation email to `pw@kingbrandsgroup.co.za`. Click the confirm link **once** → all future sheets email automatically.

## 🔜 Next ideas (not built)
- **Real per-user security** — currently the anon key is in the app + open policies (fine for an obscure internal tool behind the password gate). For hard lockdown → Supabase Auth (proper accounts). Bigger job.
- Auto-cleanup of old rows (currently keeps everything in DB; UI only shows last 7 sheets / 3 weeks).
- Embed Order Tool as a true in-page section (currently opens in a new tab).
- Link a "Staff Login" button from the public King Brands site (optional).

## Related memory
`~/.claude/projects/C--Users-pwdeb/memory/project_king_brands_staff_app.md`
