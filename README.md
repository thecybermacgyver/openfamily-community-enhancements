# OpenFamily Community Enhancements

A community-maintained set of practical quality-of-life improvements for [OpenFamily](https://github.com/NexaFlowFrance/OpenFamily), focused especially on making the shared family calendar more capable and easier to use day to day.

This is **not an official OpenFamily release**. It is a patch built against OpenFamily commit `801757e` and is intended for people who want the additional functionality described below.

## What this improves

The biggest change is the calendar. The original OpenFamily calendar works well for single appointments, but these enhancements turn it into a much more useful **recurring family scheduler**.

With this patch you can:

- create daily, weekly, monthly, and yearly recurring Calendar Events
- repeat every 2 days, every 3 weeks, every 2 months, or other custom intervals
- set an optional end date for a recurring series
- edit or delete just one occurrence without changing the rest of the series
- edit or delete the entire recurring series when needed
- assign a shared color to each Calendar Event so everyone sees the same organization
- search Calendar Events instead of manually browsing through months
- search by title, description, location, notes, or family-member name
- click a busy day and choose whether to **display its events** or **add another event**
- see all events for a selected day in time order
- use Sunday-first calendars and date pickers

The recurrence logic also handles calendar edge cases correctly. Monthly events on the 29th, 30th, or 31st skip months that do not contain that date instead of silently moving to another day, and February 29 yearly events occur only in leap years.

## Other quality-of-life improvements

The patch also improves several areas outside the calendar:

### Date picker

- custom localized calendar picker instead of relying only on browser-native date fields
- Sunday-first week layout
- previous/next month navigation
- min/max date restrictions
- separate handling for date, time, and datetime values

### Recipes

- recipe cards display their `image_url` when available
- the original chef-hat placeholder remains as a fallback

### Interface

- long Select dropdowns scroll instead of extending beyond the visible screen
- category capacity is increased from 30 to 60

### Notifications

Family ownership-transfer and access-request notifications use the recipient's saved language where supported:

- English
- French
- Chinese

### Localization

User-facing **Appointments** terminology is changed to **Calendar Events** across affected areas of the interface, with corresponding translations in:

- English
- French
- Portuguese
- Chinese

Recurring-event handling is also supported in the Dashboard, Planning, and Kiosk views.

## Original vs customized version

For a full feature-by-feature comparison, implementation notes, affected files, API changes, database changes, and privacy review, see:

**[OpenFamily Community Customization Comparison](OpenFamily_Community_Customization_Comparison.md)**

That document explains exactly how the customized version differs from the OpenFamily baseline.

## Baseline

This patch was built and tested against:

- Upstream repository: [NexaFlowFrance/OpenFamily](https://github.com/NexaFlowFrance/OpenFamily)
- Baseline commit: `801757e`
- Baseline description: `feat(recipes): Bulk recipe ingredients import to Shopping List with multi-language grocery converter (PT/EN/FR) (#81)`

Because OpenFamily may continue to change after that commit, the patch should not be assumed to apply cleanly to newer releases without review or adaptation.

## Installation

Start with an OpenFamily checkout matching commit `801757e`.

From the root of that checkout:

```bash
git apply --check openfamily-community.patch
git apply openfamily-community.patch
```

The first command verifies that the patch can be applied cleanly before making changes.

Always back up your OpenFamily installation and database before applying third-party modifications.

## What is deliberately not included

This community package contains the functional improvements only. It deliberately excludes installation-specific or personal material, including:

- personal images
- custom branding
- favicons and personal PWA icons
- local deployment configuration
- backup files
- runtime data
- credentials
- personal server information

The public patch was generated from an explicit functional-file allow-list and passed an automated privacy/exclusion scan before publication.

## Repository files

- [`openfamily-community.patch`](openfamily-community.patch) — functional source changes
- [`OpenFamily_Community_Customization_Comparison.md`](OpenFamily_Community_Customization_Comparison.md) — detailed original-vs-customized comparison and implementation notes
- [`ATTRIBUTION.md`](ATTRIBUTION.md) — upstream attribution
- [`licence.md`](licence.md) — GNU Affero General Public License v3
- `README.md` — overview, benefits, compatibility, and installation instructions

## License and attribution

OpenFamily is licensed under the GNU Affero General Public License v3. This repository includes the upstream license and attribution information.

See [`ATTRIBUTION.md`](ATTRIBUTION.md) and [`licence.md`](licence.md) for details.
