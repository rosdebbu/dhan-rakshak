# Dhan Rakshak (धन रक्षक): Everything in One Place

Last updated: 2026-10-29

This file collects everything decided and discussed so far: the idea, the decisions and why, the technical design, the step-by-step Phase 1 plan, the risks, what is verified and what is not, the git state, and how to continue in Claude Code. It can double as a `CLAUDE.md` handoff for a new session.

---

## 1. Snapshot

| Item | Value |
|---|---|
| Name | Dhan Rakshak (धन रक्षक), "protector of money". Working name; check for clashes before any public release |
| What | Android app that reads bank transaction SMS and tracks all money in and out in one place |
| For | Students first (pocket money, first time managing money). Office workers later |
| Platform | Android only (iOS does not let apps read SMS or other apps' notifications) |
| Current phase | Phase 1, Step 1 (set up and run a starter app) |
| Timeline | 2 to 3 weeks for Phase 1 |
| Week 3 goal | A working app (APK) on a few friends' phones |
| Test phones | Android 14 and Android 15 |
| Repo | https://github.com/rosdebbu/dhan-rakshak |
| Tech (recommended) | Kotlin, Jetpack Compose, Room database |

---

## 2. The idea and the problem

Many people pay through several UPI apps, so it is hard to see where money actually goes, and there is no single history. Budget apps mostly fail because they depend on manual entry. People want money tracking to happen automatically.

The flow: a UPI payment is made. A few minutes later the bank sends an SMS saying the amount was debited (sometimes with the balance). The app reads that SMS and stores the transaction. Because the bank sends one SMS no matter which UPI app was used, SMS is a single reliable source across GPay, PhonePe, Paytm and others.

The goals, in the user's words, summarized: automatic and reliable tracking, one place for all money history, money in and money out (where it goes and where it comes from), a monthly limit, backup of the history, and guidance that helps students manage money. Students first, because they are new to managing money in college.

---

## 3. Phases

| Phase | Goal |
|---|---|
| **1** | Read bank SMS, track all money in and out, monthly totals, monthly spending limit. SMS only |
| **2** | Recognize what money was spent on (categories), notification reading, balance check |
| **3** | Budgets per category, subscriptions, splitting with friends, guidance, app lock |

The user split the work this way on purpose: first read all messages and track where money goes and comes from, then add categories (shop, food, merchandise and so on) step by step.

---

## 4. Decisions log

| Topic | Decision | Why |
|---|---|---|
| Platform | Android only | iOS blocks reading SMS and other apps' notifications |
| Phase 1 data source | SMS only | One SMS covers every UPI app. Notifications are per app, duplicate the same payment, and need a second restricted permission |
| Notifications | Moved to Phase 2 | Used to show a payment instantly as "pending" until the SMS confirms it |
| Tech stack | Kotlin + Jetpack Compose + Room (recommended; used in Step 1) | SMS receiving and notification listening are native Android features. The user has not yet said which languages they know |
| Flutter, React Native | Not chosen | Their main benefit is iOS, which is irrelevant here, and both would still need native Kotlin code for SMS and notifications |
| Go | Not for the app | Android's SMS and notification APIs are Kotlin or Java. Go would need gomobile or glue code. Possible for a future backend |
| Categories | Phase 2: rules first, then learn from user corrections, then ML | ML needs hundreds of labeled examples, which come from friends who consent to share anonymized data |
| Monthly limit | In Phase 1, based on total spending only | Needs no categories, and is very useful for students |
| Splitting with friends | Phase 3 | It is a different product: it needs other people, who owes whom, and manual entry |
| Subscriptions | Phase 3 (stretch only if Phase 1 finishes early) | Small add-on once the engine works, since old SMS give months of history |
| Backup | **Open** | See the open questions |
| Privacy | No internet permission, data on the phone | A strong selling point. Raw SMS text is kept locally for re-parsing |
| Distribution | APK shared through GitHub Releases | Google Play restricts the SMS permission (see section 9) |
| Name | Dhan Rakshak | Chosen from many options (Lekha, Smriti, Hisaab, Aaya-Vyaya, Dhana Store, Dhana Relief, Dhana Mitra and others). "Rakshak" can also suggest security or insurance, so use a tagline such as "know where every rupee goes" |

---

## 5. Technical design

### Pipeline

```
Bank SMS arrives -> receiver catches it -> parser extracts fields -> Room database -> timeline and totals
```

The app never talks to GPay or any UPI app. It reads the SMS stored on the phone.

### Reading messages

- **Old messages:** query the system SMS store with the READ_SMS permission. This works whichever messaging app is used (Google Messages, Samsung Messages and others).
- **New messages:** a receiver for the SMS-received broadcast, with the RECEIVE_SMS permission. Hand the saving to WorkManager so Android does not kill the work.
- **Re-scan on open:** scan the inbox every time the app opens. If live catching fails on a phone that kills background apps, the data is still complete.
- **RCS caveat:** RCS chat messages live in Google Messages' own storage, not the system SMS store, so the SMS permission cannot read them. Bank alerts have traditionally been plain SMS. Whether banks now use RCS for alerts is not verified. The notification listener in Phase 2 would catch those.

### Parser

- Allowlist of bank sender IDs (examples to check against real messages: HDFCBK, SBIINB).
- One pattern per bank plus a generic fallback.
- Filter out OTP and promotional messages.
- Messages the parser cannot read go to an "unparsed" list so it can be improved.
- Fields to extract: amount, money in or out, date and time, account last four digits, the other party as written, reference number, balance if present.

### Duplicates

Match on the reference number. When a message has none, match on amount, direction, account and a one-minute time window.

### Data model (draft)

| Field | Notes |
|---|---|
| id | auto-generated |
| amountPaise | integer, to avoid rounding errors |
| direction | in or out |
| timestamp | from the message |
| bank | from the sender ID |
| accountLast4 | if present |
| counterparty | as written in the message (name or UPI ID) |
| refNo | unique together with bank when present |
| balancePaise | nullable, not every bank includes it |
| rawSms | kept for re-parsing |

### Edge cases to handle

Refunds and reversals, failed transactions, transfers between your own accounts, ATM cash withdrawals, credit card messages, delayed or missing SMS.

---

## 6. Phase 1 step by step

### Step 1: set up and run a starter app (current step)

1. Install the latest stable **Android Studio** from developer.android.com/studio.
2. Turn on **USB debugging** on the phone: Settings, About phone, tap Build number 7 times, then Developer options, USB debugging.
3. Clone the repo and create a work branch: `git clone https://github.com/rosdebbu/dhan-rakshak`, then `git checkout -b phase-1`.
4. In Android Studio: New Project, **Empty Activity** (Compose). Name `DhanRakshak`, package `com.debjit.dhanrakshak`, language Kotlin, minimum SDK API 26, save location inside the cloned `dhan-rakshak` folder.
5. Plug in the phone, accept the debugging prompt, pick the phone at the top, press Run.
6. Commit: `git add .` then `git commit -m "Create starter project"`.

**Done when:** the starter app runs on the phone.

**In parallel (needed for Step 3):** copy 5 to 10 bank transaction messages into a text file (UPI debits, UPI credits, an ATM withdrawal, a refund). Replace names and account numbers with X's but keep the wording and format. Never include OTP messages. Do not commit this file to GitHub until it is fully anonymized.

### Step 2: permission and list the inbox

- Add the SMS permissions to the manifest and a runtime request screen that explains why.
- Query the inbox and list messages from bank sender IDs.
- Test on both the Android 14 and Android 15 phones.
- Install the APK on a second phone *without USB* to check how the sideloaded-install permission flow behaves.

**Done when:** bank messages appear in a list on the phone.

### Step 3: parser with tests

- Put the anonymized sample messages in a test file.
- Build the sender allowlist, per-bank patterns, generic fallback and the unparsed list.
- Write unit tests for each message type.

**Done when:** all sample messages parse correctly in tests.

### Step 4: database

- Room entity, DAO and database with the fields above, amounts in paise.
- Unique index on bank and reference number, plus the fallback duplicate check.
- Keep the raw SMS text.

**Done when:** parsed transactions are saved once, even if the inbox is scanned twice.

### Step 5: live receiver and re-scan

- Broadcast receiver for new SMS, saving through WorkManager.
- Re-scan the inbox every time the app opens.
- Note for users: some phone brands kill background apps, so battery optimization may need to be disabled for the app.

**Done when:** a new bank SMS appears in the app without a manual refresh.

### Step 6: timeline and totals

- Compose list of all transactions, newest first.
- Month selector, with money-in and money-out totals.

**Done when:** totals for a month match the bank's own statement for one account.

### Step 7: monthly limit

- Setting for the monthly limit, stored on the phone.
- Progress bar: this month's money out divided by the limit.

**Done when:** the bar updates as new transactions arrive.

### Step 8: onboarding, real-world testing, release

- Permission onboarding screen with steps for Android 14 and 15 (the restricted-settings step for sideloaded apps).
- Install on friends' phones, collect real messages, fix the parser bank by bank.
- Handle the edge cases listed in section 5.
- Measure parsing accuracy per bank. Target: at least 90%, with no duplicate transactions.
- Optional CSV export as a simple backup.
- README with screenshots, and an APK release on GitHub.

### Success criteria for Phase 1

- Parsing accuracy of at least 90% per bank on real messages
- No duplicate transactions
- A month's totals match the bank statement for at least one account
- The APK installs and works on friends' phones

### Not in Phase 1

Notification reading, categories, machine learning, subscriptions, splitting, charts beyond totals, a server, a Play Store release.

---

## 7. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Too many banks, each worded differently | Support only banks with real message samples |
| Sideloaded APK blocked from SMS permission | Test the shared-APK install path in week 1; build a clear onboarding screen |
| Phones that kill background apps | Re-scan the inbox every time the app opens |
| Scope creep (notifications, categories) | Keep them out of Phase 1 |
| Spending weeks only learning | Start building on day 1; learn each piece as needed |
| "Fully automatic" is not fully possible | Capture and shop categories can be automatic; payments to people need a one-tap prompt in Phase 2 |

---

## 8. Later phases (ideas)

### Phase 2: what was the money spent on

- Rules first: a list of shop and app names mapped to categories, then learn from the user's one-tap corrections.
- Train a model only after hundreds of labeled transactions, collected from friends who consent to share anonymized data.
- Payments to people are the most common student payment and cannot be categorized automatically. Ask with a one-tap prompt (Food, Rent, Lent, Other).
- Notification listener: show a payment instantly as "pending" until the bank SMS confirms it.
- Labeling idea (feasibility not checked): food-app order notifications arrive close to the debit time, which could label transactions automatically.
- Balance check: when the SMS includes the balance, warn if the balance drop does not match the recorded transactions.

### Phase 3

- Budgets per category and month-end alerts (a daily safe-to-spend amount)
- Subscription and bill detection
- Splitting with friends
- Student-friendly guidance and weekly summaries
- App lock, dark mode, regional languages

---

## 9. Verified vs unverified

**Verified (read in a source)**

- On Android 15, apps installed outside the Play Store are blocked by default from sensitive access, including the SMS runtime permission and notification listener access. Users can still allow them manually. Confirmed for Android 15 only. Source: [9to5Google](https://9to5google.com/2024/09/12/android-15-sideloaded-apps-restrictions/).
- Teleport (continuing a cloud session in the terminal) works as described in section 11. Source: [Claude Code docs](https://code.claude.com/docs/en/claude-code-on-the-web.md).

**Not verified (treat with care)**

- Google Play's current SMS permission policy: the page would not load its text. My understanding is that SMS permission is limited to default SMS apps and a narrow exception list, and that expense trackers are often rejected. [Check the policy](https://support.google.com/googleplay/android-developer/answer/9047303) before any Play Store plan.
- Account Aggregator (Setu, Finvu): sandbox docs found but not read: [Setu](https://docs.setu.co/data/account-aggregator/v1/get-started), [Finvu](https://finvu.github.io/sandbox).
- Whether banks send alerts over RCS.
- Which banks include the balance in UPI SMS.
- Whether an app installed through USB from Android Studio counts as sideloaded.
- The current state of Flutter, React Native and Go options for SMS and notification access (general knowledge only).
- Name clashes for "Dhan Rakshak".

---

## 10. Open questions

1. **Which banks** do the user and their friends use? (Decides the parser work.)
2. **Which languages or frameworks** has the user built real projects in? (Decides how steep Kotlin is. Go was mentioned as a possibility.)
3. **Backup for version 1:** re-import from the SMS inbox plus an exported file, backup to the user's own Google account, or a server? A real backup pulls against the no-internet privacy design.
4. **Payments to people:** is a one-tap prompt acceptable? (Phase 2.)

---

## 11. Git and repo state

Repo: https://github.com/rosdebbu/dhan-rakshak

| Branch | On GitHub | Notes |
|---|---|---|
| `main` | Yes | `README.md`, authored as Debjit |
| `planning` | Yes (old version) | One commit with `PLAN.md`, authored as "Claude", not built on `main` |
| `planning` (rebuilt) | Only in the Claude session's working copy | Built on `main`, authored as Debjit. The force-push to replace the remote one was blocked as a destructive git action |

**Still pending**

- The default branch on GitHub is still `planning`. Changing repo settings is not allowed through Claude's connection, so it must be done on GitHub: Settings, Branches, set the default to `main`.
- To replace the old `planning` branch without a force-push: after switching the default to `main`, delete the `planning` branch on GitHub (Branches page, trash icon), then ask Claude to push the rebuilt `planning` as a new branch. Alternative: merge `main` into `planning`, which keeps the "Claude" commit in the history.

**Commit author**

Commits are authored as `Debjit <rosdebbu@users.noreply.github.com>`. The GitHub no-reply address was used instead of the Gmail address so the real email is not public. Whether GitHub links that address to the profile is not verified.

**Suggested branches for building:** keep `main` stable and build on `phase-1`, merging into `main` when it works.

---

## 12. Continue in Claude Code

Teleport works for sessions listed at claude.ai/code. This session's ID appears to be `session_012mXpzTcvJ3RXjNqcWdKjhz`.

1. Install the Claude Code CLI and sign in with the same claude.ai account (`claude auth login`). An API key does not work for this.
2. Go to a checkout of the repo with no uncommitted changes.
3. Run `claude --teleport` (a picker) or `claude --teleport session_012mXpzTcvJ3RXjNqcWdKjhz`. Alternatively, on claude.ai/code use Open in, then Terminal, to copy the command.

The session's branch must be pushed. The terminal gets its own copy of the conversation, so work there does not appear back in the cloud session.

**Alternative:** add a `CLAUDE.md` to the repo (this file can serve as the base) so any new Claude Code session already knows the project.

---

## 13. Description text

**One line (GitHub "About"):** Dhan Rakshak is an Android app that automatically tracks your UPI and bank money from SMS, so students can see where every rupee goes, with no manual entry.

**Short (top of the README):** Dhan Rakshak (धन रक्षक) reads your bank's transaction SMS and keeps every credit and debit in one place, whichever UPI app you paid with (GPay, PhonePe, Paytm and others). Set a monthly spending limit and see how much of it you have used. Your data stays on your phone.

**Built with:** Kotlin, Jetpack Compose, Room database.

---

## 14. Links

- [Android 15 sideloading restrictions (9to5Google)](https://9to5google.com/2024/09/12/android-15-sideloaded-apps-restrictions/)
- [Google Play SMS and call log permissions policy](https://support.google.com/googleplay/android-developer/answer/9047303)
- [Setu Account Aggregator docs](https://docs.setu.co/data/account-aggregator/v1/get-started)
- [Finvu sandbox](https://finvu.github.io/sandbox)
- [Claude Code in the cloud: teleport](https://code.claude.com/docs/en/claude-code-on-the-web.md)
- [Android Studio](https://developer.android.com/studio)
