# OpenFamily Community Enhancements

A community-maintained collection of practical quality-of-life improvements for [OpenFamily](https://github.com/NexaFlowFrance/OpenFamily).

This repository now contains three related community packages:

1. the original broad community enhancement patch
2. a follow-up **Budget & Calendar Integration** patch
3. a smaller **UI & AI Refinements** patch

These are **not official OpenFamily releases**.

## Upstream status

OpenFamily has already begun incorporating work from the first community package.

On September 1, 2026, the recurring-appointments, Calendar Event colours, and Calendar search batch was merged upstream through [PR #86](https://github.com/NexaFlowFrance/OpenFamily/pull/86), commit `8717f11`.

That is an important distinction for the follow-up packages: they are published as source/reference packages, not as a claim that the whole patch set should now be applied unchanged to current upstream `main`.

## Package 1 — Original community enhancements

The original patch was built and tested against OpenFamily commit `801757e`.

Its major additions include:

- daily, weekly, monthly, and yearly recurring Calendar Events
- custom recurrence intervals and optional repeat-until dates
- edit/delete one occurrence or the whole recurring series
- shared Calendar Event colours
- Calendar Event search
- improved busy-date Display Events / Add Event behavior
- Sunday-first calendars and custom localized date picker
- recurring-event support in Dashboard, Planning, and Kiosk
- recipe image display
- scrollable long dropdowns
- category capacity increased from 30 to 60
- recipient-language-aware family notifications
- user-facing Appointment terminology changed to Calendar Events in affected translations

For the complete first-package comparison, see:

**[OpenFamily Community Customization Comparison](OpenFamily_Community_Customization_Comparison.md)**

Patch:

**[`openfamily-community.patch`](openfamily-community.patch)**

## Package 2 — Budget & Calendar Integration

The follow-up patch focuses on making the Budget and Calendar operate as a connected transaction/scheduling system.

### Unified Budget entries

The Budget Add/Edit experience is consolidated around one transaction flow.

A transaction can be:

- Income
- Expense

and can be:

- one-time
- daily
- weekly
- monthly
- yearly

Recurring entries support custom intervals and an optional repeat-until date.

This means recurring income is not salary-specific. It can represent any repeating money-in transaction, including investment deposits, repayments, reimbursements, or other recurring deposits.

### Balance carry-forward

The selected month's Current Balance now begins with an **Opening balance** representing all prior Budget activity.

Current Balance then applies:

- current-month one-time Income
- current-month one-time Expenses
- recurring Income occurrences that are due
- recurring Expense occurrences that are due

Future recurring transactions remain in Forecast until their scheduled date arrives.

Reconciliation checkmarks are record-keeping only and do not change the balance.

### Budget ↔ Calendar

Budget and Calendar can now create permanently linked records.

From Budget:

- choose **Add to Calendar**
- a one-time Budget transaction creates an all-day Calendar Event
- a recurring Budget transaction creates a recurring all-day Calendar Event with matching recurrence

From Calendar:

- choose **Add to Budget**
- select Income or Expense
- enter amount and category
- a one-time Calendar Event creates a one-time Budget entry
- a recurring Calendar Event creates a recurring Budget transaction with matching recurrence

Linked date/recurrence changes remain synchronized, and deletion works in both directions.

### All-day Calendar Events

Calendar Events now support an explicit **All day** setting.

Budget-generated Calendar Events use this mode automatically.

### Recurring Income and Expense

The historical recurring-expense table is retained for compatibility, but recurring records now carry an Income/Expense direction.

Existing recurring rows remain Expenses by default.

The recurrence engine supports:

- daily
- weekly
- monthly
- yearly
- every N periods
- optional end date

Month-end and leap-year behavior follows the Calendar recurrence rules.

### Analytics

Budget analytics now distinguish recurring Income from recurring Expenses.

Monthly charts count actual recurring occurrences, and recurring Expenses contribute to expense/category totals and monthly limits without treating recurring Income as spending.

### Budget interface cleanup

The Budget page now uses:

- **Classic**
- **Analytics & Limits**

The old Kakeibo mode switch is removed from the Budget interface.

The working customization represented here updates English, French, Portuguese, and Chinese. OpenFamily added Russian after this customization branch was established, so an upstream rebase should add equivalent Russian strings for the new Budget/Calendar keys.

For the full implementation comparison, database changes, affected files, privacy review, and compatibility notes, see:

**[OpenFamily Budget & Calendar Integration Comparison](OpenFamily_Budget_Calendar_Comparison.md)**

Patch:

**[`openfamily-budget-calendar.patch`](openfamily-budget-calendar.patch)**

## Package 3 — UI & AI Refinements

The third package is intentionally smaller. It collects several usability fixes that were tested in the running customized installation and adds Google Gemini to OpenFamily's existing AI-provider system.

### Budget usability

- Forecast remains available but is **hidden by default** behind an eye toggle so Current Balance stays visually primary
- recurring Budget entries use a responsive mobile card/grid layout so labels, dates, recurrence details, amounts, and edit/delete controls remain readable on phones
- desktop recurring-entry layout is preserved

### Calendar and Planning layout

- the seven-column Calendar keeps a practical minimum width on phones and becomes horizontally scrollable instead of compressing day cells until they are unreadable
- Planning keeps the same clear single-column day-card presentation on desktop that already worked well on mobile

### Google Gemini AI provider

Gemini is added as a first-class option alongside Ollama, OpenAI-compatible APIs, and Anthropic.

The implementation includes:

- Google Gemini provider selection in Settings
- Gemini API-key handling
- direct `generateContent` integration
- structured JSON output using Gemini response schema support
- existing OpenFamily AI request/response validation and token-usage integration
- database provider-constraint migration for existing installations
- fresh-install schema support
- clearer server-side AI provider error logging
- Gemini provider labels in English, French, Portuguese, and Chinese

For the complete implementation and verification notes, see:

**[OpenFamily UI & AI Refinements Comparison](OpenFamily_UI_AI_Refinements_Comparison.md)**

Patch:

**[`openfamily-ui-ai-refinements.patch`](openfamily-ui-ai-refinements.patch)**

Manifest:

**[`openfamily-ui-ai-refinements-files.txt`](openfamily-ui-ai-refinements-files.txt)**

## Baselines and compatibility

### Original patch

- Upstream repository: [NexaFlowFrance/OpenFamily](https://github.com/NexaFlowFrance/OpenFamily)
- Original baseline commit: `801757e`
- Baseline description: `feat(recipes): Bulk recipe ingredients import to Shopping List with multi-language grocery converter (PT/EN/FR) (#81)`

### Follow-up patches

`openfamily-budget-calendar.patch` and `openfamily-ui-ai-refinements.patch` are **incremental to the working community customization** rather than patches generated directly from current upstream `main`.

Because upstream has since incorporated selected parts of the first package, both follow-up packages should be treated as review/rebase references for upstream development rather than assumed to apply directly to current OpenFamily.

Package 3 specifically assumes the customized Budget and Calendar structures produced by the earlier work where its UI refinements touch those pages.

## Installation from the original baseline

For a historical checkout matching `801757e`, review and apply the packages in order.

From the OpenFamily repository root:

```bash
git apply --check openfamily-community.patch
git apply openfamily-community.patch

git apply --check openfamily-budget-calendar.patch
git apply openfamily-budget-calendar.patch

git apply --check openfamily-ui-ai-refinements.patch
git apply openfamily-ui-ai-refinements.patch
```

Always back up your OpenFamily installation and database before applying third-party modifications.

For current upstream OpenFamily, review/rebase the relevant changes instead of blindly applying these historical/incremental patches.

## What is deliberately not included

The public packages contain functional improvements only.

They deliberately exclude installation-specific, personal, or unfinished material, including:

- personal images
- custom branding
- favicons and personal PWA icons
- local deployment configuration
- backup files and backup directories
- runtime data
- credentials and API keys
- personal server information
- unfinished Family Posts experiments

The Package 3 Gemini integration contains provider code only; it does not contain the contributor's Gemini API key.

## Repository files

- [`openfamily-community.patch`](openfamily-community.patch) — original community functional source changes
- [`OpenFamily_Community_Customization_Comparison.md`](OpenFamily_Community_Customization_Comparison.md) — detailed comparison for the original package
- [`openfamily-budget-calendar.patch`](openfamily-budget-calendar.patch) — follow-up Budget/Calendar integration source changes
- [`OpenFamily_Budget_Calendar_Comparison.md`](OpenFamily_Budget_Calendar_Comparison.md) — detailed comparison and implementation notes for Package 2
- [`openfamily-ui-ai-refinements.patch`](openfamily-ui-ai-refinements.patch) — Package 3 UI and Gemini AI refinements
- [`openfamily-ui-ai-refinements-files.txt`](openfamily-ui-ai-refinements-files.txt) — Package 3 verified file manifest
- [`OpenFamily_UI_AI_Refinements_Comparison.md`](OpenFamily_UI_AI_Refinements_Comparison.md) — detailed comparison, implementation, verification, and compatibility notes for Package 3
- [`ATTRIBUTION.md`](ATTRIBUTION.md) — upstream attribution
- [`licence.md`](licence.md) — GNU Affero General Public License v3
- `README.md` — package overview, benefits, compatibility, and installation instructions

## License and attribution

OpenFamily is licensed under the GNU Affero General Public License v3.

This repository includes the upstream license and attribution information.

See [`ATTRIBUTION.md`](ATTRIBUTION.md) and [`licence.md`](licence.md) for details.
