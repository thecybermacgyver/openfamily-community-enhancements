# OpenFamily — UI & AI Refinements Comparison

## Baseline and package relationship

This third community package documents a smaller set of user-interface refinements and Google Gemini AI-provider support developed after the original community enhancement package and the Budget & Calendar Integration follow-up.

- Upstream repository: `NexaFlowFrance/OpenFamily`
- Community repository: `thecybermacgyver/openfamily-community-enhancements`
- Original upstream baseline: `801757e`
- Package 3 patch: `openfamily-ui-ai-refinements.patch`
- Package 3 manifest: `openfamily-ui-ai-refinements-files.txt`
- Package relationship: **incremental to the contributor's working customized installation**

As with the earlier community packages, this is published as a review/reference package rather than presented as automatically merge-ready against current upstream `main`.

The patch was generated from explicit pre-change backups for the successful UI refinements and from the pre-Gemini backup created immediately before Gemini support was added. Git was then used to generate and reverse-verify the final patch against the working installation.

The package contains functional source and translation changes only. Personal branding, icons, deployment configuration, backups, credentials, runtime data, server-specific details, and unfinished Family Posts work are not included.

## Previous behavior vs Package 3 customization

| Area | Previous behavior | Package 3 customization |
|---|---|---|
| Budget forecast | Forecast balance and future recurring amounts are always shown when forecast data exists | Forecast card remains visible but its values are hidden by default behind an eye toggle |
| Budget focus | Forecast can visually compete with Current Balance | Current Balance remains the primary figure until the user deliberately reveals Forecast |
| Recurring Budget entries on phones | Desktop row layout compresses labels, dates, recurrence text, amounts, and actions into a narrow mobile width | Recurring entries use a responsive mobile grid/card layout while retaining the desktop row layout at `sm` and above |
| Calendar on phones | Seven-column monthly Calendar compresses badly on narrow screens | Calendar grid is placed in a horizontal scroll container with a usable minimum width |
| Planning on desktop | Week layout expands into seven narrow desktop columns | Planning keeps the same single-column day-card layout used successfully on mobile |
| AI providers | Ollama, OpenAI-compatible APIs, and Anthropic | Adds **Google Gemini** as a first-class provider |
| Gemini endpoint configuration | No native Gemini option | Uses Google's Gemini `generateContent` endpoint directly; no custom Base URL is required |
| Gemini structured output | Not supported | Uses Gemini structured JSON output with `responseMimeType` and `responseJsonSchema` so it fits OpenFamily's existing validated AI pipeline |
| Gemini authentication | Not supported | Uses the Google Gemini API key through `x-goog-api-key`; the key remains in OpenFamily's server-side AI settings |
| AI provider diagnostics | Provider failures are returned to the client but underlying `AiError` detail is not logged | AI provider errors log the machine-readable code and provider message before returning the existing 502 response |
| AI database provider constraint | `ollama`, `openai`, `anthropic` only | Database schema and migration widen the provider constraint to include `gemini` |
| AI settings translations | No Gemini provider label | Adds Google Gemini provider labels across English, French, Portuguese, and Chinese |

## Budget forecast visibility

The existing Forecast card is preserved, but the forecast amount and explanatory income/expense summary are now hidden by default.

A local `showForecast` state starts as `false`. The Forecast heading contains an eye button:

- eye icon — reveal Forecast
- eye-off icon — hide Forecast again

The Forecast calculation itself is unchanged. This is deliberately a presentation change only: it reduces distraction from Current Balance without removing access to the forward-looking information.

No preference is persisted. Opening or refreshing the Budget page returns Forecast to its hidden-by-default state.

## Responsive recurring Budget entries

Recurring Budget entries previously used one horizontal flex row for every viewport size. On a phone this caused category, date, frequency, amount, and action controls to compete for the same narrow line, making recurrence details difficult to read.

Package 3 changes only the recurring-entry card layout.

### Mobile behavior

The card uses a three-column grid:

- reconciliation checkmark
- main content / amount
- edit-delete actions

The recurring entry title can wrap rather than being forced into truncation. Category, occurrence date, and frequency use a wrapping metadata row with non-breaking individual labels, so each item stays readable.

The amount moves onto its own mobile grid row instead of squeezing beside the descriptive metadata.

### Desktop behavior

At the `sm` breakpoint and above, the card returns to the existing horizontal flex-row presentation. The rest of the Budget page is not changed by this responsive fix.

## Mobile Calendar scrolling

The monthly Calendar grid is seven columns wide by design. Rather than reducing each day cell until the contents become unreadable on a phone, Package 3 wraps the Calendar header and grid in:

- `overflow-x-auto`
- an inner minimum width of `800px`
- `lg:min-w-0` so normal desktop sizing resumes on large screens

This follows the same interaction pattern that already worked well elsewhere in the customized application: the content keeps a practical width and the phone user scrolls horizontally through it.

No Calendar recurrence, event-color, search, or data behavior is changed by this package.

## Planning layout consistency

Planning previously changed from one column on small screens to seven columns on large screens.

The customized mobile layout proved more readable, so Package 3 removes the `lg:grid-cols-7` expansion and keeps:

- one day card per row
- consistent card width
- the same visual structure across phone and desktop

This is a layout-only change. Planning data, schedules, editing behavior, and week navigation are unchanged.

## Google Gemini AI provider

Package 3 adds Google Gemini to OpenFamily's existing AI-provider abstraction rather than building a separate Gemini-only feature path.

### Client settings

The AI provider type now includes:

- `gemini`

Settings exposes **Google Gemini** alongside the existing providers.

Gemini differs from the OpenAI-compatible and Ollama providers in two ways:

1. it does not expose a user-editable Base URL
2. its API-key placeholder reflects Google's key format rather than an `sk-…` style key

The configured model is still editable by the user. The UI includes a Gemini model placeholder but does not hard-code the model into the provider implementation.

### Server provider registration

`server/src/services/ai/index.ts` adds Gemini to:

- the `AiProvider` union
- provider defaults
- completion dispatch

`server/src/routes/ai.ts` accepts `gemini` as a valid configured provider and treats it like Anthropic with respect to Base URL storage: Gemini uses its official endpoint and does not require a saved custom Base URL.

## Gemini request implementation

The new provider implementation is:

- `server/src/services/ai/gemini.ts`

It calls:

`https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent`

Authentication is sent with:

- `x-goog-api-key`

The request uses OpenFamily's existing system/user prompt abstraction and asks Gemini for structured JSON using:

- `generationConfig.responseMimeType = application/json`
- `generationConfig.responseJsonSchema = request.jsonSchema`

This is important because OpenFamily's AI features expect validated structured data rather than arbitrary conversational prose.

The provider extracts text from Gemini's first candidate, passes the result through OpenFamily's existing JSON extractor, and maps Gemini usage metadata into the shared token-usage shape.

## Gemini error handling

The provider maps common failures into OpenFamily's existing `AiError` codes.

Examples include:

- missing or rejected API key → `AI_UNAUTHORIZED`
- missing / unavailable model → `AI_MODEL_NOT_FOUND`
- empty provider result → `AI_INVALID_RESPONSE`
- other upstream Gemini failures → `AI_PROVIDER_ERROR`

The generic AI route additionally logs provider errors with:

- AI error code
- provider error message

This retains the existing client-facing behavior while making server diagnostics substantially more useful. For example, a temporary Gemini high-demand response can be distinguished from a bad API key or broken local networking without changing the API contract.

## Database compatibility

Gemini support requires widening the existing `ai_settings.provider` CHECK constraint.

Package 3 includes:

- `server/migrations/006_add_gemini_ai_provider.sql`
- corresponding `server/schema.sql` support for fresh installations
- startup migration logic in `server/src/db.ts`

The migration checks the existing provider constraint and replaces it only when Gemini is not already accepted.

The allowed provider values become:

- `ollama`
- `openai`
- `anthropic`
- `gemini`

No existing provider settings are removed or converted.

## Localization

The Gemini provider label is added to AI settings translations for:

- English
- French
- Portuguese
- Chinese

OpenFamily added Russian after this customization branch was established. An upstream rebase should add the equivalent Russian Gemini provider label and review any related AI-settings copy introduced since the baseline.

## Files included

The verified package manifest contains 15 files:

### Client

- `client/src/i18n/locales/en/ai.json`
- `client/src/i18n/locales/fr/ai.json`
- `client/src/i18n/locales/pt/ai.json`
- `client/src/i18n/locales/zh/ai.json`
- `client/src/lib/aiStatus.ts`
- `client/src/pages/Budget.tsx`
- `client/src/pages/Calendar.tsx`
- `client/src/pages/Planning.tsx`
- `client/src/pages/Settings.tsx`

### Server

- `server/migrations/006_add_gemini_ai_provider.sql`
- `server/schema.sql`
- `server/src/db.ts`
- `server/src/routes/ai.ts`
- `server/src/services/ai/gemini.ts`
- `server/src/services/ai/index.ts`

## Verification performed

Before publication, the package patch was generated with Git from explicit before/after snapshots and checked with Git itself.

The final package passed:

- Git patch parsing
- reverse `git apply --check` against the working customized installation
- successful OpenFamily rebuilds after each included functional change in the working installation
- direct Gemini API-key validation
- direct Gemini structured-output request validation
- Gemini request validation from inside the OpenFamily server container
- successful OpenFamily **Test connection** using an available Gemini model

The UI changes were also confirmed in the running application, including the mobile Calendar scroll, Planning layout, hidden Forecast toggle, and responsive recurring Budget entries.

## What is deliberately not included

Package 3 excludes installation-specific and unfinished material, including:

- personal app icons and branding
- personal photos
- local deployment or Compose configuration
- backup directories and backup files
- credentials or API keys
- runtime/database contents
- server addresses or private network details
- unfinished Family Posts experiments

The Gemini patch contains provider integration code only; it does not contain the contributor's Gemini API key.

## Compatibility notes

This package is incremental to the customized installation represented by the earlier community packages. In particular:

- the Forecast and recurring-entry UI refinements apply to the expanded Budget interface from the Budget & Calendar work
- the Calendar scrolling change applies to the customized Calendar page that already contains the earlier recurrence/color/search restructuring

For current upstream OpenFamily, review or rebase the relevant commits rather than assuming this incremental patch can be applied directly and unchanged.

## Package files

Patch:

**[`openfamily-ui-ai-refinements.patch`](openfamily-ui-ai-refinements.patch)**

Manifest:

**[`openfamily-ui-ai-refinements-files.txt`](openfamily-ui-ai-refinements-files.txt)**

Community repository:

**[thecybermacgyver/openfamily-community-enhancements](https://github.com/thecybermacgyver/openfamily-community-enhancements)**
