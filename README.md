# OpenFamily Community Enhancements

Community customization package for OpenFamily.

## Baseline

Built against:

- Repository: NexaFlowFrance/OpenFamily
- Commit: `801757e`
- Commit description: Bulk recipe ingredients import to Shopping List with multi-language grocery converter

## Main Improvements

### Calendar Events
- Renamed user-facing "Appointments" terminology to "Calendar Events"
- Daily, weekly, monthly and yearly recurrence
- Custom recurrence intervals
- Optional repeat-until date
- Edit one occurrence or all occurrences
- Delete one occurrence or all occurrences
- Monthly recurrence correctly handles months without the selected date
- Feb. 29 yearly recurrence correctly handles leap years
- Shared Calendar Event colors
- Fixed 16-color event palette
- Calendar Event search
- Search by title, description, location, notes and family-member names
- Clicking a populated date offers Display Events or Add Event
- Display all events for a selected date
- Sunday-first calendar layout
- Recurring-event support in Dashboard, Planning and Kiosk

### Date Picker
- Custom localized date picker
- Sunday-first weeks
- Previous/next month navigation
- Min/max date restrictions
- Separate date/time handling

### Recipes
- Recipe cards display `image_url`
- Original chef-hat remains as fallback

### Interface
- Long Select dropdowns now scroll instead of extending off-screen
- Category limit increased from 30 to 60

### Notifications
Family ownership and access-request notifications now use the recipient's saved language where supported:

- English
- French
- Chinese

### Localization
Calendar Event terminology and new Calendar features are localized across:

- English
- French
- Portuguese
- Chinese

## Installation

This patch is intended for OpenFamily baseline commit:

`801757e`

From the root of a matching OpenFamily checkout:

    git apply --check openfamily-community.patch
    git apply openfamily-community.patch

Always back up your installation and database before applying third-party modifications.

## Privacy

This community package deliberately excludes:

- personal images
- custom branding
- favicons and personal PWA icons
- local deployment configuration
- backup files
- runtime data
- credentials
- personal server information

The patch has also passed an automated privacy/exclusion scan.

## Files

- `openfamily-community.patch` — functional source changes
- `README.md` — overview and installation information
