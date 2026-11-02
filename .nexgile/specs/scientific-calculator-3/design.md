# Design: Scientific Calculator

## Overview

A local web app for the Person doing scientific calculations, using React, a Java Spring Boot backend and SQLite. The Scientific Calculator opens at `/` without sign-in. Calculation history and Calculator preferences have their own routes and retain their records between uses. Java owns expression evaluation, result formatting and persistence; React owns expression editing and presentation.

This design follows requirements 1.1–1.10, 2.1–2.5 and 3.1–3.5. It uses three complete screens, each independently buildable after Project setup supplies the working API, shared contracts, navigation and visual tokens.

## Architecture

- Java 21, Spring Boot 3.5, Maven, Spring Web, Spring JDBC and the Xerial SQLite JDBC driver. Use prepared SQL through JDBC rather than an ORM.
- React 19, TypeScript, Vite and React Router in `web/`; Node.js 22 LTS and npm provide frontend tooling and the root development runner.
- JUnit 5 and Spring Boot Test cover Java. Vitest, jsdom, React Testing Library and MSW cover React screens and their API interactions. `@playwright/test` is available for saved browser journeys.
- `server/` contains the Maven application. `web/` contains the React application. Root npm scripts coordinate both without shell-specific environment assignments.
- Vite serves the page on `127.0.0.1` at the `PORT` environment variable and proxies `/api` to Spring Boot. Vite supplies SPA fallback for direct visits to `/history` and `/preferences`.
- `scripts/dev.mjs` starts Spring Boot bound to `127.0.0.1` with port zero. A Java startup listener writes its actual bound port to a runner-created temporary port file. The runner waits for that file and a successful health response before starting Vite with the internal `API_PORT` environment variable. Vite reads `PORT` and `API_PORT` in its config, uses strict port binding, and never silently changes the requested page port.
- The runner uses a fresh port file per invocation, reports startup failures and terminates both child process trees on exit, including on Windows. Manual starts without `PORT` select a free page port and print its address; Greenfield's supplied `PORT` takes precedence.
- Normal local data lives in the ignored `server/data/calculator.sqlite`, resolved from the server working directory. Startup creates the directory, applies versioned schema initialization and inserts default preferences if missing. No external service or environment file is required.
- All browser requests use same-origin relative `/api` paths. Do not enable permissive CORS. Accept JSON for mutations and reject cross-origin browser mutation requests, including deletes, while permitting same-origin requests through the proxy.

### File boundaries and shared ownership

Project setup establishes these contracts before screen work begins:

| Area | Planned files and responsibility |
| --- | --- |
| Root tooling | `package.json`, `package-lock.json`, `scripts/dev.mjs`, `scripts/test.mjs`, `.gitignore`, `README.md`; install/test/dev scripts and prerequisites |
| Server | `server/pom.xml`, `server/src/main/java/com/scientificcalculator/**`, `server/src/main/resources/**`, `server/src/test/**`; every API endpoint, evaluator, schema, persistence and backend tests |
| Shared web foundation | `web/package.json`, `web/vite.config.ts`, `web/tsconfig*.json`, `web/index.html`, `web/src/main.tsx`, `web/src/App.tsx`, `web/src/styles/tokens.css`, `web/src/styles/global.css`, `web/src/shared/**`, `web/src/test/**`; shell, contracts, API client, provider and test setup |
| Scientific Calculator | `web/src/screens/calculator/**`; screen, editor/keypad, stylesheet and component tests |
| Calculation history | `web/src/screens/history/**`; list, pagination, reuse/delete actions, stylesheet and component tests |
| Calculator preferences | `web/src/screens/preferences/**`; preference controls, stylesheet and component tests |
| Browser journeys | `playwright.config.ts`, `e2e/.gitkeep`; runner configuration and location for subsequently saved journeys |

Project setup writes compile-safe default-export screen placeholders at `CalculatorScreen.tsx`, `HistoryScreen.tsx` and `PreferencesScreen.tsx` under their respective screen directories and wires the three routes once. Each screen replaces its own placeholder; it does not edit the shared router or another screen. Shared tokens and the provider are stable dependencies of every screen.

## Components and interfaces

### Browser routes and shared state

- `/`: Scientific Calculator, default export `CalculatorScreen`.
- `/history`: Calculation history, default export `HistoryScreen`.
- `/preferences`: Calculator preferences, default export `PreferencesScreen`.
- Unknown routes return to `/`. All screens share an application header with the title Scientific Calculator and labelled navigation links. Secondary screens also have a visible Back to Scientific Calculator action.

`web/src/shared/contracts.ts` defines these exact JSON shapes:

```ts
type AngleUnit = 'degrees' | 'radians';
type Preferences = {
  id: 1;
  angleUnit: AngleUnit;
  displayPrecision: number;
  updatedAt: string;
};
type Calculation = {
  id: string;
  expression: string;
  result: string;
  angleUnit: AngleUnit;
  calculatedAt: string;
};
type HistoryPage = {
  items: Calculation[];
  page: number;
  pageSize: number;
  total: number;
};
type ApiError = {
  error: { code: string; message: string; field: string | null };
};
```

Timestamps are UTC ISO-8601 strings; screens render them in the person's local date/time format. Result strings are already formatted by Java and are displayed and copied verbatim, not parsed or reformatted in JavaScript.

`web/src/shared/api.ts` exports `getPreferences()`, `patchPreferences(patch)`, `resetPreferences()`, `calculate(expression)`, `getHistory(page)`, `deleteCalculation(id)` and `clearHistory()`, matching the HTTP contracts below. It raises a typed error carrying the API error code, message and HTTP status.

`web/src/shared/CalculatorProvider.tsx` exposes `useCalculator()` with:

- `preferences: Preferences | null`, `preferencesStatus: 'loading' | 'ready' | 'error'`, `preferencesError: string | null` and `preferenceSaving: boolean`.
- `draftExpression: string`, `setDraftExpression(value: string)` and `draftRevision: number`. Every edit or history reuse increments the revision, letting the calculator clear obsolete results and errors.
- `refreshPreferences(): Promise<void>`, `savePreferences(patch: { angleUnit?: AngleUnit; displayPrecision?: number }): Promise<Preferences>` and `resetPreferences(): Promise<Preferences>`. Successful responses replace shared preferences; failures preserve the last saved values. Serialize writes and disable dependent actions until each write finishes.
- `reuseCalculation(entry: Calculation): Promise<void>` first saves the entry's angle unit, then sets the expression and increments its revision. The history screen awaits success before navigating to `/`; failure keeps the person on history with an error. Reuse never calls calculate.

The provider loads preferences once at application start. Each history mount and explicit refresh loads a new page from the server, so returning after calculation sees newly saved entries without a cross-screen history cache. The calculator disables Calculate until preferences have loaded and while a preference write or calculation is pending. It keeps result/error state locally and clears it when the draft revision changes. Duplicate submissions are blocked while pending; failed calculation requests are not automatically retried.

### HTTP API

All successful bodies use JSON except `204` responses. Unknown fields and malformed bodies return `400`; the shared error body is always `ApiError`. All endpoints are implemented and tested during Project setup.

| Method and path | Request | Success | Expected errors |
| --- | --- | --- | --- |
| `GET /api/health` | None | `200 { "status": "ok" }` after schema startup succeeds | `503`, `STORAGE_UNAVAILABLE` if the database probe fails |
| `GET /api/preferences` | None | `200 Preferences` | `503`, `STORAGE_UNAVAILABLE` |
| `PATCH /api/preferences` | JSON with one or both of `angleUnit`, `displayPrecision`; no other fields | `200 Preferences`, including updated timestamp | `400`, `INVALID_PREFERENCES`; `503`, `STORAGE_UNAVAILABLE` |
| `POST /api/preferences/reset` | No body | `200 Preferences` with degrees and 12 significant digits | `503`, `STORAGE_UNAVAILABLE` |
| `POST /api/calculations` | `{ "expression": string }` | `201 Calculation`; one committed history row, using saved angle unit and precision | `400`, `INVALID_EXPRESSION`; `422`, `DIVISION_BY_ZERO`, `DOMAIN_ERROR` or `NON_FINITE_RESULT`; `503`, `STORAGE_UNAVAILABLE` |
| `GET /api/history?page=0&pageSize=50` | Zero-based nonnegative integer page; pageSize fixed at 50 | `200 HistoryPage`, newest first by timestamp then internal insertion sequence | `400`, `INVALID_PAGE`; `503`, `STORAGE_UNAVAILABLE` |
| `DELETE /api/history/{id}` | UUID path parameter, no body | `204`, also when an already-deleted UUID is requested | `400`, `INVALID_ID`; `503`, `STORAGE_UNAVAILABLE` |
| `DELETE /api/history` | No body | `204`; delete all history, preserve preferences | `503`, `STORAGE_UNAVAILABLE` |

Invalid precision uses `field: "displayPrecision"` and message `Display precision must be a whole number from 1 through 15.` Invalid angle unit identifies `angleUnit`. Expression errors identify `expression`, for example `Complete the expression before calculating.`, `Cannot divide by zero.`, or `Square root requires a nonnegative number.` Other errors use a null field. Unexpected failures return `500` with code `INTERNAL_ERROR` and a generic message, never a stack trace or SQL text. Rejected cross-origin mutations return `403`, code `ORIGIN_NOT_ALLOWED`.

## Calculation rules

- Implement a bounded tokenizer and recursive-descent parser in Java, never dynamic code execution. Accepted tokens are decimal literals (including scientific notation), whitespace, `+ - * / ^`, parentheses and the single-argument functions `sqrt`, `ln`, `log`, `sin`, `cos`, `tan`. Names are lowercase. Multiplication is explicit; unsupported names and syntax produce explanatory validation errors.
- Grammar: expression is sums of products; product is multiplications/divisions of unary expressions; unary is an optional recursive sign or a power; power is a primary optionally followed by `^` and a unary expression; primary is a number, parenthesized expression, or function call. This gives right-associative powers, `2^3^2 = 512`, `-2^2 = -4`, and `2^-2 = 0.25`.
- Use Java double arithmetic and `StrictMath` for scientific functions. Reject division by zero, negative square-root arguments, nonpositive logarithm arguments and non-finite intermediates/results. Convert degrees before trigonometric functions. For tangent, reject inputs whose cosine has absolute value below `1e-15`, explaining that tangent is undefined at that angle.
- Reject empty expressions, expressions longer than 2,000 characters and nesting deeper than 100 levels with `400 INVALID_EXPRESSION`; the editor shows the same length bound. These bounds prevent the parser from consuming unbounded resources.
- Format finite results using `BigDecimal.valueOf(value)` and the saved number of significant digits with half-up rounding; strip insignificant trailing zeros and normalize negative zero to `0`. Use plain notation for nonzero rounded magnitudes from `1e-6` inclusive to `1e12` exclusive, scientific notation otherwise. The required examples display `14`, `20`, `8`, `9`, `0`, `2`, `0.5` and `1` respectively.
- A calculation reads a preference snapshot, evaluates and inserts the formatted result as one serialized write operation. Return success only after commit; a failed evaluation or save adds no row. A successful explicit recalculation is a new entry, even if the expression is unchanged. Editing, copying and reusing do not create entries.
- Display precision changes affect future calculations only; history retains its original formatted strings. This is a real-number scientific calculator with floating-point limitations, not exact symbolic mathematics.

## Data models

Use versioned SQL resources under `server/src/main/resources/db/` and a transactional schema initializer with a `schema_version` table. Enable foreign keys, a busy timeout of 5,000 ms and WAL mode for the local file. Serialize calculation/preference/history mutations in-process; JDBC writes use transactions. Tests use independent temporary files rather than the normal local path.

| Table | Columns and constraints |
| --- | --- |
| `calculation_history` | `sequence INTEGER PRIMARY KEY AUTOINCREMENT`, `id TEXT NOT NULL UNIQUE` (UUID exposed to clients), `expression TEXT NOT NULL`, `result TEXT NOT NULL`, `angle_unit TEXT NOT NULL CHECK` in degrees/radians, `calculated_at TEXT NOT NULL` (UTC) |
| `calculator_preferences` | `id INTEGER PRIMARY KEY CHECK (id = 1)`, `angle_unit TEXT NOT NULL CHECK` in degrees/radians, `display_precision INTEGER NOT NULL CHECK` between 1 and 15, `updated_at TEXT NOT NULL` |

Index history by `calculated_at DESC, sequence DESC`. Page reads and their total count use one read transaction. Pagination limits render work for 10,000 saved entries. There is no automatic pruning or hard 10,000-entry deletion threshold; history remains until explicitly removed. Default preferences are inserted only if absent, never overwritten on restart. Delete-all touches only `calculation_history`; reset touches only `calculator_preferences`.

## Look and feel

### Shared visual system

- Background `#F4F7FB`; surface `#FFFFFF`; primary text `#142033`; secondary text `#526176`; primary action `#2457D6`; error `#B42318`; success `#167044`; border `#D8E1ED`; focus ring `#0E7490`.
- Font: `Inter, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif`, served with system fallbacks and no external font dependency. Expression/result use `ui-monospace, Consolas, monospace` with tabular numbers.
- Shared CSS variables define colors, a 4/8/12/16/24/32 px spacing scale, 12 px control radius and 20 px card radius. Main content is at most 1120 px wide; page padding is 16 px on phones and 32 px on desktop. Surfaces have a fine border and restrained shadow.
- Primary buttons use white text on primary blue; hover `#1D46AF`, pressed `#17388C`. Secondary and digit buttons use pale surfaces, hover `#E8EEF8`, pressed `#DCE5F4`. Destructive controls use error-colored text and pale red hover/pressed backgrounds. Disabled controls have muted fills and text, do not respond to clicks, and retain readable labels.
- Every interactive control has a visible 3 px focus ring with 2 px offset. Fields have persistent labels, surface fill, border, darker hover border, focus ring and red border plus explanatory text when invalid. Disabled fields use `#EEF2F6`. Buttons and inputs have at least 44 px touch targets.
- List entries use white cards with separators; hover adds `#F6F8FC`, focused actions show the same ring, pressed actions darken their button background, and pending actions are disabled with text feedback. Do not rely on color alone for selection, errors or success.
- Use meaningful headings, native buttons/inputs, navigation current-page semantics, labelled icon actions and polite live regions for results and save/copy feedback; errors use an alert region. Reduced-motion preferences disable decorative transitions.

### Scientific Calculator — `/`

On desktop, center a calculator card up to 680 px wide beneath the shared header. Place the title and degrees/radians segmented selector above a subtly tinted display area: editable expression first, prominent right-aligned result below, then inline feedback and Copy result. A compact toolbar contains backspace and Clear. The scientific-function strip sits above a four-column numeric/operator keypad; the large Calculate button spans the final row. The keypad includes digits 0–9, decimal, `+`, `−`, `×`, `÷`, parentheses, power, `sqrt`, `ln`, `log`, `sin`, `cos` and `tan`. Visual operator symbols insert the canonical parser characters.

On phones, the card fills available width, the scientific strip wraps into three columns and the numeric keypad remains four columns with equal-width touch targets. The expression field wraps long input inside the card; the result can scroll inside its display region, never the page. Header links wrap neatly. Preserve this layout at 360 px and 1280 px without horizontal page scrolling.

Keypad insertion replaces the selected expression range or inserts at the last caret position, then restores editor focus and caret. Functions insert their name and opening parenthesis. Backspace removes the selection or previous character; Clear resets expression, result and feedback. Enter in the editor submits the complete expression, not an intermediate operation. A pending calculation displays `Calculating…`; errors replace the old result but retain the draft. Editing clears the prior result and disables Copy result until another success. Copy uses the Clipboard API and announces `Result copied`; denial shows `Could not copy. Select and copy the result manually.` without changing history.

### Calculation history — `/history`

On desktop, use a full-width history card with title, total count and Clear history above a vertically stacked list. Each row shows a monospace expression above its result, followed by an angle-unit badge and local date/time; Reuse and Delete sit on the right. Previous/Next controls and a page indicator appear below the list. On phones, actions wrap beneath the row contents, metadata wraps and long expressions wrap within the row.

Empty state: `No saved calculations yet.` with a Back to Scientific Calculator action. Loading and failed loading states occupy the list region and provide a Retry action on failure. Delete and Clear history act immediately on explicit activation, with destructive labels rather than unlabeled icons, and update the list only after server success. If the last row on a page is deleted, load the preceding page. Disable competing mutations while one is pending. Reuse saves the entry's angle unit before returning to the calculator; no automatic calculation occurs.

### Calculator preferences — `/preferences`

Use a settings card up to 640 px wide. Show an Angle unit row with degrees/radians segmented choices and a Display precision row with a labelled numeric input plus helper text `1–15 significant digits`. Below them show save status, Reset preferences and Back to Scientific Calculator. Desktop labels and controls align in two columns; phones stack each label, control and helper text.

Angle selection saves immediately. Precision stays a local draft while typing and saves on blur or Enter only when it is a whole number in range. Invalid drafts show the specified range message and do not send a mutation. Successful saves announce `Preferences saved`; failure retains saved shared values and offers retry. Reset saves defaults and replaces any invalid local draft after success, without touching history.

## Error handling and consistency

Initial preference failure blocks calculation and exposes Retry rather than pretending defaults were saved. Network/storage failures show actionable inline messages and preserve drafts and last confirmed records. A failed delete or reset leaves the displayed saved state intact. Preferences controls disable while saving so a calculation cannot overtake its own angle-unit update. Requests do not automatically repeat mutations after a lost response, because a calculation may already have committed.

History and preference screens remain navigable during loading. Request cancellation or stale-response guards prevent results from an unmounted screen overwriting newer UI state. API validation is authoritative; client validation improves feedback but does not replace it.

## Testing strategy

- Project setup supplies JUnit tests for parser precedence and every approved numerical example; invalid expressions and domains; significant-digit formatting; API schemas/status codes; preference validation/default/reset behavior; save atomicity; deletion/reuse prerequisites; persistence after reopening a temporary SQLite file; and paginated ordering across 10,000 entries. Each test receives its own temporary database, initializes its schema and cleans it up. HTTP tests use an automatically assigned server port.
- Setup also supplies one passing frontend foundation test and shared MSW/test-provider utilities. Vitest excludes `e2e/**` and browser journeys. Frontend type checking is included in the build script.
- Scientific Calculator tests cover 1.1–1.10: typing and keypad edits, Enter/Calculate, result/error rendering, preference-write sequencing, one pending submission, copying and unchanged history on editing. Backend tests provide real evaluation checks while MSW component tests check screen behavior.
- Calculation history tests cover 2.1–2.5 and the shared presentation criterion 1.10: states, ordered records, page navigation, reuse sequencing and failures, deletion and clearing.
- Calculator preferences tests cover 3.1–3.5 and 1.10: controls, loading, valid/invalid saves, reset and errors. Persistent defaults and reopening are also verified by backend tests.
- Saved Playwright journeys under `e2e/` exercise the real browser and Java API at the running preview address through Greenfield's journey runner, including keyboard focus, clipboard feedback and 360/1280 px layouts. They are not included in the default unit/integration test command.
- `scripts/test.mjs` runs `mvn -f server/pom.xml test`, `npm run test --workspace web` and `npm run build --workspace web` sequentially, exits nonzero at the first failure, and never starts watch mode. It needs no running app, fixed port or environment file. The web test script is `vitest run`; web build is `tsc -b && vite build`.

## Build and run

Install: npm install && mvn -f server/pom.xml dependency:go-offline
Test: npm test
Run locally: yes
Dev: npm run dev
Port variable: PORT
Health path: /api/health
Deploy: none

Root npm workspaces include `web`; `npm test` invokes `scripts/test.mjs`, and `npm run dev` invokes `scripts/dev.mjs`. Commands run from the app root on Windows cmd.exe as well as other ordinary shells. Prerequisites are Java 21, Maven 3.9 and Node.js 22 LTS with npm. Maven and Node tooling are prerequisites, not installed system-wide by the app. The README reproduces these commands and prerequisites. Ignore `.env`, dependency/build output, temporary runner files and SQLite data including WAL/SHM companion files.

## Assumptions

- Calculations use floating-point real-number arithmetic, not arbitrary-precision or symbolic mathematics; the chosen precision controls displayed rounding.
- Supported expressions require explicit multiplication, use lowercase function names and treat exponentiation as right-associative with precedence above unary minus.
- History has no automatic retention limit; 10,000 entries is the release-1 capacity tested, with pages of 50 entries.
- Delete and Clear history apply immediately when their labelled actions are selected; there is no undo record.
- One local app instance is the normal usage model; another concurrent instance sharing its data file is not a collaborative editing workflow.
