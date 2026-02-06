---
description: "Actionable implementation tasks for the Dice Rolling Engine feature"
---

# Tasks: Dice Rolling Engine

**Input**: Design documents from `/specs/001-dice-engine/`  
**Prerequisites**: ✅ plan.md, ✅ spec.md, ✅ research.md, ✅ data-model.md, ✅ contracts/ (openapi.yaml)

**Organization**: Tasks are organized by user story to enable independent implementation and testing.  
**Test Strategy**: TDD (Test-First) - Write failing tests before implementation per constitution requirement.  
**Path Convention**: Web app structure using `packages/api/src/` and `packages/api/tests/` per plan.md

---

## Phase 1: Setup (Project Initialization)

**Purpose**: Create solution structure, configure dependencies, establish development environment

- [ ] T001 Create .NET solution and project structure in `packages/api/` directory
- [ ] T002 Create 6 projects: AdventureGame.Api, AdventureGame.Core, AdventureGame.Infrastructure, and 3 test projects
- [ ] T003 [P] Configure project references: Infrastructure→Core, Api→[Core+Infrastructure]
- [ ] T004 [P] Add NuGet dependencies: ASP.NET Core, EF Core, Npgsql, xUnit, FluentAssertions, Swashbuckle
- [ ] T005 Create .gitignore and solution-level configuration files
- [ ] T006 Create solution README with build/test/run instructions

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure required before implementing user stories

**⚠️ CRITICAL**: No user story implementation begins until this phase completes

- [ ] T007 [P] Define domain interfaces in `src/AdventureGame.Core/Services/IDiceParser.cs`
- [ ] T008 [P] Define domain interfaces in `src/AdventureGame.Core/Services/IDiceRoller.cs`
- [ ] T009 [P] Create exception types in `src/AdventureGame.Core/Exceptions/` (InvalidNotationException, ResourceLimitExceededException)
- [ ] T010 [P] Create `RollMode.cs` enum in `src/AdventureGame.Core/Models/RollMode.cs` (Normal, Advantage, Disadvantage)
- [ ] T011 Configure Program.cs in API project: DI container, exception middleware, CORS, Swagger (sets basePath for dependencies)
- [ ] T012 Create global exception handler middleware in `src/AdventureGame.Api/Middleware/ExceptionHandlingMiddleware.cs`
- [ ] T013 [P] Create error response DTO in `src/AdventureGame.Api/Models/ErrorResponse.cs` (code, message, details)
- [ ] T014 [P] Create base validation result type in `src/AdventureGame.Core/Models/ValidationResult.cs`

**Checkpoint**: Foundation ready - all user stories can now proceed independently in parallel

---

## Phase 3: User Story 1 - Basic Dice Rolling (Priority: P1) 🎯 MVP

**Goal**: Users can roll standard dice notation (2d6, 1d20+5, 3d8-2) and receive results with individual rolls and total  
**Independent Test**: Can be fully tested via HTTP POST to /api/roll with acceptance scenarios from spec  
**Estimated Effort**: 40-50% of total implementation

### Tests for User Story 1 (REQUIRED - TDD)

> **Write these FIRST, ensure they FAIL before implementation**

- [ ] T015 [P] [US1] Create unit test fixture `tests/AdventureGame.Core.Tests/DiceParserTests.cs` with infrastructure-agnostic tests
- [ ] T016 [P] [US1] Add parameterized tests for valid single-group notation (2d6, 1d20, 1d8) in DiceParserTests
- [ ] T017 [P] [US1] Add parameterized tests for notation with single positive modifier (+5, +1) in DiceParserTests
- [ ] T018 [P] [US1] Add parameterized tests for notation with single negative modifier (-2, -1) in DiceParserTests
- [ ] T019 [P] [US1] Add tests for invalid notations (2d, d6, 2d0, 0d6, 21d6, 2d21) in DiceParserTests
- [ ] T020 [P] [US1] Create unit test fixture `tests/AdventureGame.Infrastructure.Tests/RegexDiceParserTests.cs` for implementation
- [ ] T021 [P] [US1] Add tests for regex pattern matching edge cases and error scenarios in RegexDiceParserTests
- [ ] T022 [P] [US1] Create unit test fixture `tests/AdventureGame.Core.Tests/DiceRollerTests.cs`
- [ ] T023 [P] [US1] Add tests for roll distribution (all dice produce values 1..sides) in DiceRollerTests
- [ ] T024 [P] [US1] Add tests for modifier application (positive and negative) in DiceRollerTests
- [ ] T025 [P] [US1] Create integration test fixture `tests/AdventureGame.Api.Tests/DiceControllerTests.cs`
- [ ] T026 [P] [US1] Add integration test for POST /api/roll with valid 2d6 notation in DiceControllerTests
- [ ] T027 [P] [US1] Add integration test for POST /api/roll with modifier (1d20+5) in DiceControllerTests
- [ ] T028 [P] [US1] Add integration test for POST /api/roll returning correct JSON schema in DiceControllerTests
- [ ] T029 [P] [US1] Add integration test for POST /api/roll with invalid notation (HTTP 400) in DiceControllerTests

### Implementation for User Story 1

- [ ] T030 [P] [US1] Create `DiceExpression.cs` model in `src/AdventureGame.Core/Models/DiceExpression.cs`
- [ ] T031 [P] [US1] Create `DiceGroup.cs` model in `src/AdventureGame.Core/Models/DiceGroup.cs` (Count, Sides)
- [ ] T032 [P] [US1] Create `RollResult.cs` model in `src/AdventureGame.Core/Models/RollResult.cs` (GroupResults, Modifiers, Total, Timestamp)
- [ ] T033 [P] [US1] Create `GroupRollResult.cs` model in `src/AdventureGame.Core/Models/GroupRollResult.cs` (Rolls, Subtotal)
- [ ] T034 [US1] Implement `RegexDiceParser.cs` in `src/AdventureGame.Infrastructure/Dice/RegexDiceParser.cs` (depends on T032, T031)
  - Regex pattern: `^(\d+)d(\d+)(([+\-]\d+))*$` with modifications for complex expressions
  - Validation: 1-20 dice, 1-20 sides per group
  - Return: Parsed DiceExpression or throw InvalidNotationException
- [ ] T035 [US1] Implement `CryptoDiceRoller.cs` in `src/AdventureGame.Infrastructure/Dice/CryptoDiceRoller.cs`
  - Use System.Security.Cryptography.RandomNumberGenerator
  - Generate individual die rolls with uniform distribution
  - Return: RollResult with all individual rolls and total
- [ ] T036 [US1] Implement `DiceService.cs` in `src/AdventureGame.Core/Services/DiceService.cs` (depends on T034, T035)
  - Orchestrate parsing and rolling
  - Validate parsed expression (resource limits: max 20 dice total per expression)
  - Call roller with validated expression
  - Return complete RollResult with timestamp
- [ ] T037 [US1] Create request/response DTOs in `src/AdventureGame.Api/Models/`
  - `RollRequest.cs` (expression, mode fields)
  - `RollResponse.cs` (maps RollResult to JSON)
  - `GroupRollResultDto.cs` (DTO for GroupRollResult)
- [ ] T038 [US1] Create `DiceController.cs` in `src/AdventureGame.Api/Controllers/DiceController.cs`
  - POST /api/roll endpoint
  - Validate request JSON
  - Call DiceService
  - Return RollResponse (HTTP 200) or ErrorResponse (HTTP 400 for validation, 500 for unexpected)
- [ ] T039 [US1] Add validation for resource limits: max 20 dice per group, max 20 sides per die
  - Add check in DiceService after parsing
  - Throw ResourceLimitExceededException → caught by middleware → HTTP 400
- [ ] T040 [US1] Register DiceService and dependencies in Program.cs DI container
  - AddScoped<IDiceParser, RegexDiceParser>()
  - AddScoped<IDiceRoller, CryptoDiceRoller>()
  - AddScoped<DiceService>()
- [ ] T041 [US1] Generate OpenAPI documentation for /api/roll endpoint
  - [Produces("application/json")] and [Consumes("application/json")] attributes
  - [ProducesResponseType(200)] with RollResponse schema
  - [ProducesResponseType(400)] with ErrorResponse schema
  - Update contracts/openapi.yaml with generated spec

**Checkpoint**: User Story 1 complete - basic dice rolling (2d6, 1d20+5) fully functional, independently testable

---

## Phase 4: User Story 2 - Complex Expression Parsing (Priority: P2)

**Goal**: Users can roll complex expressions with multiple dice groups (2d6+1d4+3, 1d20+2d6-1) with proper parsing and evaluation  
**Independent Test**: Can be tested via HTTP POST to /api/roll with complex expressions  
**Dependency**: Requires US1 foundation (DiceService, infrastructure)  
**Estimated Effort**: 25-30% of total implementation

### Tests for User Story 2 (REQUIRED - TDD)

> **Write these FIRST, ensure they FAIL before implementation**

- [ ] T042 [P] [US2] Add parameterized tests for multi-group notation (2d6+1d4, 1d8+2d6+1d4) in DiceParserTests
- [ ] T043 [P] [US2] Add parameterized tests for mixed positive/negative modifiers (2d6+1d4-2) in DiceParserTests
- [ ] T044 [P] [US2] Add parameterized tests for whitespace handling (2d6 + 1d4 + 3) in DiceParserTests
- [ ] T045 [P] [US2] Add tests for invalid complex expressions (2d6++, 1d4+, +2d6) in DiceParserTests
- [ ] T046 [P] [US2] Add tests for maximum total dice limit (multiple groups totaling >20 dice) in DiceParserTests
- [ ] T047 [P] [US2] Create integration test for POST /api/roll with expression "2d6+1d4+3" in DiceControllerTests
- [ ] T048 [P] [US2] Add integration test for POST /api/roll with expression "1d20+2d6-1" in DiceControllerTests
- [ ] T049 [P] [US2] Add integration test for POST /api/roll with whitespace in expression in DiceControllerTests

### Implementation for User Story 2

- [ ] T050 [US2] Enhance `RegexDiceParser.cs` to support multiple dice groups
  - Update regex pattern to allow: `^(\d+)d(\d+)(([+\-])(\d+|(\d+)d(\d+)))*$`
  - Parse multiple DiceGroup objects from single expression
  - Extract all modifiers (constants) separately
  - Validate total dice across all groups <= 20
  - Return: DiceExpression with multiple groups and modifiers
- [ ] T051 [US2] Enhance `CryptoDiceRoller.cs` to handle multiple DiceGroup objects
  - Iterate through each DiceGroup
  - Roll each group independently
  - Create separate GroupRollResult for each
  - Sum all rolls + all modifiers for total
  - Return: RollResult with array of GroupRollResult objects
- [ ] T052 [US2] Enhance `DiceService.cs` validation
  - Check total dice across ALL groups: sum(group.Count) <= 20
  - Individual group limits: group.Sides <= 20
  - Throw ResourceLimitExceededException with details if exceeded
- [ ] T053 [P] [US2] Update `RollResponse.cs` to handle multiple GroupRollResult objects
  - groupResults: List<GroupRollResultDto>
  - Ensure JSON serialization correctly represents all groups
- [ ] T054 [US2] Add whitespace handling in DiceService before passing to parser
  - Strip spaces: expression.Replace(" ", "")
  - Validate no trailing/leading special chars
- [ ] T055 [P] [US2] Add test for resource limit validation in DiceService (total dice across groups)
  - Expression: "15d6+10d6" (25 dice total) → ResourceLimitExceededException
- [ ] T056 [US2] Run all US1 tests to ensure backward compatibility maintained
  - All basic notation tests still pass
  - All single-group scenarios still work

**Checkpoint**: User Story 2 complete - complex multi-dice expressions fully functional, independently testable

---

## Phase 5: User Story 3 - Advantage/Disadvantage (Priority: P3)

**Goal**: Users can roll with advantage (roll twice, take higher) or disadvantage (roll twice, take lower) for single-die expressions  
**Independent Test**: Can be tested via HTTP POST to /api/roll with mode parameter  
**Dependency**: Requires US1 & US2 foundation (DiceService, infrastructure)  
**Estimated Effort**: 20-25% of total implementation

### Tests for User Story 3 (REQUIRED - TDD)

> **Write these FIRST, ensure they FAIL before implementation**

- [ ] T057 [P] [US3] Add test for advantage mode with single die (1d20 with mode=advantage) in DiceRollerTests
- [ ] T058 [P] [US3] Add test for disadvantage mode with single die (1d20 with mode=disadvantage) in DiceRollerTests
- [ ] T059 [P] [US3] Add test that advantage produces higher value between two rolls in DiceRollerTests
- [ ] T060 [P] [US3] Add test that disadvantage produces lower value between two rolls in DiceRollerTests
- [ ] T061 [P] [US3] Add test that advantage/disadvantage with modifier applies modifier to selected roll in DiceRollerTests
- [ ] T062 [P] [US3] Add test that advantage/disadvantage on multi-group expression returns HTTP 400 in DiceControllerTests
- [ ] T063 [P] [US3] Add test that invalid mode value ("invalid") returns HTTP 400 in DiceControllerTests
- [ ] T064 [P] [US3] Add parameterized test for advantage mode with various die sizes (d4, d6, d12, d20) in DiceRollerTests
- [ ] T065 [P] [US3] Create integration test for POST /api/roll with mode=advantage in DiceControllerTests
- [ ] T066 [P] [US3] Create integration test for POST /api/roll with mode=disadvantage in DiceControllerTests

### Implementation for User Story 3

- [ ] T067 [P] [US3] Update `RollMode.cs` enum (already created in T010)
  - Values: Normal, Advantage, Disadvantage
  - Add validation method: IsValidForExpression(expression)
- [ ] T068 [US3] Update `RollRequest.cs` to include mode field (already created, just verify)
  - mode: string (default "normal")
  - Valid values: normal, advantage, disadvantage
  - Add validation in controller
- [ ] T069 [US3] Update `RollResult.cs` model
  - Add Mode property (RollMode)
  - Include selected rolls separately from all rolls for advantage/disadvantage
- [ ] T070 [P] [US3] Update `GroupRollResult.cs` to track all rolls for advantage/disadvantage
  - If mode is Normal: rolls contain single roll per die
  - If mode is Advantage: rolls contain TWO rolls per die (both values, then selected)
  - If mode is Disadvantage: rolls contain TWO rolls per die (both values, then selected)
- [ ] T071 [US3] Enhance `CryptoDiceRoller.cs` to implement advantage/disadvantage logic
  - Accept RollMode parameter
  - If Normal: roll once per die
  - If Advantage: roll twice per die, select higher value
  - If Disadvantage: roll twice per die, select lower value
  - Return: RollResult with appropriate rolls and selected subtotal
- [ ] T072 [US3] Add validation in `DiceService.cs` for mode applicability
  - Advantage/Disadvantage ONLY valid if expression has exactly ONE dice group with ONE die
  - Examples:
    - Valid: 1d20 with advantage ✓
    - Valid: 1d20+5 with disadvantage ✓ (one die with modifier)
    - Invalid: 2d6 with advantage ✗ (would reroll to 4 dice)
    - Invalid: 1d20+1d4 with advantage ✗ (two groups)
  - Throw exception → HTTP 400 if invalid
- [ ] T073 [P] [US3] Update `DiceController.cs` to validate and pass mode to DiceService
  - Extract mode from RollRequest
  - Validate: must be null or one of {normal, advantage, disadvantage}
  - Default to normal if not provided
  - Pass to DiceService as RollMode enum
- [ ] T074 [P] [US3] Update error handling in middleware
  - New error code: "INVALID_MODE" for advantage/disadvantage on invalid expressions
  - Error message: "Advantage/disadvantage only applies to single die rolls"
  - Include details: which expression was invalid
- [ ] T075 [US3] [REQUIRED - SC-003] Add parametric statistical test for distribution validation
  - Roll 1d20 with advantage 1000 times → verify distribution skews toward higher values
  - Roll 1d20 with disadvantage 1000 times → verify distribution skews toward lower values
  - Use chi-squared test or visual inspection of percentile graphs
  - **Rationale**: SC-003 is a mandatory success criterion requiring statistical validation; this is NOT optional
- [ ] T076 [US3] Update OpenAPI spec for mode parameter and advantage/disadvantage responses
  - Document mode field in RollRequest schema
  - Show example requests with mode=advantage in contracts/openapi.yaml
  - Ensure RollResponse shows both rolls (if applicable) in UI documentation

**Checkpoint**: User Story 3 complete - advantage/disadvantage mechanics fully functional

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final quality improvements, documentation, performance optimization

- [ ] T077 [P] Run full test suite and verify all tests pass
  - `dotnet test` across all projects
  - Target: 100% pass rate
- [ ] T078 [P] [P] Add API versioning header in responses (X-API-Version: 1.0)
- [ ] T079 [P] Add request correlation ID for logging/debugging in middleware
- [ ] T080 Add comprehensive API documentation comments (///  XML comments) to DiceController
- [ ] T081 Create `packages/api/README.md` with setup, build, test, and deployment instructions
- [ ] T082 [P] Add performance benchmarks in `tests/AdventureGame.Infrastructure.Tests/PerformanceBenchmarks.cs`
  - Verify <10ms per roll for up to 20 dice
  - Measure regex compilation overhead
  - Verify crypto RNG <0.1ms per die
- [ ] T083 [P] Add logging to DiceService and DiceController
  - Log successful rolls (non-PII: expression only)
  - Log validation errors (non-PII: error code only)
  - Use ILogger<T> injected via DI
- [ ] T084 Add SonarQube/code quality configuration (optional)
  - .sonarqube.yml file
  - Code coverage thresholds
- [ ] T085 Create deployment documentation: Docker, GitHub Actions, deployment to Azure (for Lab 3)
- [ ] T086 [P] [P] Final verification against spec acceptance scenarios
  - Run all acceptance tests from spec.md manually
  - Document any deviations
- [ ] T086b [P] [REQUIRED - SC-007] Implement concurrent load testing
  - Simulate 1000 concurrent POST /api/roll requests
  - Measure p95 and p99 latency under load (target: <200ms)
  - Verify no thread pool exhaustion or connection timeouts
  - Document: baseline (single-request <10ms) vs degradation threshold
- [ ] T087 Commit all code to feature branch `001-dice-engine` with meaningful commit messages
- [ ] T088 Create pull request with summary of implementation
- [ ] T089 Run `/speckit.checklist` to validate implementation against spec

---

## Dependencies & Execution Order

### Understanding the Task Graph

```
Phase 1: Setup (T001-T006)
         ↓
Phase 2: Foundational (T007-T014)
         ↓ [Blocks all user stories]
┌─────────┴─────────┬─────────────┐
↓                   ↓             ↓
Phase 3: US1        Phase 4: US2  Phase 5: US3
(T015-T041)         (T042-T056)   (T057-T076)
         ↓                   
    [Independent Test]       
         ↓                   
Phase 6: Polish (T077-T089)
```

### Parallel Opportunity: Once Foundation Complete

After Phase 2 completes, you CAN work on user stories in parallel:
- Developer A: Implement US1 (Phase 3) while...
- Developer B: Write tests for US2 (Phase 4 tests)

However, **no user story implementation starts before Phase 2 completes**.

### Sequential Constraint Within Each Phase

Within each phase, some tasks have dependencies marked with **"depends on"**:

**Phase 3 Example**:
- T030-T033 (models) can run in parallel with T015-T029 (tests)
- T034-T035 (infrastructure) can start immediately after dependencies
- T036 (DiceService) depends on T034 + T035 completion
- T037-T041 (API layer) can start after T034-T036

### Recommended MVP Path (Minimal Viable Product)

Complete in this order to get a working MVP:
1. Phase 1: Setup (T001-T006)
2. Phase 2: Foundational (T007-T014)
3. Phase 3: User Story 1 (T015-T041)
4. Phase 6 Polish: Testing + Documentation (T077-T089)

**Result**: Minimal working dice engine (basic notation only)

Then expand:
5. Phase 4: User Story 2 (T042-T056)
6. Phase 5: User Story 3 (T057-T076)

---

## Quality Gates

### Per-Phase Checkpoints

- **After Phase 2**: All foundation tests pass ✓
- **After Phase 3**: US1 fully testable independently ✓
- **After Phase 4**: US2 fully testable independently ✓
- **After Phase 5**: US3 fully testable independently ✓
- **After Phase 6**: All tests pass, documentation complete, performance verified ✓

### Constitutional Requirements (From constitution.md)

- [ ] ✅ **RESTful design**: POST /api/roll follows REST conventions
- [ ] ✅ **Documentation**: OpenAPI 3.0.1 spec generated (T041)
- [ ] ✅ **Testability**: Unit tests for all layers (T015+, T042+, T057+)
- [ ] ✅ **Simplicity**: Clean Architecture justified (monorepo, no unnecessary abstractions)
- [ ] ✅ **Performance**: <10ms per roll verified (T082)

---

## Testing Strategy

### Test-First (TDD) Workflow

For each user story:

1. **Write Tests First** (marked as REQUIRED ⚠️)
   - Unit tests for domain logic (Core layer)
   - Integration tests for API layer
   - Tests should FAIL initially
   - Example: T015-T029 for US1

2. **Implement Feature**
   - Make tests pass
   - Refactor if needed
   - Verify no regressions

3. **Verify Coverage**
   - Core layer: 100% (business logic)
   - Infrastructure: 95% (implementations)
   - API: 85% (controllers)

### Running Tests

```bash
# All tests
dotnet test

# Specific test class
dotnet test --filter NameSpace.DiceParserTests

# With coverage
dotnet test /p:CollectCoverage=true
```

---

## Notes & Best Practices

### Validation & Error Handling

- **Invalid Notation**: Return HTTP 400 with INVALID_NOTATION code
- **Resource Limit**: Return HTTP 400 with RESOURCE_LIMIT_EXCEEDED code
- **Invalid Mode**: Return HTTP 400 with INVALID_MODE code
- **Server Error**: Return HTTP 500 with INTERNAL_ERROR code

### Performance Promises (From constitution)

- Dice rolls: <10ms per rolling operation
- Total p95 request latency: <200ms (including validation, serialization)
- Resource limits: Max 20 dice, max 20 sides

### Future Extensibility

This task breakdown prepares for future Labs:
- **Lab 2 (Frontend)**: API will integrate with frontend UI calling POST /api/roll
- **Lab 3 (Infrastructure)**: Containerize and deploy to Azure

---

## Summary

| Phase | Effort | Tasks | Blocking | Dependency |
|-------|--------|-------|----------|-----------|
| 1: Setup | Small | 6 | None | - |
| 2: Foundation | Medium | 8 | All stories | Phase 1 |
| 3: US1 P1 | Large | 27 | US2, US3 | Phase 2 |
| 4: US2 P2 | Medium | 15 | US3 | US1 complete |
| 5: US3 P3 | Medium | 20 | - | US2 complete |
| 6: Polish | Small | 14 | - | All phases |
| **TOTAL** | **Large** | **90** | - | **~40-65 hours** |

**MVP Scope** (recommended for immediate delivery): Phases 1-3 = ~25-30 hours for working dice engine with basic notation support.
