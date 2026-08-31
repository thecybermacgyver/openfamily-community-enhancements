# OpenFamily — Community Customization Comparison

## Baseline

This customization set was compared against the public OpenFamily repository:

- Repository: `NexaFlowFrance/OpenFamily`
- Baseline commit: `801757e`
- Baseline commit message: `feat(recipes): Bulk recipe ingredients import to Shopping List with multi-language grocery converter (PT/EN/FR) (#81)`

This document describes only application-level changes that remain present in the customized installation when compared with that baseline.

## Original vs Customized

| Area | Original OpenFamily | Customized Version |
|---|---|---|
| Calendar terminology | Uses “Appointments” throughout the interface | Uses “Calendar Events” throughout supported UI translations |
| Calendar recurrence | Calendar entries are single events | Supports daily, weekly, monthly, and yearly recurring Calendar Events |
| Recurrence interval | Not available | Supports intervals such as every 2 days, every 3 weeks, every 2 months, etc. |
| Recurrence end date | Not available | Optional “Repeat until” date |
| Individual recurring occurrence edits | Not available | Edit only one occurrence without altering the rest of the series |
| Individual recurring occurrence deletion | Not available | Delete only one occurrence while keeping the rest of the series |
| Whole-series editing | Not applicable | “All occurrences” option preserves correct series timing even when editing from a later occurrence |
| Monthly edge cases | Not applicable | Events on the 29th, 30th, or 31st skip months that do not contain that date |
| Leap-year handling | Not applicable | Feb. 29 yearly events occur only in leap years |
| Calendar Event colors | Display color derived from a family member / default styling | Each Calendar Event has a shared stored color visible consistently to users |
| Color selection | No event color picker | Fixed 16-color palette |
| Calendar Event search | No dedicated all-event search | Search Calendar Events by title, description, location, notes, and assigned family-member names |
| Multiple events on one date | Month cell shows up to three events and “more”; clicking the date opens event creation | Clicking a populated date offers **Display events** or **Add event**; Display shows all events for that day |
| Empty date click | Opens event creation | Preserved: empty dates open event creation directly |
| Month week layout | Monday-first | Sunday-first |
| Date input | Browser-native date fields | Custom localized calendar date picker |
| Date-picker week layout | Browser-dependent | Sunday-first |
| Dashboard recurring events | Assumes event ID is unique | Uses recurring `occurrence_id` when available |
| Planning recurring events | Assumes event ID is unique | Uses recurring `occurrence_id` when available |
| Kiosk recurring events | Assumes event ID is unique | Uses recurring `occurrence_id` when available |
| Recipe cards | Chef-hat placeholder | Displays `image_url` when supplied, with the chef-hat retained as fallback |
| Long select menus | Can extend beyond available screen space | Maximum-height dropdown with vertical scrolling |
| Category limit | 30 categories | 60 categories |
| Family notifications | Ownership/join-request notifications are hard-coded in French | Notification text follows the recipient account’s saved language for English, French, and Chinese |
| UI terminology localization | Appointment terminology | Calendar Event terminology updated across English, French, Portuguese, and Chinese |

## Recurring Calendar Event Implementation

### Database changes

A recurrence migration adds:

- `appointments.recurrence_frequency`
- `appointments.recurrence_interval`
- `appointments.recurrence_until`
- `appointment_recurrence_exceptions` table
- index for recurrence exceptions

A separate migration adds:

- `appointments.color`

The recurrence-exceptions table supports both:

- `skip` — omit a single occurrence
- `override` — change a single occurrence while leaving the series intact

### Recurrence frequencies

Supported values:

- none
- daily
- weekly
- monthly
- yearly

Intervals are normalized to integer values from 1 through 365.

### Recurrence expansion

Recurring series are expanded into occurrences for the requested calendar date range.

Each generated occurrence receives recurrence metadata including:

- `series_id`
- `occurrence_id`
- `occurrence_date`
- `is_recurring_occurrence`
- original series start/end values

The implementation estimates the starting occurrence near the requested range rather than always iterating from the original series start.

### Special calendar-date behavior

Monthly recurrence deliberately skips a month when the requested calendar day does not exist in that month.

Example:

- A monthly event on January 31 does not silently move to February 28/29.

Yearly recurrence similarly validates the date:

- A February 29 event appears only in leap years.

### Per-occurrence API

Two routes were added for individual occurrences:

- `PUT /api/appointments/:id/occurrences/:date`
- `DELETE /api/appointments/:id/occurrences/:date`

These create an override or skip exception instead of rewriting the original recurring series.

## Calendar Interface Improvements

### Shared Calendar Event colors

Calendar Events now store a `color` value on the appointment itself rather than deriving display color from the first assigned family member.

The server validates colors as six-digit hexadecimal values and falls back to the default event color when necessary.

The selected color is used in:

- month-grid entries
- upcoming-event cards
- search results
- daily event display

### Fixed color palette

The free-form color input was replaced with a fixed 16-color palette:

- red
- orange
- yellow
- lime
- green
- dark green
- aqua
- cyan
- blue
- navy
- purple
- violet
- pink
- magenta
- brown
- black

### Calendar Event search

A search panel loads stored Calendar Events separately from the visible month range and searches:

- title
- description
- location
- notes
- assigned family-member names

Matching entries can be opened directly for editing.

### Display / Add behavior for populated dates

When a date contains one or more Calendar Events, clicking the date now opens a choice:

- **Display events**
- **Add event**

Display mode shows all events for that date, ordered by start time. Selecting an event opens it for editing.

Dates with no existing events still open the new-event form directly.

### Sunday-first calendar

The month grid and weekday headings were changed from Monday-first to Sunday-first.

## Date Picker Improvements

The browser-native date input was replaced with a reusable custom calendar picker.

The custom picker includes:

- localized month/day formatting
- Sunday-first week layout
- previous/next month navigation
- currently selected date indication
- min/max date restrictions
- click-outside closing behavior
- separate support for date, time, and datetime inputs

## Recipe Improvements

Recipe cards now render the recipe’s `image_url` when available.

If the recipe has no image URL, the original chef-hat placeholder remains visible.

## Select Dropdown Improvement

Long selection menus now use an available-height-aware maximum height and vertical scrolling instead of allowing options to extend beyond the visible screen.

## Category Capacity

The server category limit was increased:

- Original: 30
- Customized: 60

## Localized Family Notifications

Family ownership-transfer and join-request notifications now use the recipient’s saved language.

Localized notification text was added for:

- English
- French
- Chinese

Covered notification types include:

- ownership transferred
- new family access request
- request accepted
- request declined

If a saved language is missing or unsupported by this notification helper, the existing French behavior is retained as fallback.

## Terminology Changes

User-facing “Appointment” terminology was changed to “Calendar Event” equivalents throughout affected interface translations.

Updated areas include:

- Calendar
- Dashboard
- Navigation
- Planning
- Settings
- Integrations
- Kiosk where applicable

Languages affected:

- English
- French
- Portuguese
- Chinese

Internal API paths, database tables, and internal code identifiers continue to use `appointments` to avoid unnecessary compatibility breakage.

## Functional Files Changed

Once personal branding-only changes are excluded, the functional customization set affects 38 tracked source/translation files.

### Client components

- `client/src/components/ui/DatePicker.tsx`
- `client/src/components/ui/Select.tsx`

### Client pages

- `client/src/pages/Calendar.tsx`
- `client/src/pages/Dashboard.tsx`
- `client/src/pages/Kiosk.tsx` — recurring-event changes only for the public package
- `client/src/pages/Planning.tsx`
- `client/src/pages/Recipes.tsx`

### Server

- `server/src/db.ts`
- `server/src/lib/notifications.ts`
- `server/src/routes/appointments.ts`
- `server/src/routes/categories.ts`
- `server/src/routes/familyInvites.ts`

### Translations

Affected translation files are under:

- `client/src/i18n/locales/en/`
- `client/src/i18n/locales/fr/`
- `client/src/i18n/locales/pt/`
- `client/src/i18n/locales/zh/`

## Explicitly Excluded From the Community Version

The following are installation-specific or personal and should not be included in a public patch/package:

- customized `client/public/OpenFamily.png`
- customized favicon files
- customized Apple touch icon
- customized PWA icon files
- branding/cache-busting-only changes in `client/index.html`
- branding/cache-busting-only changes in `client/src/components/layout/Layout.tsx`
- branding/cache-busting-only changes in `client/src/pages/Login.tsx`
- branding/cache-busting-only changes in `client/vite.config.ts`
- the `OpenFamily.png?v=2` branding hunk in `client/src/pages/Kiosk.tsx`
- local `compose.override.yaml`
- all `*.backup*` files
- `client/public/icon-backup-original/`

These exclusions do not remove any of the functional features described above.

## Privacy Review

A source-diff privacy scan reported no obvious added:

- email addresses
- web URLs
- IP addresses
- local `/home`, `/Users`, or `/opt` filesystem paths
- obvious password/secret/API-key/access-token assignments

This automated scan is a safety check, not a guarantee. The public package should still be generated from an explicit allow-list of functional files and reviewed before publication.

## Suggested Community Release Description

> A set of quality-of-life and calendar enhancements for OpenFamily, based on commit `801757e`. Major additions include recurring Calendar Events with per-occurrence editing/deletion, recurrence intervals, shared event colors, Calendar Event search, improved same-day event browsing, Sunday-first calendars and date pickers, recipe image display, scrollable long select menus, expanded category capacity, and recipient-language-aware family notifications. User-facing Appointment terminology is also updated to Calendar Events across the supported interface translations included in this customization set.

## Packaging Recommendation

For a public release, generate the patch/package from an explicit allow-list of the functional files above rather than publishing the entire working directory.

This prevents accidental inclusion of:

- personal branding
- local deployment configuration
- backup files
- runtime data
- credentials or environment files
