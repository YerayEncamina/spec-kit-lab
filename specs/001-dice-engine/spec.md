# Feature Specification: Dice Rolling Engine

**Feature Branch**: `001-dice-engine`  
**Created**: 2026-02-06  
**Status**: Draft  
**Input**: User description: "Build a dice rolling engine that supports: - Standard dice notation (e.g., 2d6, 1d20+5, 3d8-2) - Parsing expressions like "2d6+1d4+3" - Return individual rolls and total - Support for advantage/disadvantage (roll twice, take higher/lower) - Cryptographically secure random number generation"

## Clarifications

### Session 2026-02-06

- Q: What is the interface/delivery mechanism for the dice rolling engine (library, REST API, CLI, WebSocket)? → A: REST API endpoint
- Q: What HTTP status code and error response format should be used for validation errors and server errors? → A: Standard HTTP status codes with structured JSON errors
- Q: What are the resource limits for die counts and die sizes to prevent abuse? → A: max 20 dices, max 20 sides
- Q: How should advantage/disadvantage be communicated in REST API requests (query parameter, body field, notation syntax)? → A: Query parameter (e.g., ?mode=advantage)
- Q: What HTTP method and request structure should be used (GET with query params vs POST with JSON body)? → A: POST with JSON body: {"expression": "2d6", "mode": "advantage"}

**Resolution Note**: Q5 supersedes Q4. The implementation MUST use JSON body field "mode" (not query parameter). The mode field is included in POST /api/roll body with valid values: "normal", "advantage", or "disadvantage". This decision supports consistency with expression parsing (body-based) and simplifies client implementation.

## User Scenarios & Testing _(mandatory)_

### User Story 1 - Basic Dice Rolling (Priority: P1)

Users need to roll standard dice using simple notation like "2d6" (two six-sided dice) or "1d20+5" (one twenty-sided die plus five). This is the foundation of any dice system and must work reliably for the most common use cases in tabletop gaming.

**Why this priority**: This is the core functionality that delivers immediate value. Without basic dice rolling, no other features matter. It's the minimum viable product for a dice engine.

**Independent Test**: Can be fully tested by sending HTTP POST requests with JSON body containing dice notation to the REST API endpoint and verifying the JSON responses contain results within expected ranges with both individual rolls and totals.

**Acceptance Scenarios**:

1. **Given** a POST request with JSON body {"expression": "2d6"}, **When** the user rolls the dice, **Then** the system returns a JSON response with two individual rolls (each 1-6) and their sum (2-12)
2. **Given** a POST request with JSON body {"expression": "1d20+5"}, **When** the user rolls the dice, **Then** the system returns a JSON response with one roll (1-20), the modifier (+5), and the total (6-25)
3. **Given** a POST request with JSON body {"expression": "3d8-2"}, **When** the user rolls the dice, **Then** the system returns a JSON response with three individual rolls (each 1-8), the modifier (-2), and the total (1-22)
4. **Given** a POST request with JSON body {"expression": "2d"}, **When** the user attempts to roll, **Then** the system returns HTTP 400 status with a structured JSON error message

---

### User Story 2 - Complex Expression Parsing (Priority: P2)

Users need to combine multiple dice rolls and modifiers in a single expression, such as "2d6+1d4+3" for calculating damage that involves multiple dice types. This enables more sophisticated game mechanics without requiring multiple separate rolls.

**Why this priority**: While not essential for basic functionality, this significantly improves user experience by allowing complex rolls in one operation rather than multiple steps.

**Independent Test**: Can be tested independently by passing complex expression strings and verifying the parser correctly identifies all components and evaluates them in the proper order.

**Acceptance Scenarios**:

1. **Given** a POST request with JSON body {"expression": "2d6+1d4+3"}, **When** the user rolls, **Then** the system returns all individual rolls from both dice groups, all modifiers, and the grand total
2. **Given** a POST request with JSON body {"expression": "1d20+2d6-1"}, **When** the user rolls, **Then** the system correctly combines results from different dice types with positive and negative modifiers
3. **Given** a POST request with JSON body {"expression": "2d6 + 1d4 + 3"}, **When** the user rolls, **Then** the system handles whitespace gracefully and produces correct results
4. **Given** a POST request with JSON body {"expression": "2d6+"}, **When** the user attempts to roll, **Then** the system returns HTTP 400 status with a structured JSON error indicating the incomplete expression

---

### User Story 3 - Advantage/Disadvantage (Priority: P3)

Users need to roll with advantage (roll twice, take higher) or disadvantage (roll twice, take lower) as commonly used in modern tabletop RPG systems. This mechanic affects probability distributions and is a popular game design pattern.

**Why this priority**: This is a valuable enhancement for specific game systems but not universally required. The basic and complex rolling features are more fundamental.

**Independent Test**: Can be tested by rolling with advantage/disadvantage flags and verifying that two rolls are made and the correct one (higher or lower) is selected.

**Acceptance Scenarios**:

1. **Given** a POST request with JSON body {"expression": "1d20", "mode": "advantage"}, **When** the user rolls, **Then** the system returns two individual d20 rolls and selects the higher value as the result
2. **Given** a POST request with JSON body {"expression": "1d20+5", "mode": "disadvantage"}, **When** the user rolls, **Then** the system returns two individual d20 rolls, selects the lower value, and applies the +5 modifier
3. **Given** a POST request with JSON body {"expression": "2d6", "mode": "advantage"}, **When** the user rolls, **Then** the system returns HTTP 400 status with a structured JSON error indicating advantage/disadvantage only applies to single die rolls
4. **Given** a POST request with conflicting mode values, **When** the user attempts to roll, **Then** the system returns HTTP 400 status with a structured JSON error indicating invalid mode

---

### Edge Cases

- What happens when a user specifies an invalid die size (e.g., "2d0" or "2d-6")?
- How does the system handle die counts or sizes exceeding limits (e.g., "21d6" or "2d100")?
- What happens when modifiers would result in a negative or zero total?
- How does the system handle malformed expressions with unbalanced operators (e.g., "2d6++")?
- What happens when advantage/disadvantage is applied to complex expressions?
- How does the system handle Unicode characters or special symbols in input?

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST expose dice rolling functionality via POST /api/roll endpoint that accepts JSON body and returns JSON responses
- **FR-002**: System MUST accept JSON request body with "expression" field containing dice notation and optional "mode" field
- **FR-003**: System MUST parse standard dice notation in the format "NdX" where N is the number of dice and X is the number of sides
- **FR-004**: System MUST parse dice notation with positive modifiers (e.g., "2d6+5")
- **FR-005**: System MUST parse dice notation with negative modifiers (e.g., "1d20-3")
- **FR-006**: System MUST parse complex expressions combining multiple dice groups and modifiers (e.g., "2d6+1d4+3")
- **FR-007**: System MUST return detailed JSON results including all individual die rolls, modifiers applied, and the total sum
- **FR-008**: System MUST use cryptographically secure random number generation for all dice rolls
- **FR-009**: System MUST support advantage mode where two dice are rolled and the higher value is selected (specified via "mode": "advantage" in request body). Advantage applies ONLY to single-die expressions (e.g., "1d20" or "1d20+5"), not to multi-die or multi-group expressions.
- **FR-010**: System MUST support disadvantage mode where two dice are rolled and the lower value is selected (specified via "mode": "disadvantage" in request body). Disadvantage applies ONLY to single-die expressions (e.g., "1d20" or "1d20+5"), not to multi-die or multi-group expressions.
- **FR-011**: System MUST validate input and return HTTP 400 (Bad Request) for malformed or invalid dice notation with structured JSON error responses
- **FR-012**: System MUST reject invalid die specifications such as zero-sided dice, negative dice counts, or negative die sizes
- **FR-013**: System MUST enforce resource limits: maximum 20 dice per group and maximum 20 sides per die
- **FR-014**: System MUST handle whitespace in expressions gracefully (spaces between operators and operands)
- **FR-015**: System MUST ensure each die roll is independent and uses fresh random values
- **FR-016**: System MUST return HTTP 500 (Internal Server Error) for unexpected server failures with structured JSON error responses
- **FR-017**: System MUST include error code, message, and optional details in all JSON error responses

### Error Codes (FR-017 Implementation)

All error responses must use one of these standardized error codes:

| Code | Status | Scenario |
|------|--------|----------|
| INVALID_NOTATION | 400 | Malformed syntax, incomplete expressions |
| RESOURCE_LIMIT_EXCEEDED | 400 | >20 dice or >20 sides per die |
| INVALID_MODE | 400 | Advantage/disadvantage on invalid expressions |
| INVALID_DIE_SIZE | 400 | Zero or negative die sizes |
| INTERNAL_ERROR | 500 | Unexpected server failures |

### Key Entities

- **DiceExpression**: Represents a complete dice rolling expression parsed from user input, containing one or more DiceGroups and modifiers
- **DiceGroup**: Represents a single group of dice (e.g., "2d6"), including count and die size
- **RollResult**: Contains the outcome of a dice roll, including individual die values, applied modifiers, and calculated total
- **Modifier**: Represents a numeric bonus or penalty applied to dice rolls (positive or negative integer)
- **RollMode**: Enumeration specifying normal, advantage, or disadvantage rolling behavior

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: The dice engine correctly parses and evaluates 100% of valid standard dice notation inputs (verified through automated test suite)
- **SC-002**: All dice rolls complete in under 10 milliseconds for expressions within resource limits (up to 20 dice)
- **SC-003**: The random number distribution passes statistical uniformity tests (chi-squared test with p-value > 0.05) over 10,000 rolls
- **SC-004**: The system rejects 100% of invalid inputs with clear, actionable error messages
- **SC-005**: Advantage and disadvantage mechanics produce statistically correct probability distributions when tested over 1,000 rolls
- **SC-006**: Complex expressions with multiple dice groups are evaluated correctly 100% of the time (verified through comprehensive test cases)
- **SC-007**: The dice engine handles at least 1,000 concurrent roll requests without performance degradation
