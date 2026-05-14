# SereniTrack

A browser-based wellness companion that translates your daily mood into personalised Ayurvedic, yoga, and meditation guidance — powered by an LLM via OpenRouter.

---

## 1. Project Overview

SereniTrack is a mood-aware wellness tool built as a single-page HTML application. The user selects a mood score (1–5) and optionally describes how they are feeling in plain text. That input is sent to a large language model, which returns three structured wellness suggestions: an Ayurvedic tip, a yoga pose, and a meditation cue.

**Problem it solves:**
Most wellness apps are either too generic (static content) or too clinical (symptom trackers). SereniTrack sits in between it uses AI to make every suggestion feel personally relevant, without collecting any health data or requiring an account.

**Positioning:**
SereniTrack is a wellness tool only. It is not a medical device, clinical application, or mental health service. Every response includes an explicit disclaimer to that effect.

---

## 2. Live Demo

```
Demo URL: [https://serenitrack-demo.netlify.app/]
```

Open in Chrome or any modern browser. No account or installation required.

---

## 3. Features

| Feature | Status |
|---|---|
| Mood selector — 1 to 5 emoji scale | ✅ Built |
| Optional free-text emotion label | ✅ Built |
| AI-generated Ayurvedic tip via OpenRouter | ✅ Built |
| AI-generated yoga pose suggestion | ✅ Built |
| AI-generated meditation cue | ✅ Built |
| Free tier — 3 suggestions per day | ✅ Built |
| Daily usage counter with midnight auto-reset | ✅ Built |
| Usage indicator (3-dot progress bar) | ✅ Built |
| "Last suggestion" banner on 3rd use | ✅ Built |
| Upgrade wall with Free vs Pro plan comparison | ✅ Built |
| Consent modal on first visit (GDPR-style) | ✅ Built |
| localStorage consent + usage persistence | ✅ Built |
| Privacy Policy page | ✅ Built |
| Terms of Service page | ✅ Built |
| "Your mood is never stored" privacy pill | ✅ Built |
| Mobile-responsive layout (≤480px) | ✅ Built |
| file:// URL detection with developer warning | ✅ Built |
| Stripe payment flow | 🟡 Mocked |
| Pro tier — unlimited suggestions | 🟡 Mocked |
| Ayurvedic meal plans | 🟡 Mocked |
| Curated yoga videos | 🟡 Mocked |
| Mood history export | 🟡 Mocked |
| User accounts / authentication | 🟡 Not implemented |
| GoCardless direct debit | 🟡 Not implemented |

---

## 4. Tech Stack

### Demo (current)

| Layer | Technology | Reason |
|---|---|---|
| UI | Vanilla HTML, CSS, JavaScript | No build step — runs anywhere, easy to inspect |
| Fonts | Google Fonts (Cormorant Garamond + DM Sans) | Serif/sans pairing suited to a calm wellness aesthetic |
| AI | OpenRouter.ai → `meta-llama/llama-3.3-70b-instruct:free` | Free-tier access |
| State | Browser `localStorage` | No backend dependency; sufficient for a demo |
| Hosting | Static file (Netlify / Live Server) | Zero infrastructure |

### Production (intended architecture)

| Layer | Technology | Reason |
|---|---|---|
| Frontend | Vite + React | Component model, fast builds, tree-shaking |
| API proxy | Vercel serverless functions | Keeps the LLM API key server-side; adds rate limiting and per-user quotas |
| Usage tracking | Supabase (PostgreSQL) | Replaces localStorage; enables cross-device limits and audit logs |
| Payments | Stripe Billing | Subscription lifecycle, webhooks for plan changes and cancellations |
| Analytics | Plausible | Privacy-friendly, no cookies, GDPR-compliant out of the box |
| Auth | Supabase Auth | Email magic-link or OAuth — no password storage |

---

## 5. Privacy & Data Design

SereniTrack is built around data minimisation. The goal is to collect nothing that is not strictly necessary.

**What happens to mood input:**
The text the user types is sent directly to the LLM to generate suggestions. It is not logged, stored in localStorage, written to any database, or retained after the API response is returned. It exists in memory for the duration of one request.

**What is stored (demo):**
- `serenitrack_usage` — a JSON object containing today's date (YYYY-MM-DD) and a usage count (0–3). Resets automatically at midnight via date comparison on read.
- `serenitrack_consent` — a single flag (`"1"`) confirming the user has acknowledged the data notice. No timestamp, no identifier.

**What is never stored:**
Name, email, IP address, location, health data, biometrics, device fingerprint, or any personally identifiable information.

**In production:**
- The anonymous usage record moves from localStorage to Supabase, keyed on a hashed device fingerprint or session token — never on email or name until the user explicitly creates an account.
- GDPR consent is recorded with a timestamp and policy version number.
- Users can request full deletion via a single API call that drops their Supabase row.

**GDPR posture:**
The consent modal on first visit satisfies the requirement for informed, affirmative consent before data processing begins. The Privacy Policy and Terms of Service are accessible from the modal and the site footer before the user takes any action.

---

## 6. Subscription Model

### Free Plan — £0/month
- 3 AI suggestions per day
- Ayurvedic tips, yoga pose guidance, meditation cues
- No account required
- Usage tracked in browser localStorage

### Pro Plan — £9.99/month *(mocked in demo)*
- Unlimited suggestions
- Ayurvedic meal plans
- Curated yoga video library
- Mood history export (CSV)
- Priority AI responses
- Early access to new features

### Team / Coach Plan *(roadmap)*
- Practitioner dashboard
- Client mood summaries
- White-label option for wellness studios

**Stripe integration (production):**
Clicking "Upgrade to Pro" would open a Stripe Checkout session created server-side via a Vercel function. On successful payment, a Stripe webhook updates the user's plan status in Supabase. The frontend reads that status on load to unlock the Pro experience. Cancellations are handled via the Stripe customer portal — no custom cancellation flow needed.

In the demo, clicking "Upgrade to Pro" displays a static confirmation message.

---

## 7. Architecture Decisions

### Why no backend in the demo

The demo prioritises concept clarity and UX over engineering infrastructure. A backend would require deployment, environment variables, auth, and a database — all of which add friction when the goal is to evaluate product thinking and UI quality. Everything that matters for the demo — the AI call, the usage gate, the consent flow, the upgrade wall — works correctly without a server.

The trade-off is that the API key is visible in the source. This is intentional and acceptable for a demo. It is not acceptable in production.

### What changes in production

| Concern | Demo approach | Production approach |
|---|---|---|
| API key exposure | Hardcoded in client JS | Stored in Vercel environment variables; proxied via serverless function |
| Usage enforcement | localStorage (bypassable) | Supabase row with server-side enforcement on the proxy |
| Subscription status | Not real | Stripe webhook → Supabase `plan` field → read on each session |
| Consent record | localStorage flag | Supabase row with timestamp and policy version |
| Rate limiting | None | Vercel edge middleware, per-IP and per-user |
| CORS | Open | Restricted to production domain only |

---

## 8. Running Locally

**Step 1 — Get the files**

Clone the repository or download the ZIP:

```bash
git clone https://github.com/your-username/serenitrack.git
cd serenitrack
```

The project is three files in the same folder: `index.html`, `privacy.html`, `terms.html`.

**Step 2 — Open via a local server**

Do not open `index.html` by double-clicking. Browsers block external API calls from `file://` URLs.

*Option A — VS Code Live Server:*
Right-click `index.html` → Open with Live Server. Visit `http://localhost:5500`.

*Option B — Python:*
```bash
python -m http.server 8080
```
Visit `http://localhost:8080`.

**Step 3 — Set your API key**

Open `index.html` and find this block near the top of the `<script>` section:

```js
const API_KEY  = "sk-or-v1-...";
const ENDPOINT = "https://openrouter.ai/api/v1/chat/completions";
const MODEL    = "meta-llama/llama-3.3-70b-instruct:free";
```

Replace the `API_KEY` value with your own OpenRouter key. Get one free at [openrouter.ai](https://openrouter.ai). The model `meta-llama/llama-3.3-70b-instruct:free` requires no credits. If it returns empty responses, switch to `mistralai/mistral-7b-instruct:free`.

To use the Anthropic API directly instead, the comment block in the script lists the four changes required to switch endpoints and parse the response format.

---

## 9. Interview Notes / Assumptions

- **This is a demo, not production software.** It is built to demonstrate product thinking, UX decisions, and front-end implementation quality — not to be deployed as a live service as-is.

- **The API key is hardcoded intentionally.** In production, the key would live in a server-side environment variable and all AI calls would go through a serverless proxy. The client would never see the key.

- **Stripe and GoCardless are fully mocked.** No real payments are taken. The upgrade button shows a static confirmation message. The billing section in the Terms of Service describes the intended production behaviour.

- **No real health data is collected or stored at any point.** Mood input is transient — it goes to the LLM and is discarded. The only localStorage values are a daily usage count and a consent flag.

- **The free-tier limit is bypassable** by clearing localStorage or using a private window. This is a known and accepted limitation of a client-only demo. In production, enforcement moves server-side via the API proxy.

- **LLM responses are non-deterministic.** On rare occasions the free-tier model returns an empty response or fails to follow the output format. The app handles both cases: empty responses show a descriptive error with a retry prompt; format mismatches fall back to a dash (`—`) in the relevant field.

- **GDPR compliance in this demo is best-effort.** The consent modal, privacy policy, and data minimisation approach reflect production intent. A real deployment would require legal review, a Data Processing Agreement with the LLM provider, and formal registration if operating in the EU.

---

*SereniTrack is a portfolio project. It is not affiliated with any medical institution, Ayurvedic authority, or yoga governing body.*
