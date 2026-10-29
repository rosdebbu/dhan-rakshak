# Dhan Rakshak (धन रक्षक) — Project Plan

Last updated: 2026-10-29

> Working name. Repo name: `dhan-rakshak`. Search the Play Store and GitHub for name clashes before any public release.

## What it is

Dhan Rakshak reads your bank's transaction SMS and keeps every credit and debit in one place, whichever UPI app you paid with (GPay, PhonePe, Paytm and others). Set a monthly spending limit and see how much of it you have used. Your data stays on your phone.

## The problem

Students often pay through several UPI apps, so it is hard to see where pocket money actually goes. Most budgeting apps fail because they depend on manual entry. The bank's own transaction SMS is the same no matter which UPI app made the payment, so reading it gives one reliable, automatic source.

## Who it is for

Students first (pocket money, irregular income, first time managing money). Office workers can come later, since salary and EMI budgeting is a different problem.

## Phases at a glance

| Phase | Goal | Timeline |
|---|---|---|
| 1 | Read bank SMS, track all money in and out, monthly limit | 3 weeks |
| 2 | Recognize what money was spent on (categories), notifications | After Phase 1 |
| 3 | Budgets per category, subscriptions, splitting, guidance | Later |

---

## Phase 1

### Delivers

- One timeline of every credit and debit read from bank SMS
- Monthly totals for money in and money out
- A monthly spending limit with a progress bar
- Everything stored on the phone
- A working APK on a few friends' phones at the end of week 3

### Not in Phase 1

Notification reading, categories, machine learning, subscriptions, splitting with friends, charts beyond totals, a server, a Play Store release.

### Build plan

**Week 1: prove it works, then parse**

- [ ] Day 1: a tiny app that lists bank SMS on the Android 14 / 15 test phones
- [ ] Install the APK on a second phone *without USB* to check the sideloaded-install permission flow
- [ ] Collect 30 to 50 sample bank messages (anonymized: no real names or account numbers) into a test file
- [ ] Build the parser with unit tests. Fields to extract:
  - amount
  - money in or out
  - date and time
  - last four digits of the account
  - the other party as written in the message
  - reference number
  - balance, if present

**Week 2: store, show, limit**

- [ ] Room database (amounts stored as whole paise, not decimals)
- [ ] Live receiver for new messages, plus a re-scan of the inbox every time the app opens
- [ ] Timeline screen and monthly totals
- [ ] Monthly limit setting with a progress bar
- [ ] Permission onboarding screen with steps for Android 14 and 15

**Week 3: real-world hardening**

- [ ] Install on friends' phones, collect real messages, fix the parser bank by bank
- [ ] Handle the awkward cases:
  - refunds and reversals
  - failed transactions
  - transfers between your own accounts
  - ATM cash withdrawals
  - credit card messages
  - OTP and promotional filtering
- [ ] CSV export as a simple backup, if time allows
- [ ] README with screenshots, and release the APK on GitHub

---

## Design decisions

- **Source:** SMS only in Phase 1. The bank sends one SMS regardless of which UPI app was used, so SMS alone covers every UPI app.
- **Reading messages:** a receiver for new messages, plus a re-scan of the inbox whenever the app opens. If live catching fails on a phone that kills background apps, the data is still complete once the app is opened.
- **Parsing:** a list of known bank sender IDs, one pattern per bank, and a generic fallback. Messages the parser cannot read go to an "unparsed" list so the parser can be improved.
- **Duplicates:** match on the reference number. When a message has none, match on amount, direction, account and a one-minute time window.
- **Amounts** are stored as whole paise to avoid rounding errors.
- **Raw message text** is kept on the phone so everything can be re-parsed when the parser improves.
- **Privacy:** do not ask for the internet permission at all, and say so in the README. Data stays on the device.

## Data model (draft)

| Field | Notes |
|---|---|
| id | auto-generated |
| amountPaise | integer |
| direction | in or out |
| timestamp | from the message |
| bank | from the sender ID |
| accountLast4 | if present |
| counterparty | as written in the message (name or UPI ID) |
| refNo | unique with bank when present |
| balancePaise | nullable, not every bank includes it |
| rawSms | kept for re-parsing |

## Success criteria

- Parsing accuracy of at least 90% per bank on real messages
- No duplicate transactions
- A month's totals match the bank's own statement for at least one account
- The APK installs and works on friends' phones

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| Too many banks, each worded differently | Support only banks you have real samples for |
| Sideloaded APK blocked from SMS permission | Test the shared-APK install path in week 1; build a clear onboarding screen |
| Phones that kill background apps | Re-scan the inbox every time the app opens |
| Scope creep (notifications, categories) | Keep them out of Phase 1 |
| Spending weeks only learning | Start building on day 1; learn each piece as needed |

---

## Verified vs unverified

**Verified (read in a source)**

- On Android 15, apps installed outside the Play Store are blocked by default from sensitive access, including the SMS runtime permission and notification listener access. Users can still allow them manually. Source: [9to5Google](https://9to5google.com/2024/09/12/android-15-sideloaded-apps-restrictions/). Confirmed for Android 15 only, not newer versions.

**Not verified (treat with care)**

- Google Play's current SMS permission policy: the page would not load its text. My understanding is that SMS permission is limited to default SMS apps and a narrow exception list, and that expense trackers are often rejected. [Check the policy](https://support.google.com/googleplay/android-developer/answer/9047303) before any Play Store plan.
- Account Aggregator (Setu, Finvu): sandbox docs found but not read: [Setu](https://docs.setu.co/data/account-aggregator/v1/get-started), [Finvu](https://finvu.github.io/sandbox).
- Whether any Indian banks now send transaction alerts over RCS (RCS messages are not in the system SMS store, so an app cannot read them with the SMS permission).
- Which banks include the balance in UPI SMS (needs real samples per bank).
- Whether an app installed through USB from Android Studio is treated as sideloaded.
- The current state of Flutter and Go options for SMS and notification access (general knowledge only).
- Name clashes for "Dhan Rakshak".

## Assumptions (change any of these)

1. Kotlin with Jetpack Compose and Room.
2. Banks: SBI, HDFC, ICICI, Axis, Kotak and Canara.
3. Backup in Phase 1 is an optional CSV export.
4. Test phones: Android 14 and Android 15.

## Open questions

1. Which banks do you and your friends use? (decides the parser work)
2. Which languages or frameworks have you built real projects in? (decides how steep Kotlin is)
3. Backup for version 1: re-import from the SMS inbox plus an exported file, backup to the user's own Google account, or your own server?
4. Is the Phase 1 scope right: "where did my money go" plus a monthly limit?

---

## Later phases (ideas)

### Phase 2: what was the money spent on

- Rules first: a list of shop and app names mapped to categories.
- Then learn from the user's one-tap corrections.
- Train a machine learning model only after there are hundreds of labeled transactions, collected from friends who consent to share anonymized data.
- Payments to people (the most common student payment) cannot be categorized automatically. Ask with a one-tap prompt: Food, Rent, Lent, Other.
- Notification listener: show a payment instantly as "pending" until the bank SMS confirms it.
- Labeling idea (feasibility not checked): food-app order notifications arrive close to the debit time, which could label transactions automatically.
- Balance check: when the SMS includes the balance, warn if the balance drop does not match the recorded transactions.

### Phase 3

- Budgets per category and month-end alerts (a daily safe-to-spend amount)
- Subscription and bill detection
- Splitting with friends (needs its own data model and manual entry)
- Student-friendly guidance and weekly summaries
- App lock, dark mode, regional languages

---

## Description text

**One line (GitHub "About"):** Dhan Rakshak is an Android app that automatically tracks your UPI and bank money from SMS, so students can see where every rupee goes, with no manual entry.

**Built with:** Kotlin, Jetpack Compose, Room database.
