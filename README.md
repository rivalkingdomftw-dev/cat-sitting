# 🐾 Wendy's Cat Sitting — Booking Site

*Last updated: 24 Oct 2026*

A free booking website where friends pick dates for Wendy & Steven to cat sit. Dates are checked against two Google Calendars, and every request needs our approval by email.

**Cost: $0.** The website is on GitHub Pages. The backend is Google Apps Script, running as Steven's Google account, and it uses Google Calendar, Gmail and a Google Sheet.

## Quick reference

| What | Where |
|---|---|
| Live website | https://rivalkingdomftw-dev.github.io/cat-sitting/ |
| GitHub repo | https://github.com/rivalkingdomftw-dev/cat-sitting |
| Backend (Web App URL) | removed due to safety reason |
| Backend code | Apps Script project "Cat Sitting" at https://script.google.com |
| Booking database | Google Sheet **"Cat Sitting Bookings"** in Google Drive |
| Calendars checked | removed due to safety reason (bookings are written here too) and removed due to safety reason |

Files:
- **index.html**: the website, uploaded to the GitHub repo.
- **Code.gs**: the backend, pasted into Apps Script.
- **README.md**: this record.

```
Friend's browser ──► GitHub Pages (index.html)
                         │  fetch()
                         ▼
               Google Apps Script (Code.gs)
               ├─ 2 × Google Calendar → conflict check + pending/confirmed events
               ├─ Google Sheet        → booking records
               └─ Gmail               → emails to us and to friends
```

---

## How it works

### For friends (the website)
1. **Book tab:** pick a start and end date on the calendar. Busy days are crossed out; past days are greyed out. Then enter the **cat's name** and an **email**, and tap **Request booking**.
2. The backend re-checks **both calendars** and the bookings sheet.
   - If any day clashes, the site says: *"Sorry, we're not available for some of those dates."*
   - If the dates are free, a thank-you pop-up appears: *"We've got your request to look after Mochi from … to … We'll double-check the details and get back to you by email. These dates are held for you in the meantime."*
3. **Manage my booking tab:** a friend enters their email and receives a private link. From the link they can change dates, rename the cat or cancel. Changing dates sends the booking back to *pending* for re-approval. Only one link is emailed per address every 2 minutes.

### Behind the scenes
- **On request:** an all-day event **`⏳ [PENDING] Cat sitting: <cat>`** goes on the booking calendar. This holds the dates so nobody else can book them.
- **We get an email** with the cat, email and dates, plus **✅ Yes, accept** and **❌ No, reject** buttons. Replying to it goes straight to the friend.
  - **Accept:** a confirm page opens. The event is renamed **`🐱 Cat sitting: <cat>`** and the friend gets the confirmation email.
  - **Reject:** a page opens with a box for the reason. The event is deleted, the dates free up, and the friend gets an email that includes the reason.
- The Accept/Reject links contain a secret key, so don't forward those emails.

### Emails friends receive
All are sent from **"Wendy & Steven (cat sitting)"** and signed *🐾 Wendy & Steven's cat sitting*.

| When | Subject | Message (summary) |
|---|---|---|
| Request sent | 🐾 Got your cat sitting request | We've got your request to look after **Mochi** on/from…; we'll double-check; dates held; Manage button |
| **Approved** | ✅ Cat sitting confirmed! | *You're booked! 🎉 We'll look after **Mochi** from **Sat 24 Oct 2026** to **Wed 28 Oct 2026**. Have a lovely trip, and bring us back a souvenir or a treat too 😉🎁* + 📅 **Google Calendar** and **Apple / Outlook (.ics)** buttons + Manage button. The `.ics` file is also attached. |
| Rejected | About your cat sitting request | Unfortunately we can't look after Mochi on/from…; our reason; "Book other dates" button |
| Dates changed | 🐾 Got your updated dates | New dates; we'll confirm again |
| Cancelled | 🐾 Booking cancelled | Booking for Mochi on/from… cancelled |
| Manage link | 🐾 Your cat sitting bookings | List of upcoming bookings with "View / change" buttons |

**Add to calendar (confirmation email only):** the event is an all-day entry called *"🐱 Wendy & Steven looking after Mochi"*, with the Manage link in its notes. It's marked as "free" time, so it won't block the friend's own calendar. There are three ways to add it:
- **Google Calendar button:** opens Google Calendar with the event pre-filled.
- **Apple / Outlook (.ics) button:** downloads the calendar file from our backend. It only works for the booking's own private link, and only while the booking is confirmed.
- **Attached `cat-sitting-<date>.ics`:** Apple Mail and Outlook show "Add to Calendar". In Gmail, tap the attachment to open it.

If a friend later changes or cancels a confirmed booking, the entry already in their calendar does **not** update automatically. They need to delete it themselves. After we re-confirm new dates, the new confirmation email has fresh calendar buttons.

Date wording: a one-day booking reads "on **Sat 24 Oct 2026**". Longer bookings read "from **…** to **…**".

### Design
- Title **Wendy's Cat Sitting**. Subtitle: *"Off on an adventure? Your cat gets snacks, chin scratches and zero judgement while you're away. 🐟"*
- **Soft pastel purple** theme. Main colour `#8e6cc9`, light lavender `#ece4fa`, background `#f7f3fc`, deep plum text `#2e2440`. For a paler purple, change `--accent` in index.html to `#a98fdc`; the buttons get slightly harder to read.
- Works on phones, PCs and Macs in current Chrome, Safari, Edge, Firefox and Samsung Internet. Phones show one month at a time; desktops show two.

---

## Settings (top of Code.gs → `CONFIG`)

| Setting | Current value | What it does |
|---|---|---|
| `OWNER_NAME` | `Wendy & Steven` | Sender name and email sign-off |
| `SITE_URL` | `https://rivalkingdomftw-dev.github.io/cat-sitting/` | Used for "Manage my booking" links in emails |
| `WEB_APP_URL` | the `/exec` URL above | Used for the Accept/Reject buttons |
| `CHECK_CALENDAR_IDS` | the two calendar IDs | A booking must be free on **all** of them. Add `'primary'` to also check the main calendar. |
| `BOOKING_CALENDAR_ID` | `e1f1f698…` (first calendar) | Where pending and confirmed events are written. Must be a calendar we can edit. Always checked. |
| `ONLY_ALL_DAY_EVENTS_BLOCK` | `false` | `false` = any event blocks that day. `true` = only all-day/multi-day events block. |
| `OWNER_EMAIL` | blank | Where Yes/No emails go (blank = the script's Google account) |
| `MAX_DAYS` | `60` | Longest booking allowed |
| `BOOKING_WINDOW_DAYS` | `365` | How far ahead people can book |

The website's `API_URL` (near the bottom of index.html) is set to the same Web App URL.

---

## How to make changes

**Changed Code.gs?**
1. Paste it into Apps Script and save.
2. Go to **Deploy → Manage deployments → ✏️ → Version: New version → Deploy**. The URL stays the same. Without this step, the live site and the email buttons keep running the old code.
3. If you changed calendars, run `setup` once. The log lists the calendars it checks and the one it writes to.

**Changed index.html?**
1. In the GitHub repo, click **Add file → Upload files**, drag in `index.html` (same name, so it replaces the old one), then **Commit changes**.
2. Wait 1–2 minutes, then hard refresh: **Ctrl+Shift+R** on Windows or **Cmd+Shift+R** on Mac.

**GitHub Pages setting** (one-off): Repo → **Settings** tab → **Pages** in the left sidebar → Source *Deploy from a branch* → `main` / `(root)` → Save. The repo must be **public** for free Pages.

---

## First-time setup (for reference / rebuilding)

1. **Apps Script:** go to https://script.google.com → New project → paste Code.gs → set `CONFIG`.
2. In **Project Settings**, set the time zone to match Google Calendar.
3. Run **`setup`** and grant permissions. Google warns "hasn't verified this app": click **Advanced → Go to … (unsafe)**. This is normal for your own script. Setup creates the "Cat Sitting Bookings" sheet and a secret admin key.
4. **Deploy → New deployment → Web app**, with Execute as **Me** and access for **Anyone**. Copy the `/exec` URL into `CONFIG.WEB_APP_URL` and `API_URL` in index.html, then publish a **New version**.
5. **GitHub:** create the public repo `cat-sitting`, upload index.html and enable Pages (see above).
6. Use a **personal** Google account. Work/Workspace accounts often block public web apps.

---

## Troubleshooting

| Problem | Cause / fix |
|---|---|
| Site says "Setup needed" | `API_URL` in index.html is still the placeholder |
| 404 from a "Manage my booking" email link | `SITE_URL` was wrong when the email was sent. Request a fresh link from the Manage tab. |
| "Calendar not found" | The calendar must appear in the left-hand list of Google Calendar for the script's account |
| Accept/Reject page shows dates one day off | The Sheet's time zone (**File → Settings**) doesn't match the Apps Script time zone |
| Changes don't show | Publish a **New version** in Apps Script, and hard refresh the website |
| Friend didn't get an email | Check spam and ask them to mark it "Not spam". Free Gmail allows about 100 script emails/day. |

## Change log

| Date | Change |
|---|---|
| 23 Oct 2026 | Initial build: booking, calendar check, pending hold, Yes/No approval email, rejection reason, manage-by-email link |
| 23 Oct 2026 | Title "Wendy's Cat Sitting"; field renamed to **Cat's name**; notes box removed; emails reworded around the cat's name |
| 23 Oct 2026 | Conflict check across **two calendars**; bookings written to the first |
| 23 Oct 2026 | **Soft pastel purple** theme (site, emails and the Accept/Reject page) |
| 23 Oct 2026 | Web App URL and site URL filled in |
| 23 Oct 2026 | **Fix:** Accept/Reject page error "Please choose valid dates" (Sheets was auto-converting dates); dates now stored as plain text; friendlier error page |
| 23 Oct 2026 | Sign-off **"Wendy & Steven"**; wording switched to "we"; one-day bookings read "on …"; souvenir/treat line added to the confirmation |
| 23 Oct 2026 | Small-phone spacing fix; only busy days are crossed out (past days greyed) |
| 24 Oct 2026 | Confirmation email gets **Add to calendar**: Google Calendar button, Apple/Outlook `.ics` download link, and the `.ics` file attached |
