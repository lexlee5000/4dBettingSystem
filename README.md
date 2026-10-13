# 4D Betting System

A complete online 4D betting platform with a public results website, a members' area, an agent network, an admin back office and an Android app. Results are collected automatically, bets are settled automatically, and built-in risk controls protect the house.

> **Private and proprietary.** This repository holds a commercial betting system. Running a betting operation needs the proper licence in each market where it is offered.

---

## Contents

- [Features](#features)
- [Tech stack](#tech-stack)
- [Requirements](#requirements)
- [Installation](#installation)
- [Scheduled jobs](#scheduled-jobs)
- [Console commands](#console-commands)
- [Testing](#testing)
- [Currency and languages](#currency-and-languages)
- [Mobile app and API](#mobile-app-and-api)
- [Project structure](#project-structure)
- [Security notes](#security-notes)

---

## Features

### Betting
- **13 companies:** Magnum, Da Ma Cai, Sports Toto, Singapore Pools, Sabah 88, Sandakan, Sarawak Cash Sweep, GD Lotto, Perdana, Lucky HariHari, Nine Lotto, plus the Perdana and HariHari 3:30pm draws. The admin chooses which companies are on sale.
- **Bet types:**
  - 4D: Big, Small, 4A, 4SB, 4SC, 4SD, 4SE and 4C.
  - 3D: ABC and A.
  - Toto 5D and 6D.
- **Normal, Box and iBox** play, priced automatically.
- **Two bet forms:**
  - a simple card form;
  - a spreadsheet-style grid for many numbers, with fill-down, tick-all and a full-screen mode.
- **Several draw dates and companies on one slip.** Days when a company doesn't draw are left out automatically.
- **Bet slips:** QR code, printable, "Bet again", and a payout mark for dealers.

### Results
- **Automatic collection** from each operator's official website, cross-checked against independent result sites, with a third source as tie-breaker.
- Results confirmed by two sources are approved automatically. When sources disagree, the admin chooses which to use.
- **Result corrections** re-settle only the bets that change: unpaid winnings are cancelled and paid winnings are taken back.
- **Public results board:**
  - every company's results, plus display-only jackpots, lotto games and 3+3D;
  - a number checker that can also search permutations;
  - history by date.

### Settlement and winnings
- Bets settle automatically against verified results, using the prize rates the admin sets.
- **Winnings wait for admin approval** before they are paid into the member's wallet. Approval can be done one at a time or in bulk.
- Bets that can't be decided automatically (for example, a missing rate) are held for review.
- **Draw cancellation** refunds every stake on that draw automatically.

### Risk control
- **Payout pool:**
  - a maximum payout per number per draw;
  - a worst-case limit per draw day across all companies.
- **Maximum stake per customer per number**, set separately for each bet type.
- **Red numbers:** blocked permanently or for one draw, optionally with every permutation. Numbers that reach the pool limit are blocked automatically.
- **Dashboard** showing exposure, numbers near their limit and the profit or loss of each draw.

### Agent network
- Three levels: **agents → dealers / end users**.
- **Agent terms:** commission (a share of sales, no risk) or risk share (a share of sales and of wins). Defaults apply to every agent, and each agent can have their own terms.
- Agents give credit to dealers, approve top-ups, and see their earnings and statements.
- **End-user self sign-up** (admin can switch it on or off). End users top up by bank transfer and upload the receipt.

### Promotions
- **House jackpot** (admin can switch it on or off):
  - Jackpot 1 "Double Strike" and Jackpot 2 "1st Prize Hit";
  - funded by a percentage of stakes, with a starting amount and a cap, and every win approved by the admin;
  - shown only to members who are signed in.
- **Rebate plans:** end users get back a percentage of their net loss for each draw day, credited to their wallet automatically.

### Administration
- A dashboard covering today's bets, results to check, winnings to approve, member wallets and pool usage.
- **Wallet history** for every member. Corrections are made as new entries; past entries are never edited.
- **Excel export** of bets and winnings (streamed, up to 100k rows).
- Prize rates, draw calendar, betting close times, company bank account, sign-up, currency and language settings.

### Android app
- Everything a member can do on the website: betting (simple form and grid), slips, results, winnings, wallet and top-up.
- Bluetooth thermal slip printing, and QR scanning to open a printed slip or bet again.

### International
- **One base currency per site:** MYR, SGD, USD, AUD or AED.
- **English, 中文 and Bahasa Melayu** on the website, the admin pages, the API and the app. Each user picks their own.

---

## Tech stack

| Part | Technology |
|---|---|
| Web application | PHP 7.4, CakePHP 3.8 |
| Database | MariaDB 10.5 / MySQL 5.7+ |
| Front end | Server-rendered templates, plain CSS and JavaScript |
| Mobile app | Flutter (Android), in a separate repository (`betdata_app`) |
| Mobile API | JSON over HTTPS, bearer tokens (`/api/v1`) |
| Translations | gettext `.po` files (`src/Locale`) |

---

## Requirements

- PHP 7.4 with the `intl`, `mbstring`, `pdo_mysql`, `zip` and `xmlwriter` extensions.
- MariaDB 10.5 or MySQL 5.7+.
- Composer.
- A web server (Apache or nginx) serving `webroot/`, with HTTPS.
- Cron, for result collection and settlement.

---

## Installation

```bash
git clone <this repository> 4dbet
cd 4dbet
composer install --no-dev
cp config/app.default.php config/app.php
```

Edit `config/app.php`:

- `Datasources.default`: database host, name, user and password.
- `Security.salt`: a long random string.
- `Results.cronKey`: a random key for the result cron URLs.
- `App.defaultLocale` (optional): `en_US`.

Then create the tables and clear the caches:

```bash
bin/cake migrations migrate
bin/cake schema_cache clear
bin/cake cache clear_all
```

Sign in at `/admin` with an admin account. Then:

1. Under **Settings**, set the website name, contact details, company bank account, **base currency** and **default language**.
2. Under **Prize Rates**, enter the payout per unit for each company and bet type.
3. Under **Betting Controls**, set close times, the companies on sale, the payout pool and the stake limits.
4. Under **Draw Calendar**, check the upcoming draw dates (they are also filled in automatically every month).

`config/app.php`, `logs/`, `tmp/`, `uploads/` and `vendor/` are never committed (see `.gitignore`).

---

## Scheduled jobs

Times are Malaysian time.

```cron
CRON_TZ=Asia/Kuala_Lumpur
# Collect results and settle bets, every 10 minutes on draw evenings.
*/10 18-23 * * * cd /path/to/4dbet && php bin/cake.php results_fetch >> logs/results-cron.log 2>&1
# Keep 12 months of draw days in the calendar.
30 3 1 * * cd /path/to/4dbet && php bin/cake.php results_fetch --calendar >> logs/results-cron.log 2>&1
```

Settlement also updates agent earnings, rebates and jackpot contributions for each draw date.

---

## Console commands

| Command | What it does |
|---|---|
| `bin/cake results_fetch` | Collect results from all sources, verify them and settle bets |
| `bin/cake results_fetch --calendar` | Add missing regular draw dates for the next 12 months |
| `bin/cake results_backfill <from> <to>` | Fetch and cross-check results for past dates (`--dry-run` to only compare) |
| `bin/cake settle [<date>]` | Settle open bets against stored results (`--dry-run` to preview) |
| `bin/cake simulate setup` / `simulate all` | Run the end-to-end simulation suite in the **sandbox database** |
| `bin/cake live_test place\|status\|verify\|pay` | Stress test with marked test accounts (`--accounts=agents`, `--date=YYYY-MM-DD`) |

`simulate` only uses the separate sandbox database (`config/sim_db.php`), never the live data.

---

## Testing

- **Simulation suite:** `bin/cake simulate all`. It checks 330 scenarios:
  - every bet type and company;
  - Box and iBox;
  - settlement, approvals, refunds and result corrections;
  - pool limits, agent earnings, rebates and jackpots;
  - ledger invariants: every wallet equals its credits minus debits, and nothing is paid twice.
- **PHPUnit:** `vendor/bin/phpunit` (tests in `tests/TestCase`). It covers:
  - member, agent and admin pages;
  - security hardening;
  - Excel exports;
  - result sources;
  - currency and language.
- **Stress test:** `live_test` placed more than 30,000 bet records in one day through agents, dealers and end users, then verified every payout.

---

## Currency and languages

- **Currency:**
  - set in **Admin → Settings → Currency and language**;
  - in code, always format amounts with `money($amount)`, with `currency()` for the symbol alone;
  - never write "RM" in code.
- **Languages:**
  - English is the source language;
  - translations live in `src/Locale/zh_CN/default.po` and `src/Locale/ms_MY/default.po`;
  - wrap every visible text in `__('...')`, using `{0}` placeholders for values;
  - after adding texts, run `bin/cake i18n extract --paths src --output src/Locale` and translate the new entries;
  - texts kept in constants are listed in `src/I18n/ExtraStrings.php` so the extract step finds them.
- **Switching:** visitors use the globe menu (`/lang/en`, `/lang/zh`, `/lang/ms`), which remembers the choice in a cookie. The app sends an `X-Lang` header.
- **Dates:** format draw dates with `App\Service\Language::date($date, 'day'|'dayYear'|'long'|'short')`.

---

## Mobile app and API

The Android app lives in its own repository (`betdata_app`); see its README for build steps (Flutter 3.22+).

- Base URL: `https://<your-site>/api/v1`.
- Sign in with `POST /auth/login`. Then send `Authorization: Bearer <token>`.
- Every reply looks like `{"ok": true, "data": …}` or `{"ok": false, "error": "…", "errors": […]}`, plus `currency` and `lang`.
- App bets go through the same server code as the website (`BetSlip`, `BetPlacement`), so the same limits, close times, red numbers and pool apply.

---

## Project structure

```
config/                 settings (app.php is local only), routes, migrations
src/Controller/         web pages (Admin/, Agent/, Member/ prefixes) and Api/
src/Service/            business logic
  BetSlip, BetPlacement   slip building, pricing, placing bets
  Payout, Settlement      prize rates and settling bets
  Pool, StakeLimits       risk control
  Results/                result sources, verification, extra games
  Resettle, DrawVoid      result corrections, cancelled draws
  Agents, AgentEarnings, AgentCredit
  Jackpot, Rebates        promotions
  Money, Language         currency and languages
src/Command/            console commands (results, settlement, simulation, stress test)
src/Simulation/         simulation sandbox and expected-payout oracle
src/Template/           page templates
src/Locale/             translations
webroot/                public files (CSS, JS, images)
tests/                  PHPUnit tests
```

---

## Security notes

- HTTPS only, with security headers (HSTS, frame protection, nosniff, referrer policy) set in `.htaccess`.
- CSRF protection on all signed-in areas.
- Separate access for admins, agents, dealers and end users; anything not allowed is denied.
- Rate limits on sign-in attempts for both the website and the app.
- Real visitor IP addresses are read safely behind Cloudflare.
- Every money movement is recorded and reconciled against the wallet balance; past entries are never edited.
- Secrets (database password, salt, cron key) stay in `config/app.php`, which is not committed. **Never commit credentials or tokens.**
