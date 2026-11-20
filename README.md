# BauAssist – AI Site Manager Assistant (Challenge B prototype)

A single-file web app that takes documentation work off site managers. You capture a situation by voice or text. BauAssist turns it into a structured, formal notice under VOB/B, links it to the schedule and site diary, asks about anything missing, and only exports once the site manager has approved it.

> Drafting support only, not legal advice. The site manager or company reviews and is responsible for everything sent.

---

## Run it (2 minutes)

```bash
cd bauassist
python3 -m http.server 8000
# open http://localhost:8000 in Chrome or Edge
```

Opening `index.html` by double-click also works. However, **dictation (microphone) needs `http://localhost`** or HTTPS, so use the server for the demo. On a phone, open the laptop's address on the same Wi-Fi, or deploy to Netlify/Vercel/GitHub Pages (static file, no build step).

### Two modes
| Mode | How | What you get |
|---|---|---|
| **Demo mode (default)** | Nothing to set up | Rule-based analysis of German input. All 5 example dictations, alerts, plan vs. actual, daily report and minutes summary work **offline**. Safe fallback if Wi-Fi dies during the pitch. |
| **AI mode** | Settings → paste an Anthropic (or OpenAI-compatible) API key | Free-form input, **dictation in Polish/Romanian/Turkish/English → German notice**, AI review of drafts, better summaries. Falls back to offline mode automatically on errors. |

The API key is stored only in the browser's localStorage. That's fine for a demo, but **not** production-safe (see roadmap: move calls to a small backend).

**Before every demo: Settings → "Demo zurücksetzen".** Demo dates are always relative to today.

---

## What's in it (mapped to the challenge)

| Challenge requirement | Where |
|---|---|
| Voice or text capture (mobile) | **Erfassen** tab: Web Speech API, 5 languages, example chips |
| Daily report analysis, plan vs. actual | **Übersicht**: cumulative planned vs. actual quantities from daily reports, days behind per activity (computed in code, not guessed by the AI) |
| Smart drafts from a few keywords | Behinderungsanzeige (§ 6 Abs. 1), Mehrkostenanzeige (§ 2 Abs. 6), Bedenkenanzeige (§ 4 Abs. 3), Mängelrüge (§ 4 Abs. 7 VOB/B) |
| Structured summaries of minutes/emails | **Protokolle & Mails**: open items, owner, deadline (colored by urgency), risks → "draft notice" button |
| Automatic correspondence on deviations | Alerts such as "Behinderung seit 3 Tagen – keine Anzeige" or "Pläne fehlen, Bodenplatte beginnt in 2 Tagen", each with one click to a draft |
| Human in the loop | Every field shows its **origin** (dictation / project data / AI suggestion / manual / missing). Missing required fields become **questions**. **Approval is blocked until they're filled.** Drafts are watermarked "ENTWURF". |
| PDF/Word export | Letters and daily reports → PDF (print dialog) and Word (.doc) |
| Photos | Attach photos to a notice. They're downscaled and listed as attachments. |

### Design decisions worth saying out loud in the pitch
1. **The AI fills fields; it does not write the legal text.** The letter body comes from fixed templates per notice type, so the wording is consistent and reviewable, and the AI can't make up a legal claim.
2. **Numbers come from code, not from the AI.** Plan vs. actual and days behind are deterministic. The AI only gets them as context.
3. **No invented facts.** Unknown → empty → question to the site manager.
4. **Evidence trail.** Each notice links to the site diary entries and photos it's based on, and they appear as *Anlagen* (attachments).
5. **Deadlines.** Notices must be sent without delay (*unverzüglich*), so the app warns when a hindrance is older than 2 days and still has no notice.

---

## Demo script: "A day in the life" (≈5 min)

Run **Demo zurücksetzen** first. Have a phone ready for step 3 if you're using AI mode.

1. **07:30 – Übersicht.** "Before Markus even has coffee, BauAssist has read yesterday's diary and schedule." Show 2 red alerts: hindrance for 3 days without a notice, and missing reinforcement drawings 2 days before the slab starts. Click **Anzeige entwerfen** → a complete delay notice with the Telekom incident as an attachment.
2. **10:15 – Erfassen.** Click the chip *🚜 Bagger steht* (or dictate it live) → **Analysieren**. A delay notice appears in about a second, with the 1.8-day delay **pulled automatically from plan vs. actual**. Point at the origin badges.
3. **10:20 – Polish foreman** (AI mode). Dictate in Polish or use the *🇵🇱* chip → formal German notice. "40% of site workers don't speak German natively."
4. **11:00 – Client orders extra work verbally.** Use the chip *➕ Zusatzleistung* → Mehrkostenanzeige plus the hint "verbal order, get it confirmed in writing". "That hint alone is worth thousands of euros."
5. **11:30 – Human in the loop.** Use the chip *⚠️ Bedenken Beton* → one field is red with a question. **Approval is disabled.** Type the answer → approve → PDF. "Nothing leaves the site without the site manager."
6. **14:00 – Protokolle & Mails.** Load the meeting minutes example → open items with owners and deadlines → risk "additional site road" → **Mehrkostenanzeige entwerfen**.
7. **17:00 – Bautagesbericht.** Pick today → *Bericht anlegen* → dictate "5 Mann, bewölkt 6 Grad, Aushub 85 m³, ab 13 Uhr Regen…" → **Bericht befüllen** → PDF.
8. **Close:** back to Übersicht. Every alert has been handled. "From 2 hours of paperwork to 15 minutes of review."

---

## Code map (`index.html`, all vanilla JS, no build)

| Section | What it does |
|---|---|
| `TYPES` | Notice types: required fields, questions, letter templates. **Add new notice types here.** |
| `makeSeed()` | Demo construction site: project, schedule, 9 daily reports, site diary events |
| `progress()`, `computeAlerts()` | Plan vs. actual and proactive alerts (deterministic) |
| `callLLM()`, `extractAI()` | AI calls (Anthropic or OpenAI-compatible), JSON-only prompts |
| `extractOffline()` | Rule-based fallback so the demo never breaks |
| `letterHTML()`, `reportHTML()`, `exportDoc()` | Letter and report rendering, PDF/Word export |
| `render*()` | One function per tab |

State lives in `localStorage` (`bauassist-v1`). Export it as JSON from Settings.

---

## Team split (5 people, 48 h)

| Person | Owns | First tasks |
|---|---|---|
| **Dev 1 – AI & backend** | Prompts, extraction quality, backend proxy | Test AI mode with 20 realistic dictations and tune `extractAI` prompts. Optional: tiny Node/FastAPI proxy so the key isn't in the browser. |
| **Dev 2 – Frontend & mobile** | UX, phone layout, export quality | Make **Erfassen** feel great on a phone (big mic button, PWA install). Polish the PDF layout with company letterhead. |
| **Dev 3 – Intelligence features** | Alerts, plan vs. actual, photos | Add **photo → daily report text** (vision model). Add a deadline tracker (*Frist läuft ab*). Import a schedule from CSV/Excel. |
| **Domain & data** | Realism | Learn the 4 notice types (1–2 h of reading on VOB/B §§ 2, 4, 6). Write 20 test dictations including messy ones. Check every generated letter: "would a real site manager sign this?" Make the demo data richer. |
| **Pitch** | Story & Q&A | Learn the demo script by heart. Build slides around pain → demo → why we're different. Prepare answers for: *Liability? Data protection? Integration with existing software? Business model?* |

**Rule for Sunday:** feature freeze 4 hours before the deadline. Then only rehearse and fix bugs.

---

## Roadmap ideas to stand out further (pick 2–3)

- **Photo → diary entry.** Take a photo of the excavation, and the AI describes progress and issues (Claude vision).
- **"Hindrance ended" notice** (*Anzeige Wegfall der Behinderung*), plus automatic calculation of the extension time claimed.
- **Deadline radar.** Every notice and open item with a date goes into one timeline, with push reminders.
- **Email inbox triage.** Forward emails to BauAssist, which auto-detects deviations (like the Kraus email) and proposes notices.
- **WhatsApp/Telegram bot** for foremen. Voice message in, diary entry out.
- **Real schedule import** (MS Project XML / Excel) instead of demo data.
- **Backend + login + audit log.** Who approved what, and when. This matters for disputes.

---

## Limits (be honest if judges ask)
- Offline mode understands German keyword patterns only. AI mode is needed for real-world phrasing and other languages.
- No backend: data stays in one browser. The API key is client-side (demo only).
- Templates are drafts based on common VOB/B practice, not reviewed by a lawyer.
