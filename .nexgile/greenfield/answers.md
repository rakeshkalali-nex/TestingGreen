# Requirements brief — Scientific Calculator
## Profile
A scientific calculator with a React frontend, Java backend and SQLite storage, for people doing scientific calculations. · web app · Scientific calculation · size: small · depth: short
- **Has a user interface?** Yes (The description explicitly requests a React frontend.)
- **Runs locally?** Yes (The React pages and Java backend can run on the person's machine with a local SQLite database.)
## The app
The app is a scientific calculator with a React frontend, Java backend and SQLite storage, supporting complete expressions with arithmetic, powers, roots, logarithms and trigonometry. It retains successful calculation history and calculator preferences between uses, with a modern, professional frontend and no deployment.
## Ruled out by the person
- OUT1 Deployment: "no deployment" (description) [no requirement, design, task or code for it]
## Goals and success
- G1 Provide a scientific calculator. (description)
## Roles
- R1 One person uses the calculator locally to enter expressions, obtain results, reuse calculation history and change calculator preferences; no sign-in is required. — Person doing scientific calculations (1) [never signs in to the app] [assumed, high impact] (assumption)
## Workflows
### W1 Calculate a scientific expression (release 1) — actors: R1 Person doing scientific calculations; trigger: The person needs the result of a scientific expression.; outcome: The person has a usable result and a saved calculation. [assumed] (assumption, Q6, Q2, Q3, Q4)
Enter a complete expression, calculate its result, and retain successful calculations in history.
1. The person enters a complete expression.
2. The person selects degrees or radians when using trigonometry.
3. The person requests the result.
4. The app checks the expression and calculates it, or explains an error while keeping the expression available to correct.
5. The app shows the result and saves the successful expression with its result in calculation history.
6. The person copies the result or edits the expression for another calculation.
- X1 If an expression is incomplete or cannot produce a valid result, explain the problem, keep the expression for correction, and do not save it as a successful calculation. [assumed] (assumption, Q6)
## Records
- E1 Calculation history: Calculation history is saved in SQLite between uses. A successful calculation creates a saved entry; reusing an entry copies its expression into the calculator without changing the entry, while deleting an entry or clearing history permanently removes it.. fields: Calculation identifier, Expression, Result, Angle unit used, Calculation date and time; states: Saved → Deleted; owner: Person doing scientific calculations; how many: Plan for up to 10,000 saved calculations for one local user.; kept for: Until the person deletes the entry or clears history. [assumed, high impact] (Q4, assumption)
- E2 Calculator preferences: Calculator preferences are saved in SQLite between uses. First use creates default preferences of degrees and 12 significant digits; changing a preference saves the customised values, and resetting preferences restores the defaults.. fields: Preferences identifier, Angle unit: degrees or radians, Display precision: number of significant digits, Last updated date and time; states: Default → Customised; owner: Person doing scientific calculations; how many: One preferences record for the local calculator.; kept for: Retained between uses; resetting replaces the saved values with defaults. [assumed, high impact] (Q4, assumption)
## Scope and constraints
- S1 Use Java, SQLite and React. [constraint] (description)
- S3 Support arithmetic, powers, roots, logarithms and trigonometry in the first release. (Q2)
## Decisions
- **Stack:** Java, SQLite and React (Spring Boot) (as the user's description or documents say, description: "Java, Sqlite and React" Greenfield picked Spring Boot, as you named no framework.) (Q1)
- **Deploy target:** No deploy for now (as the user's description or documents say, description: "no deployment") (Q5)
## Assumptions
- A1 People enter an expression, choose the angle unit when needed, calculate, and copy or use the result; successful calculations are saved automatically, while errors leave the expression available for correction. — This supports complete-expression entry without making people save each calculation manually.; high impact [assumed] (assumption, Q6)
- A2 There are no separate administrator roles or affected non-users; the calculator user bears the risk of relying on an incorrect result. — The described app is a small calculator rather than a shared business service.; high impact [assumed] (assumption)
- A3 No fixed delivery date or budget is assumed; the first release is bounded by the agreed scientific functions, complete-expression entry, saved history and saved preferences. — No delivery date or budget was supplied.; high impact [assumed] (assumption)
## Other facts
- F1 The frontend should be modern and professional. (description)
- F2 People enter a complete expression before calculating. [workflows] (Q3)
## Out of scope and later releases
- S2 Do not deploy the app. [out of scope] (description)
- S4 Graph plotting, symbolic algebra, programming and unit conversion are outside the first release. [out of scope] [assumed, high impact] (assumption)
- Hosting and deployment: not relevant, the user said so [out of scope]
## The interview
- Q2 [scope · owner] Which range of calculations should the calculator support first? → Arithmetic, powers, roots, logarithms and trigonometry
- Q3 [workflows · owner] How should people enter a calculation? → Enter a complete expression before calculating
- Q4 [entities · owner] What should SQLite keep between uses? → Calculation history and calculator preferences
- Q6 [workflows · owner] Using a scientific calculation from your work as an example, how should someone go from entering the complete expression to finishing their task with the result? → Decide for me (settled: People enter an expression, choose the angle unit when needed, calculate, and copy or use the result; successful calculations are saved automatically, while errors leave the expression available for correction.)
