# 🎓 School Meeting Portal — v4.5
### A cPanel-native, PHP-only parent-teacher meeting management system for Kenyan schools

**Live demo:** [create.sovereignai.co.ke](https://create.sovereignai.co.ke) &nbsp;|&nbsp; **Stack:** PHP 8 · JSON flat-file DB · MySQL optional · Groq AI · Anthropic API

---

## Overview

School Meeting Portal is a lightweight, zero-dependency web application built for Kenyan secondary schools running on cPanel shared hosting. It gives parents, teachers, principals, and admins a single place to schedule meetings, send apologies, exchange messages, and draft AI-assisted communications — all without touching a terminal or installing a framework.

The system runs entirely from a single `index.php` file and a handful of includes, making it trivial to deploy, upgrade, and maintain on any shared host.

---

## Features

| Module | What it does |
|---|---|
| **Meeting Management** | Principals/admins create school meetings; teachers see meetings for their streams; parents see meetings for their child's class |
| **Booking System** | Parents book time slots for one-on-one teacher meetings; automatic email confirmation sent on booking and cancellation |
| **Apology Workflow** | Parents submit structured apologies for student absences; tracked through approval states |
| **Messaging** | Universal message sender — parents write to teachers/admin, school writes back; all messages stored and emailed |
| **Direct Chat** | Private conversation threads between any two users, with unread counts and inbox view |
| **AI Writing Assistant** | Groq-powered (Llama 3.3 70B) draft assistant for composing messages and notices; Anthropic API proxy also available |
| **Calendar Export** | Export meetings to `.ics` for Google Calendar / Outlook |
| **Audit Log** | Every key action (login, booking, cancel, settings change) recorded with timestamp and IP |
| **Admin Settings** | School name, motto, logo, class lists, departments, and time slots all configurable from the UI |
| **CBC Grade Support** | Built-in Kenya CBC grade and stream helpers; custom streams can be added per school |
| **Role-Based Access** | Four roles — Parent, Teacher, Principal, Admin — each with a scoped dashboard |

---

## Tech Stack

- **Backend:** PHP 8.0+ (single-file architecture, no Composer, no framework)
- **Primary storage:** JSON flat-files (`data/*.json`) with atomic file-lock writes
- **Optional DB:** MySQL / MariaDB (credentials via `.env`, connection managed by `Database.php`)
- **Mailer:** PHPMailer-compatible SMTP via `Mailer.php` — STARTTLS on port 587
- **AI:** Groq API (Llama 3.3 70B) called client-side through `api-proxy.php`; Anthropic API also proxied server-side
- **Hosting:** cPanel shared hosting — no Node, no Composer, no SSH required

---

## Project Structure

```
school-portal-v4/
│
├── index.php              ← Entire application — routing, logic, HTML, CSS, JS
├── api-proxy.php          ← Server-side AI proxy (injects API key, enforces rate limit)
├── .htaccess              ← Security headers, routing, Gzip, browser caching
├── .env                   ← Secrets (API keys, DB password, SMTP password)
├── robots.txt             ← Blocks crawlers from /data/, /admin/, /install/
│
├── config/
│   ├── app.php            ← All constants: feature flags, rate limits, school defaults
│   └── env.php            ← .env file loader (parsed at bootstrap)
│
├── includes/
│   ├── bootstrap.php      ← Loads config + includes, starts hardened session
│   ├── Database.php       ← JSON flat-file DB class (atomic read/write, locking)
│   ├── Mailer.php         ← SMTP email sender with HTML templates
│   ├── helpers.php        ← Utility functions: e(), uid(), jd(), jo(), csrfToken()
│   └── Audit.php          ← Append-only audit trail logger
│
├── data/                  ← JSON "database" — blocked from public by .htaccess
│   ├── meetings.json
│   ├── bookings.json
│   ├── apologies.json
│   ├── messages.json
│   ├── chats.json
│   ├── users.json
│   ├── sessions.json
│   ├── audit_log.json
│   └── settings.json
│
├── logs/                  ← PHP error log — blocked from public
├── cache/                 ← Rate-limit state files
│
└── install/               ← ⚠ Web-based setup wizard — DELETE AFTER FIRST RUN
    ├── index.php
    └── schema.sql
```

---

## Quick Start (cPanel)

### Option A — Automated (Recommended)

1. Upload the entire `school-portal-v4/` folder to `public_html` (or a subdirectory)
2. Open `https://yourdomain.com/install/` in your browser
3. Complete the 3-step setup wizard
4. **Delete the `/install/` folder immediately after**

### Option B — Manual

1. Upload all files to `public_html`
2. Fill in `.env` with your credentials:

```ini
ANTHROPIC_API_KEY=sk-ant-your-key-here
GROQ_API_KEY=gsk_your-groq-key-here
DB_PASSWORD=your-mysql-password
SMTP_PASSWORD=your-smtp-password
APP_SECRET=some-long-random-string-min-32-chars
```

3. Set folder permissions (via cPanel File Manager or FTP):

```
data/   → 755
logs/   → 755
cache/  → 755
```

4. Visit your domain — the default admin is seeded automatically on first run

---

## Configuration (`config/app.php`)

| Constant | Default | Description |
|---|---|---|
| `APP_ENV` | `production` | Set to `development` to display PHP errors |
| `APP_VERSION` | `2.0.0` | Application version string |
| `APP_TIMEZONE` | `Africa/Nairobi` | PHP timezone |
| `DEMO_MODE` | `true` | Accepts any password — disable for production |
| `FEATURE_AI_ASSISTANT` | `true` | Toggle the Groq AI writing helper |
| `FEATURE_CALENDAR_EXPORT` | `true` | Toggle `.ics` calendar export |
| `FEATURE_APOLOGY_WORKFLOW` | `true` | Toggle the apology submission module |
| `FEATURE_AUDIT_LOG` | `true` | Toggle audit trail recording |
| `FEATURE_MAINTENANCE_MODE` | `false` | Lock portal to admins only |
| `RATE_LIMIT_REQUESTS` | `120 / hr` | Per-IP limit for all requests |
| `RATE_LIMIT_AI` | `30 / hr` | Stricter per-IP limit for AI proxy calls |
| `SESSION_LIFETIME` | `28800` | Session expiry in seconds (8 hours) |
| `GROQ_MODEL` | `llama-3.3-70b-versatile` | Groq model to use for AI drafts |

---

## User Roles

| Role | Permissions |
|---|---|
| **Parent** | Book meetings, send apologies, message teachers/admin, view child's meeting schedule |
| **Teacher** | View bookings for their streams, send messages to parents, manage their availability |
| **Principal** | Create school-wide meetings, broadcast notices, view all bookings |
| **Admin** | Full access — all of the above plus user management and school settings |

### Demo Accounts (DEMO_MODE only — any password accepted)

| Role | Email |
|---|---|
| Parent | james.omondi@parent.com |
| Teacher | a.wanjiru@springfield.ac.ke |
| Principal | principal@springfield.ac.ke |
| Admin | admin@springfield.ac.ke |

> **Disable `DEMO_MODE` before going live.** In production, passwords are bcrypt-hashed.

---

## Security

| Control | Implementation |
|---|---|
| Directory protection | `.htaccess` blocks all direct access to `/data/`, `/logs/`, `/cache/`, `/config/` |
| Security headers | `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy` |
| Session hardening | `HttpOnly`, `SameSite=Strict`, session ID regeneration on login |
| Rate limiting | Per-IP, file-lock–based counter; separate tighter limit for the AI proxy |
| Audit trail | Append-only log of all sensitive actions (login, booking, cancel, settings) |
| Atomic writes | `LOCK_EX` on every JSON write prevents data corruption under concurrent load |
| Secrets management | All credentials in `.env`, never hardcoded; loaded once at bootstrap |
| CSRF helpers | `csrfToken()` and `verifyCsrfToken()` available for all state-changing forms |
| AI key protection | Anthropic and Groq API keys are injected server-side — never exposed to the browser |

---

## Email Notifications

The `Mailer.php` class sends HTML emails via SMTP (STARTTLS, port 587) for:

- Booking confirmation to parent
- Booking cancellation receipt to parent + admin notification
- Message delivery (parent → school, school → parent)
- Auto-reply confirmation to parent on message send
- Meeting broadcast to all booked parents

Configure SMTP in `.env`:

```ini
SMTP_PASSWORD=your-email-password
```

And in `config/app.php`:

```php
define('MAIL_SMTP_HOST', 'mail.yourdomain.com');
define('MAIL_SMTP_PORT', 587);
define('MAIL_FROM',      'noreply@yourdomain.com');
```

---

## AI Integration

The portal ships two AI integrations:

**1. Groq AI (primary — free & fast)**
- Model: `llama-3.3-70b-versatile`
- Used for: message drafting, notice writing
- Called via browser → `api-proxy.php` → Groq API (key never leaves the server)
- Rate-limited to 30 calls/hr per IP

**2. Anthropic API (secondary proxy)**
- `api-proxy.php` also proxies requests to the Anthropic messages endpoint
- Key injected server-side; CORS locked to your `APP_URL`

Both integrations respect `FEATURE_AI_ASSISTANT` — set it to `false` to disable the AI panel entirely.

---

## PHP Requirements

| Requirement | Value |
|---|---|
| PHP version | 8.0 or higher |
| Extensions | `curl`, `json`, `session`, `openssl` |
| Writable directories | `data/`, `logs/`, `cache/` |

All of the above are available on every standard cPanel shared hosting plan.

---

## Upgrading

The JSON data format is stable across versions. To upgrade:

1. Back up your `data/` folder
2. Upload the new files over the existing ones (do **not** overwrite `data/` or `.env`)
3. Review `config/app.php` for any new constants and add them to your live config

---

## License

Proprietary — built for Springfield Academy / SovereignAI. Contact the author before redistributing.
