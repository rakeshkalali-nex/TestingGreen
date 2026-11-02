# Requirements: Scientific Calculator

## Requirement 1: Calculate a scientific expression

**User Story:** As a Person doing scientific calculations, I want to enter a complete expression and obtain a result, so that I can use it in my work.

_Source: G1, R1, W1, X1, S1, S3, A1, F1, F2_

### Acceptance Criteria

1. WHEN the person opens `/` THE SYSTEM SHALL show the Scientific Calculator without sign-in, with an expression display above a keypad of digit, decimal, arithmetic and scientific-function buttons, a result display, an angle-unit selector, and access to Calculation history and Calculator preferences.
2. WHEN the person types into the expression display or clicks keypad buttons THE SYSTEM SHALL allow entry and editing of a complete expression, including parentheses, with backspace to remove a character and clear to empty the expression and result.
3. WHEN the person selects Calculate or presses Enter with a valid arithmetic expression THE SYSTEM SHALL display its result using parentheses and conventional operator precedence, including `2 + 3 * 4 = 14` and `(2 + 3) * 4 = 20`.
4. WHEN the person calculates an expression containing powers, roots or logarithms THE SYSTEM SHALL support exponentiation, square root, natural logarithm and base-10 logarithm, including `2^3 = 8`, `sqrt(81) = 9`, `ln(1) = 0` and `log(100) = 2`.
5. WHEN the person calculates an expression containing sine, cosine or tangent THE SYSTEM SHALL use the selected degrees or radians setting, including `sin(30) = 0.5` in degrees and `cos(0) = 1` in radians.
6. WHEN a calculation succeeds THE SYSTEM SHALL display the result at the saved display precision and automatically save one Calculation history entry containing the expression, result, angle unit used, calculation identifier, and calculation date and time.
7. IF an expression is incomplete, divides by zero or has no finite real result THEN THE SYSTEM SHALL show an explanatory error instead of a result while retaining the expression for correction and leaving Calculation history unchanged.
8. WHEN the person selects Copy result after a successful calculation THE SYSTEM SHALL copy the displayed result to the clipboard without changing the expression or adding a Calculation history entry.
9. WHEN the person edits an expression after calculating THE SYSTEM SHALL keep the expression editable for the next calculation without changing the previously saved entry.
10. WHILE any calculator screen is open THE SYSTEM SHALL present a consistent modern, professional interface with styled controls, visible keyboard focus, labelled actions, and no horizontal page scrolling at viewport widths of 360 and 1280 pixels.

## Requirement 2: Reuse and manage Calculation history

**User Story:** As a Person doing scientific calculations, I want to reuse and manage Calculation history, so that I can revisit previous work and remove calculations I no longer need.

_Source: R1, E1, W1, A1_

### Acceptance Criteria

1. WHEN the person opens Calculation history THE SYSTEM SHALL show saved calculations newest first, with each entry's expression, result, angle unit used, calculation date and time, and actions to reuse or delete the entry, plus Clear history and a return action to the Scientific Calculator.
2. WHEN the person reopens the app THE SYSTEM SHALL retain Calculation history in SQLite, supporting up to 10,000 saved calculations until the person deletes an entry or clears history.
3. WHEN the person reuses a saved calculation THE SYSTEM SHALL return to the Scientific Calculator with that entry's expression and angle unit selected, without calculating automatically or modifying the saved entry.
4. WHEN the person deletes a Calculation history entry THE SYSTEM SHALL permanently remove only that entry from the displayed and saved history.
5. WHEN the person selects Clear history THE SYSTEM SHALL permanently remove all saved calculations and show an empty-history message, also shown when no calculation has yet been saved.

## Requirement 3: Change Calculator preferences

**User Story:** As a Person doing scientific calculations, I want to change Calculator preferences, so that calculations use my angle unit and display precision between uses.

_Source: R1, E2, W1_

### Acceptance Criteria

1. WHEN the person opens Calculator preferences THE SYSTEM SHALL show the current angle unit and display precision, controls for degrees or radians and significant digits, a Reset preferences action, and a return action to the Scientific Calculator.
2. WHEN the app is used for the first time THE SYSTEM SHALL create default Calculator preferences of degrees and 12 significant digits.
3. WHEN the person changes the angle unit in the calculator or changes a valid preference in Calculator preferences THE SYSTEM SHALL save the selected values and last updated date and time in the single local SQLite preferences record, apply them to subsequent calculations, and restore them when the app is reopened.
4. IF the person enters a display precision other than a whole number from 1 through 15 THEN THE SYSTEM SHALL show a validation message stating that range without changing the saved precision.
5. WHEN the person selects Reset preferences THE SYSTEM SHALL restore and save degrees and 12 significant digits without changing Calculation history.

## Later releases

- S4 Graph plotting, symbolic algebra, programming and unit conversion are outside release 1.

## Assumptions

- Arithmetic includes addition, subtraction, multiplication, division, negative numbers and decimal numbers; parentheses and powers take precedence over multiplication and division, which take precedence over addition and subtraction.
- Release-1 roots mean square roots, logarithms mean natural and base-10 logarithms, and trigonometry means sine, cosine and tangent over real numbers.
- Display precision accepts 1–15 significant digits, rounds the displayed result, and does not pad exact integers with trailing zeros; changing precision does not rewrite saved results.
- Reusing a calculation restores its recorded angle unit through the same preference-saving behaviour as selecting the angle unit directly.
