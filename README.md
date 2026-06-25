# Meridian Ops — Operations App (Standalone Demo)

A purchase-order **operations app** for a medical-supply distributor: every order moves
through a gated, task-by-task workflow where each step is owned by a department and nothing
can be skipped. This repository is a **single self-contained HTML demo** of that app, running
entirely in the browser on synthetic data.

> **There is a real, working version.** The production app is built with **Next.js + Supabase
> + Vercel** — Postgres with row-level security, realtime updates, Google sign-in, and email
> notifications. That codebase is private. **What you see here is a standalone HTML demo** that
> recreates the same screens and flows with **mock, anonymized data** so it can be explored
> without a backend, login, or deployment.

> **Synthetic data only.** "Meridian Medical Supply Co." is a fictional company; every
> customer, order, price, and staff name here is made up.

**▶ Live demo:** https://riegodavid-git.github.io/meridian-ops/
&nbsp;·&nbsp; click any demo account (password: `password`).

![Operations overview](docs/pages/overview.png)

---

## What the app does

A distributor was running its purchasing out of one shared spreadsheet that ~30 people edited
at once — no ownership, no workflow, frequent data loss. This app replaces that with a system
where every purchase order is a small **dependency graph of tasks** (blocked → ready → done),
each owned by the right role, with proof required at every step. Completing a task unlocks the
next one and notifies its owner; conditional steps (canvassing, client confirmation) only
appear when there's a shortage.

## Demo accounts

Every password is `password`. Each role lands on its own home screen and sees only what it's
allowed to.

| Username | Role | Sees |
|---|---|---|
| `admin` | Super Admin | Everything + the permission matrix |
| `manager` | Management | Operations overview + all dashboards (read-only) |
| `sales` | Sales / Encoder | Encode POs, confirm with clients |
| `warehouse` | Warehouse | Stock checks, prepare & load, inventory (no prices) |
| `purchasing` | Purchasing | Canvassing and supplier receivings |
| `logistics` | Logistics Head | Schedule deliveries, fleet, exceptions |
| `driver` | Driver | Phone view of own deliveries only |
| `billing` | Billing / Collections | Invoices, payments, document returns |
| `newuser` | Viewer | Nothing yet — admin must assign a role |

## Tech

| | Demo (this repo) | Production app (private) |
|---|---|---|
| Frontend | Single-file **vanilla JS + HTML + CSS** | **Next.js 16** (App Router), **React**, **TypeScript** |
| Styling | Hand-written CSS, Geist font | **Tailwind CSS 4**, same design tokens |
| Data | In-memory synthetic seed (deterministic) | **Supabase / Postgres** with **row-level security** |
| Auth | Pick-a-role demo login | Supabase Auth + **Google sign-in** |
| Realtime / email | — | Supabase Realtime + **Resend** notifications |
| Hosting | GitHub Pages (static) | **Vercel** |

The demo intentionally has **no backend and no dependencies** — open `index.html` and it runs.
The production app enforces the same permissions at the database level (RLS), so reads and
writes are safe even though the demo simulates them client-side.

---

# Page-by-page tour

## Sign in

![Sign in](docs/pages/login.png)

The entry screen lists all nine demo accounts, one per role. Click a row to sign in instantly —
there's no real authentication in the demo, it's a role switcher so you can see the app from
every angle. In the production build this screen is Supabase Auth with **Google sign-in**
restricted to company Workspace accounts; here it's swapped for a one-click picker. The left
panel makes clear this is a browser-only re-creation of a real Next.js/Supabase app.

## Dashboards

The home screen is role-aware. Operational staff get a personal task feed; management gets a
control-tower overview.

### My Tasks (operational staff)

![My Tasks](docs/pages/my-tasks.png)

Everyone who isn't management lands here, and it answers one question: *what's waiting on me?*
Three counters (ready for you, waiting on others, done recently) sit above a grid of "ready"
task cards — each shows the PO number, the step (Stock check, Prepare & load…), the customer,
and an **age badge** that turns amber after 2 days and red after 4. "Waiting on others" lists
your blocked tasks and names the exact task and role you're waiting on. This shot is signed in
as **Warehouse**, so the sidebar is already trimmed to what that role can touch.

### Operations overview (management)

![Operations overview](docs/pages/overview.png)

Management and admins get a control-tower view instead. Four KPI tiles — in-flight orders, on
hold, delayed deliveries, and overdue invoices (with the outstanding peso amount) — each link
straight to the relevant page. Below them, **Needs your decision** surfaces every on-hold order
with its reason and how long it's been stuck, **Bottlenecks** ranks the oldest in-stage orders,
and **Delivery exceptions** lists active delays. The whole page is designed so a manager can
walk in and immediately see the three or four things that actually need a human.

## Orders & workflow

### Orders

![Orders](docs/pages/orders.png)

The full order book. Filter chips across the top jump to any workflow stage (with live counts),
plus Mine / All / On hold; the search box filters by PO number or customer as you type. Each row
shows the stage badge, fulfillment type (deliver vs client pickup), how many days it's sat in
its current stage, and the total — which is **hidden for roles without the prices permission**.
Click any row to open the order.

### Order detail — the task graph

![Order detail](docs/pages/order-detail.png)

This is the heart of the app. Line items show quantity, unit price, and a per-line status
(in stock / shortage / substituted). The **Workflow** panel renders the order's task graph as a
vertical timeline — done steps are green checks, the current step is a ready ring, later steps
are blocked — and every completed step carries its **evidence chip** (count sheet, sales
invoice, loaded goods, signed DR, deposit slip…). The right rail summarizes the customer, the
linked invoice (with days-late if overdue), and the delivery (driver, vehicle, slot, who
received it). In the real app, completing a step is **gated server-side on uploading that proof
first**, then flips the next task to *ready* and notifies its owner.

### Pipeline board

![Pipeline board](docs/pages/pipeline.png)

A read-only Kanban of every in-flight order — one column per stage (stock check → canvassing →
… → collection), plus a separate red **On hold** column. Cards show the PO, customer, an age
badge once they go stale, and a rush flag. Above the board, a Bottlenecks table calls out the
orders stuck longest. This is the oversight view: management sees the whole floor at a glance
and clicks into anything.

### Canvassing

![Canvassing](docs/pages/canvassing.png)

When a stock check finds a shortage, that line drops here for sourcing. Each shortage line is
laid out side by side across suppliers with their quoted prices, and the cheapest is flagged
**best**. Purchasing picks a quote, which records it and lets the order advance to client
confirmation. (Signed in as **Purchasing** here — note the trimmed sidebar.)

## Inventory

![Inventory](docs/pages/inventory.png)

Stock on hand by location, where **every number is the sum of an append-only movement ledger**
rather than a mutable counter. KPI tiles track total SKUs, low-stock items, open receivings, and
locations; a location switcher (Warehouse / Production / Secondary Store) and a search box filter
the table, and anything under its reorder point is flagged **Low**. The production app extends
this page with receiving, transfers between locations, stock counts, lot/expiry tracking, and
adjustments that require a reason.

## Logistics

### Schedule

![Schedule](docs/pages/schedule.png)

The delivery planner. A **Preparation queue** on the left lists orders ready to schedule (rush
first); the **day board** on the right is a timeslot × vehicle grid, each cell holding a delivery
card with its status (scheduled / dispatched / delivered / delayed). KPI tiles count what's
ready, what's going out today, what's delayed, and how many drivers are on shift.

### My Deliveries (driver)

![My Deliveries](docs/pages/my-deliveries.png)

The driver's phone view — and the cleanest demonstration of access control in the app: the
sidebar collapses to just the three things a driver needs. Each stop is a card with the customer,
address, contact, and exactly what to deliver, plus large **Navigate / Problem / Add photo /
Mark delivered** buttons. Marking delivered is **gated on attaching the signed delivery receipt
first**; completed stops drop into "Done today" with who received them.

## Billing & collections

![Billing](docs/pages/billing.png)

Invoices, payments, and collections in one place. KPI tiles track the to-invoice queue, open
invoices, overdue (with amount), and recently collected. The main table lists open invoices with
due dates and **days-overdue badges**; a **collections aging** panel buckets receivables into
0–30 / 31–60 / 61–90 / 90+ days, alongside a to-invoice queue and a paid-recently feed.

## Oversight

### Team Workload

![Team workload](docs/pages/workload.png)

Who's holding what. Per-role counters of tasks *ready right now* sit above a table of every open
task across all orders — the step, its owner role, whether it's ready or blocked, and how long
it's aged. It's the view a manager uses to spot a department that's underwater.

### Activity

![Activity](docs/pages/activity.png)

A straight audit trail — every order event (created, stock-checked, quoted, invoiced, loaded,
scheduled, delivered, paid, held, resumed) with the actor and timestamp, newest first. Each entry
links back to its order. In production this is backed by an append-only `activity_log` table.

## People

### Contacts

![Contacts](docs/pages/contacts.png)

A combined directory of customers and staff with search. Customers show their address and contact
person; staff show their role and email. Type chips distinguish the two.

### Calendar

![Calendar](docs/pages/calendar.png)

One place for everything time-bound — deliveries, task due dates, and meetings — as a month grid
with colored event dots plus an agenda list below. Management can toggle between their own
calendar and the whole team's.

## Admin

### Permissions matrix

![Permissions matrix](docs/pages/admin-permissions.png)

The live permission matrix: **roles across the top, permissions down the side**, grouped into
pages / orders / workflow / inventory / logistics / billing / tasks / collaboration / data /
admin. Tick or untick a box and the current user's navigation and available actions update on the
spot. Super Admin is always fully granted and locked. In the real app these grants drive
**Postgres row-level-security policies**, so they're enforced at the database — not just hidden
in the UI.

### Users

![Users](docs/pages/admin-users.png)

The roster: every account with its email, assigned role, department, and status. This is where a
Viewer with "no access yet" gets promoted into a real role.

### Settings

![Settings](docs/pages/admin-settings.png)

App configuration — the company name and short label shown in the sidebar, the **aging
thresholds** that drive those amber/red badges, and the delivery timeslots used by the scheduler.

## Account

### Profile

![Profile](docs/pages/profile.png)

The signed-in user's own account — name, email, role, department, status, and how many
permissions their role carries.

### No-role state

![No access yet](docs/pages/viewer.png)

What a brand-new account sees: **nothing**. The Viewer role grants zero permissions, so the app
default-denies — the sidebar is bare and the page explains that an admin must assign a role first.
It's the visible proof that access is opt-in, not opt-out.

---

_Standalone demo of a real Next.js + Supabase operations app. Synthetic data only._
