# OpenFamily — Budget & Calendar Integration Comparison

## Baseline and package relationship

This follow-up community package documents the Budget and Calendar integration work developed after the original OpenFamily community enhancement set.

- Upstream repository: `NexaFlowFrance/OpenFamily`
- Original community repository: `thecybermacgyver/openfamily-community-enhancements`
- Original upstream baseline: `801757e`
- Follow-up patch: `openfamily-budget-calendar.patch`
- Follow-up patch relationship: **incremental to the original community customization package**

The first community package was intentionally published as a reviewable reference rather than as a claim of being merge-ready against future OpenFamily versions. The same rule applies here.

As of September 1, 2026, OpenFamily has already integrated the first recurring-calendar batch — recurring appointments, colours, and search — through upstream PR `#86` / commit `8717f11`. This follow-up package therefore focuses on the newer Budget ↔ Calendar integration and Budget transaction model. Calendar recurrence code may still appear in the patch where the newer integration depends on or restructures it.

This follow-up patch was generated from the contributor's working customized installation after the original community patch baseline. It should be treated as a reference package for review or rebasing, not assumed to apply directly to current upstream `main`.

## Previous behavior vs follow-up customization

| Area | Previous / original behavior | Follow-up customization |
|---|---|---|
| Budget Add workflow | Separate entry paths and a separate recurring-debit workflow | One general **Add entry** flow |
| Entry type | One-time entries can be Income or Expense; recurrence is debit/expense-oriented | Both one-time and recurring entries can be **Income** or **Expense** |
| Recurring income | Not supported by the recurring-debit model | Generic recurring income, suitable for pay, investment deposits, repayments, reimbursements, or any repeating money-in entry |
| Recurrence frequency | Recurring Budget items are historically monthly debit-oriented | Daily, weekly, monthly, or yearly recurrence |
| Recurrence interval | Fixed monthly behavior | Custom interval such as every 2 weeks or every 6 months |
| Recurrence end | No general end-date model | Optional **Repeat until** date |
| Month-end recurrence | Monthly debit-day behavior | Invalid dates are skipped rather than silently shifted, matching Calendar recurrence behavior |
| Leap-year recurrence | Not part of the recurring Budget model | February 29 yearly occurrences appear only in leap years |
| Budget edit UI | Separate forms/state for one-time and recurring items | Unified Add/Edit model with shared type, amount, category, date, recurrence, and Calendar-link controls |
| Budget modes | Classic / Kakeibo switch | Classic / **Analytics & Limits** |
| Current Balance | Calculated from the selected month, so a new month can begin at zero | Carries the complete prior Budget balance forward as an **Opening balance** |
| Recurring transaction timing | Pointing/reconciliation historically controls whether recurring debits affect balance | A recurring occurrence affects Current Balance when its scheduled date becomes due |
| Reconciliation | Debit-specific “pointing” terminology | Generic reconciliation marker; reconciliation does **not** change Current Balance |
| Forecast | Primarily future recurring debits | Includes upcoming recurring **income and expenses** |
| Statistics | Recurring items treated as expenses/debits | Recurring income and expenses are separated correctly in totals and monthly analytics |
| Monthly limits | Primarily one-time expense totals | Recurring expense occurrences contribute to expense/category totals |
| Calendar all-day events | Calendar data is time-oriented | Calendar Events can explicitly be **All day** |
| Budget → Calendar | No permanent linked transaction/event relationship | A Budget entry can create a linked all-day Calendar Event |
| Recurring Budget → Calendar | No recurrence-aware link | Calendar recurrence mirrors the Budget recurrence rule |
| Calendar → Budget | No Budget creation from a Calendar Event | Calendar Event can create a linked Budget Income or Expense |
| Calendar recurring → Budget | No recurrence-aware conversion | Recurring Calendar Event creates a recurring Budget transaction with matching recurrence |
| Link synchronization | Independent records | Linked date/recurrence changes remain synchronized at series level |
| One-time ↔ recurring conversion from Calendar | Independent records | Changing a linked Calendar Event between non-recurring and recurring converts the linked Budget record between one-time and recurring storage |
| Linked deletion | Deleting one side leaves the other | Deleting the linked Budget item or Calendar Event deletes its counterpart |
| Link uniqueness | No link columns | Unique partial indexes prevent multiple Calendar Events from claiming the same linked Budget record |
| Generated event styling | No Budget-generated Calendar Event | Budget-generated Income defaults to green and Expense defaults to red |
| Localization | Debit-oriented recurring wording | Generic recurring-entry / reconciliation / linking text across English, French, Portuguese, and Chinese |

## Unified Budget transaction model

### One-time entries

One-time Budget records continue to use `budget_entries`.

The existing field:

- `budget_entries.is_expense`

already distinguishes money out from money in, so the follow-up UI uses that same field in the unified form rather than creating a second income model.

### Recurring entries

Recurring records remain in the existing `recurring_expenses` table for backward compatibility. The internal table name is deliberately not renamed, because renaming it would create unnecessary migration and integration risk.

Migration 027 adds:

- `recurring_expenses.is_expense BOOLEAN NOT NULL DEFAULT true`

The default preserves every existing recurring row as an Expense. New recurring records may now use either:

- `is_expense = true` — Expense
- `is_expense = false` — Income

This means recurring income is generic. It is not salary-specific and can represent any repeating money-in transaction.

## Recurring Budget implementation

### Migration 025

The recurring Budget model is expanded from monthly debit-day behavior to occurrence-based recurrence.

It adds:

- `recurring_expenses.start_date`
- `recurring_expenses.recurrence_frequency`
- `recurring_expenses.recurrence_interval`
- `recurring_expenses.recurrence_until`
- `recurring_expense_logs.occurrence_date`

Existing recurring rows are preserved as monthly schedules.

The former monthly-only reconciliation uniqueness rule is replaced with an occurrence-level unique index:

- `(recurring_expense_id, occurrence_date)`

### Supported frequencies

- daily
- weekly
- monthly
- yearly

Intervals are normalized to integer values from 1 through 365.

Examples:

- every 2 days
- every 2 weeks
- every 3 months
- every 2 years

### Occurrence expansion

Recurring Budget records are expanded into occurrences for the requested date range.

Generated occurrences include:

- `series_id`
- `occurrence_id`
- `occurrence_date`

The expansion estimates a useful starting occurrence near the requested date range and includes a hard safety iteration limit.

### Calendar-date edge cases

The recurring Budget engine deliberately follows the same behavior as recurring Calendar Events.

For monthly recurrence:

- if the original day does not exist in a target month, that occurrence is skipped rather than moved

For yearly recurrence:

- February 29 occurs only in leap years

## Balance carry-forward

The `/api/budget/forecast` calculation now has an explicit opening balance.

### Opening balance

For the selected month, the server calculates all activity before the first day of that month:

- prior one-time income
- prior one-time expenses
- historical recurring income occurrences
- historical recurring expense occurrences

Conceptually:

`Opening Balance = prior income - prior expenses + prior recurring income - prior recurring expenses`

### Current Balance

For the selected month:

`Current Balance = Opening Balance + current one-time income - current one-time expenses + due recurring income - due recurring expenses`

Recurring occurrences are considered due according to the selected month and the supplied local `as_of` date:

- future month: no recurring occurrence is due yet
- past month: every scheduled occurrence in that month is due
- current month: occurrences through today are due

### Forecast Balance

Forecast extends Current Balance with the remaining scheduled occurrences:

`Forecast = Current Balance + upcoming recurring income - upcoming recurring expenses`

### Reconciliation

The recurring transaction checkmark is now reconciliation only.

Marking an occurrence reconciled or unreconciled does not add or remove money from Current Balance. The scheduled due date controls balance timing.

## Calendar all-day support

Migration 026 adds:

- `appointments.is_all_day BOOLEAN NOT NULL DEFAULT false`

The Calendar interface adds an **All day** option.

Budget-generated Calendar Events are created as all-day events. Their date is the transaction date, while the server uses start/end-of-day timestamps for compatibility with the existing appointment storage.

## Budget → Calendar linking

A Budget entry can be created or edited with **Add to Calendar**.

### One-time transaction

The server creates or synchronizes one all-day Calendar Event linked through:

- `appointments.linked_budget_entry_id`

The Calendar title is derived from the Budget description, falling back to the Budget category.

### Recurring transaction

A recurring Budget transaction creates or synchronizes an all-day recurring Calendar Event linked through:

- `appointments.linked_recurring_expense_id`

The Calendar recurrence mirrors:

- start date
- frequency
- interval
- optional recurrence end date

Generated Calendar Events use a default visual distinction:

- Income: green
- Expense: red

## Calendar → Budget linking

The Calendar form adds **Add to Budget**.

When selected, the user chooses:

- Income or Expense
- amount
- Budget category

The Calendar Event's date and recurrence determine the Budget transaction schedule.

The integration endpoint is:

- `POST /api/budget/from-appointment/:appointmentId`

### Non-recurring Calendar Event

Creates or synchronizes a one-time `budget_entries` record.

### Recurring Calendar Event

Creates or synchronizes a `recurring_expenses` record using the same:

- frequency
- interval
- recurrence end date

### Recurrence conversion

If a linked Calendar Event changes:

- from recurring to non-recurring, the recurring Budget record is replaced with a one-time entry
- from non-recurring to recurring, the one-time Budget entry is replaced with a recurring record

The existing Budget amount/category/type are preserved when the Calendar is synchronizing an already-linked record unless explicitly supplied.

## Bidirectional synchronization and deletion

Migration 026 adds:

- `appointments.linked_budget_entry_id`
- `appointments.linked_recurring_expense_id`

It also adds unique partial indexes for both link types.

This keeps each Budget item paired with at most one Calendar Event.

### Synchronization

For linked records, series-level changes keep the transaction/event relationship aligned, including date and recurrence changes.

### Deletion

Deletion works in both directions:

- deleting a linked Budget entry deletes its Calendar Event
- deleting a linked recurring Budget series deletes its Calendar Event
- deleting a linked Calendar Event deletes the linked one-time Budget entry
- deleting a linked Calendar Event deletes the linked recurring Budget series

This prevents orphaned linked records.

## Budget analytics and limits

The recurring transaction type is used throughout Budget calculations.

### Monthly statistics

Recurring occurrences are separated into:

- recurring income
- recurring expenses

Monthly analytics therefore represent the number of actual occurrences in each month rather than assuming every recurring series contributes exactly once.

### Expense categories and limits

Only recurring records with:

- `is_expense = true`

are added to recurring expense/category totals and Kakeibo expense calculations.

This prevents recurring income from being misclassified as spending.

## Budget interface cleanup

The Budget page is simplified around two modes:

- **Classic**
- **Analytics & Limits**

The Kakeibo mode switch is removed from the Budget page.

The Add/Edit experience is consolidated so the same transaction form handles:

- Income / Expense
- amount
- description
- category
- date
- recurrence
- repeat interval
- optional repeat-until
- Add to Calendar

Separate recurring-debit form state and duplicated entry paths are removed.

## Calendar interface additions

The Calendar page gains:

- All day
- Add to Budget
- Budget Income / Expense selector
- Budget amount
- Budget category
- linked-Budget status text

The existing recurrence controls are reused so Calendar-to-Budget recurrence uses the same visible schedule the user has already configured for the Calendar Event.

## Localization

The follow-up package updates affected Budget and Calendar strings in:

- English
- French
- Portuguese
- Chinese

Wording is changed away from debit-only language where the feature now applies equally to Income and Expense.

The customized installation represented by this patch predates OpenFamily's later Russian translation. Therefore this follow-up patch does **not** contain equivalent Russian keys. Any upstream rebase onto a version that includes Russian should add matching Russian Budget/Calendar strings rather than dropping that language.

Examples include:

- recurring entries
- reconciliation
- Add entry
- Opening balance
- Add to Calendar
- Add to Budget
- All day
- Budget link status

## Functional files changed

The follow-up patch contains 14 functional source/translation files.

### Client

- `client/src/pages/Budget.tsx`
- `client/src/pages/Calendar.tsx`

### Server

- `server/src/db.ts`
- `server/src/routes/appointments.ts`
- `server/src/routes/budget.ts`
- `server/src/routes/kakeibo.ts`

### Translations

- `client/src/i18n/locales/en/budget.json`
- `client/src/i18n/locales/en/calendar.json`
- `client/src/i18n/locales/fr/budget.json`
- `client/src/i18n/locales/fr/calendar.json`
- `client/src/i18n/locales/pt/budget.json`
- `client/src/i18n/locales/pt/calendar.json`
- `client/src/i18n/locales/zh/budget.json`
- `client/src/i18n/locales/zh/calendar.json`

## Database migrations represented in this source set

The current customized `server/src/db.ts` contains the following relevant migration stages:

- Migration 023 — recurring Calendar Events
- Migration 024 — shared Calendar Event color
- Migration 025 — occurrence-based recurring Budget transactions
- Migration 026 — all-day Calendar Events and Calendar/Budget links
- Migration 027 — recurring Budget Income / Expense direction

The first two relate to the earlier Calendar contribution and may already exist upstream in a different form after PR #86. Migrations 025–027 are the main new Budget/Calendar integration work documented here.

## Explicitly excluded from the public package

As with the first community package, the follow-up patch deliberately excludes installation-specific and personal material.

Excluded categories include:

- personal images
- custom branding
- favicon/PWA branding files
- local deployment configuration
- backup files
- runtime data
- credentials
- personal server information

## Privacy and exclusion review

The generated package audit reported:

- no known excluded branding/config/backup paths
- no obvious local paths, private IPs, or credential assignments

A second source scan of added patch lines also found no obvious:

- personal names from the contributor's household
- email addresses
- IPv4 addresses
- local `/home`, `/Users`, or `/opt` paths
- obvious password/API-key/access-token/secret assignments

This is a packaging safety check, not a formal security audit.

## Validation performed

The source represented by this patch was rebuilt and started in the contributor's self-hosted OpenFamily Docker installation after application of the changes.

The reported service state after rebuild was:

- built
- healthy
- started

The follow-up package should still be reviewed and tested against whatever upstream commit it is rebased onto before merge.

## Suggested community release description

> Follow-up OpenFamily community enhancements focused on making Budget transactions and Calendar Events work as one connected system. The package adds a unified Income/Expense entry flow, generic recurring income and expenses, daily/weekly/monthly/yearly Budget recurrence with custom intervals, balance carry-forward, due-date-based Current Balance, income-aware forecasting and analytics, all-day Calendar Events, and bidirectional Budget ↔ Calendar linking with synchronized recurrence and deletion. Existing recurring Budget rows remain expenses by default for backward compatibility. The package excludes personal branding, deployment configuration, backups, credentials, runtime data, and server-specific information.

## Upstream review note

This repository is intended to make the work easy to inspect.

The original package was built against `801757e`, and upstream has since accepted part of that package. The follow-up patch was generated from the customized package baseline rather than current upstream `main`.

For upstream integration, the safest approach is to review/rebase the Budget/Calendar work as its own batch instead of assuming this patch is directly merge-ready.
