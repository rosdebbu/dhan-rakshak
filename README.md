# Dhan Rakshak (धन रक्षक)

Automatically track your UPI and bank money from SMS, built for students.

> **Status: planning.** There is no app code yet. The full Phase 1 plan is on the [`planning` branch](https://github.com/rosdebbu/dhan-rakshak/blob/planning/PLAN.md).

## What it will do

Dhan Rakshak reads your bank's transaction SMS and keeps every credit and debit in one place, whichever UPI app you paid with (GPay, PhonePe, Paytm and others). Set a monthly spending limit and see how much of it you have used. Your data stays on your phone.

## Why

Students often pay through several UPI apps, so it is hard to see where pocket money actually goes. Most budgeting apps fail because they depend on manual entry. The bank's own transaction SMS looks the same no matter which UPI app made the payment, so reading it gives one automatic source.

## Roadmap

| Phase | Goal |
|---|---|
| 1 | Read bank SMS, track all money in and out, monthly spending limit |
| 2 | Recognize what money was spent on (categories), notification reading |
| 3 | Budgets per category, subscriptions, splitting with friends, guidance |

## Planned tech (not final)

Kotlin, Jetpack Compose, Room database. Android only, because iOS does not let apps read SMS.

## Privacy goals

- No internet permission
- Data stored on the phone

## Author

Debjit
