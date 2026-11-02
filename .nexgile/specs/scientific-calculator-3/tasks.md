# Tasks: Scientific Calculator

- [ ] 1. Project setup — Java API, SQLite and React foundation
  - Create the Java 21 / Spring Boot 3.5 / Maven server and React 19 / TypeScript / Vite npm workspace, with Node.js 22 prerequisites and the approved install, test and local-run commands in README.md. Root scripts coordinate both packages; `.gitignore` includes `.env`, dependency/build output, runner temporary files and SQLite files with WAL/SHM companions.
  - Implement every HTTP endpoint in design.md, including health, calculation, paginated history, single/all-history deletion and preference read/update/reset. Match the exact request/response shapes, status codes, validation messages and error envelope; use prepared SQL, same-origin mutation checks and generic unexpected-error responses.
  - Build the bounded Java tokenizer/parser, precedence and scientific functions, degree/radian conversion, domain checks and significant-digit formatting exactly as designed. Serialize writes, atomically save successful calculations with their preference snapshot, and never save invalid calculations or retry mutations automatically.
  - Supply versioned SQLite initialization, default singleton preferences, indexed newest-first history and pages of 50. Preserve saved data on restart, support 10,000 entries without pruning, and keep preference reset and history deletion independent.
  - Implement the shared API client, exact contracts and CalculatorProvider interface from design.md, including serialized preference writes, draft revisions and save-before-reuse behavior. Wire `/`, `/history` and `/preferences` to compile-safe default-export placeholders, unknown routes to `/`, and the shared labelled navigation so later screens need no router changes.
  - Write the approved visual tokens, global font, focus and control states, responsive header and test utilities. Add JUnit parser/API/persistence tests and `web/src/test/foundation.test.tsx` for the shared foundation; cover the approved numerical examples, errors, atomicity, precision, defaults/reset, reopening and 10,000-entry pagination. Server tests initialize and clean independent temporary SQLite files and start HTTP servers on automatically assigned ports.
  - Implement the development runner and startup port-file handshake: Java binds to `127.0.0.1` on port zero, Vite binds to `127.0.0.1` using the supplied `PORT` and proxies `/api` using the discovered internal port. Support a free page port when none is supplied, startup failure reporting and child-process cleanup on Windows. Add `@playwright/test`, `playwright.config.ts` and `e2e/.gitkeep` for saved journeys; exclude browser journeys from Vitest. The root test runner executes Maven tests, non-watch web tests and the type-checked web build, without a manually running app or environment file.
  - _Requirements: 1.3, 1.4, 1.5, 1.6, 1.7, 2.2, 3.2, 3.3, 3.4, 3.5_
  - _Files: package.json, package-lock.json, scripts/dev.mjs, scripts/test.mjs, .gitignore, README.md, server/pom.xml, server/src/main/java/com/scientificcalculator/**, server/src/main/resources/**, server/src/test/**, web/package.json, web/vite.config.ts, web/tsconfig*.json, web/index.html, web/src/main.tsx, web/src/App.tsx, web/src/styles/tokens.css, web/src/styles/global.css, web/src/shared/**, web/src/test/**, web/src/screens/calculator/CalculatorScreen.tsx, web/src/screens/history/HistoryScreen.tsx, web/src/screens/preferences/PreferencesScreen.tsx, playwright.config.ts, e2e/.gitkeep_
  - _Verify: npm test_
  - _Area: setup_

- [ ] 2. Scientific Calculator — complete expression entry and results
  - Replace the calculator placeholder with the finished screen: expression display above the result, degree/radian selector, scientific-function strip, four-column digit/operator keypad, backspace, Clear, full-width Calculate and Copy result. Use a centered desktop card and wrapped phone controls with screen-owned styles using shared tokens; preserve visible focus, labelled controls and no horizontal page scrolling at 360 and 1280 pixels.
  - Implement caret/selection-aware keypad editing, canonical operator insertion, complete-expression Enter submission, length limits, result/error announcements and clipboard success/failure feedback. Use the shared provider and API, wait for preferences and pending writes, block duplicate submissions, retain invalid drafts, clear obsolete results on revision changes and leave saved history unchanged when editing or copying.
  - Add `CalculatorScreen.test.tsx` with React Testing Library and MSW covering entry/editing, keypad, Clear/backspace, Enter/Calculate, approved arithmetic/scientific result rendering, selected angle units, precision, saving responses, expression errors, loading/retry, pending-write sequencing, copying and retained drafts. Use the real evaluator tests from Project setup for numerical correctness; component tests check request counts and that edits/copies send no calculation mutation. Include accessible controls and responsive style checks, with real viewport behavior exercised by saved browser journeys.
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7, 1.8, 1.9, 1.10_
  - _Depends: 1_
  - _Files: web/src/screens/calculator/**_
  - _Verify: npm run test --workspace web -- CalculatorScreen_
  - _Verify: npm run build --workspace web_
  - _Area: ui_

- [ ] 3. Calculation history — browse, reuse and remove saved calculations
  - Replace the history placeholder with the finished screen: newest-first cards showing expression, saved result, angle unit and local date/time, total count, Reuse/Delete actions, Clear history, Previous/Next pagination and Back to Scientific Calculator. Put desktop actions beside entries and wrap them below on phones; add screen-owned styles using shared tokens with visible focus and no horizontal page scrolling at 360 and 1280 pixels.
  - Load fresh history on mount/refresh, show loading, retry and empty states, and apply deletes only after success. Move to the preceding page when the last entry is deleted, disable competing mutations and preserve confirmed records after failure. Await the provider's reuse operation before navigating to `/`, restoring the expression and saved angle unit without calculating or modifying the entry.
  - Add `HistoryScreen.test.tsx` with React Testing Library and MSW for ordered metadata, total/pages, fresh remounts, empty/loading/error states, deletion/clear successes and failures, last-row pagination and save-before-reuse navigation. Verify reuse does not call calculation or history mutation endpoints; use a test-owned route consumer of the shared provider rather than depending on the calculator screen implementation. Backend persistence and capacity tests are supplied by Project setup; include accessible actions and responsive style checks for the screen.
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 1.10_
  - _Depends: 1_
  - _Files: web/src/screens/history/**_
  - _Verify: npm run test --workspace web -- HistoryScreen_
  - _Verify: npm run build --workspace web_
  - _Area: ui_

- [ ] 4. Calculator preferences — save angle unit and display precision
  - Replace the preferences placeholder with the finished settings card: degree/radian segmented choices, labelled significant-digits input and range helper, save status, Reset preferences and Back to Scientific Calculator. Align labels/controls in two desktop columns and stack them on phones; add screen-owned styles using shared tokens, visible focus and no horizontal page scrolling at 360 and 1280 pixels.
  - Read the shared saved preferences, save angle choices immediately and valid precision on blur or Enter, and retain precision as a local draft while typing. Reject non-whole values outside 1–15 with the exact approved range message and no mutation. Disable controls while saving, announce saved status, preserve confirmed values on failure with retry, and replace an invalid draft after successful reset without changing history.
  - Add `PreferencesScreen.test.tsx` with React Testing Library and MSW for current/default values, loading/retry, valid saves and shared-state updates, invalid precision, pending/failing saves and reset success/failure. Assert reset sends no history mutation, and use a test-owned provider consumer to check subsequent shared values without depending on the calculator screen implementation. Backend tests from Project setup verify SQLite defaults, timestamps, singleton updates and restart persistence; include accessible controls and responsive style checks.
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 1.10_
  - _Depends: 1_
  - _Files: web/src/screens/preferences/**_
  - _Verify: npm run test --workspace web -- PreferencesScreen_
  - _Verify: npm run build --workspace web_
  - _Area: ui_

## Assumptions

- No additional product assumptions; tasks follow the approved requirements and design.
