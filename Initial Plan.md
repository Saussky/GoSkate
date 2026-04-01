# GoSkate Swift Master Plan

## Summary

- Build `GoSkate` as an `iPhone-only`, `iOS 17+` app in `SwiftUI` with `SwiftData` for local-first persistence.
- Ship v1 with `3 sesh types`: `attempt-based`, `timed`, and `consistency`, all powered by one shared sesh engine.
- Treat `attempt records` as the source of truth. All summaries, streaks, success rates, and PBs are derived from those records.
- Store qualitative data as `cleanliness` only, on a `1-6` scale, captured for landed attempts. Remove `confidence` entirely from the product, domain, persistence, UI, and analytics plan.
- Use this document as the master delivery plan. Each phase is intended to become an independently assigned agent task with a clear boundary, deliverable, and acceptance gate.

## Core Product And Technical Decisions

- `Platform:` iPhone only, portrait-first, offline-first, no watchOS, iPad, or macOS in v1.
- `UI stack:` SwiftUI with `Observation`; no business logic in views.
- `Persistence:` SwiftData from the start. No cloud sync in v1.
- `Session model:` one active sesh at a time; restore on relaunch.
- `Trick model:` one trick per sesh; ship with seeded trick data and allow user-created tricks.
- `Qualitative model:` only `cleanliness 1...6`; required for landed attempts, absent for misses.
- `History model:` lists and aggregate summaries only; no charts in v1.
- `PB rules:` compare like-for-like sessions only:
  - attempt-based: same trick and same `maxAttempts`
  - timed: same trick and same `durationSeconds`
  - consistency: same trick and same sesh kind
- `Editing support:` undo last attempt, edit attempt, delete attempt, then recompute all derived values.
- `Architecture style:` layered app with `App`, `Features`, `Domain`, `Data`, and `Shared` folders; do not split into packages in v1.

## Public Types And Interfaces

- `enum SeshKind { attemptBased, timed, consistency }`
- `enum SeshRules { case attemptBased(maxAttempts: Int, targetLands: Int?), case timed(durationSeconds: Int, targetLands: Int?), case consistency }`
- `enum CleanlinessRating: Int, CaseIterable` with valid values `1...6`
- `enum AttemptOutcome { case landed(cleanliness: CleanlinessRating, note: String?), case missed(note: String?) }`
- `struct SeshDefinition` with `trickId`, `kind`, `rules`, `title?`, `notes?`
- `struct SeshState` with live counters, streaks, timer state, completion state, and progress state
- `struct SeshSummary` with attempts, lands, misses, success rate, best streak, first land attempt, duration, attempts per minute, lands per minute, average cleanliness, PB flags
- `enum SeshAction` with `start`, `recordAttempt`, `pause`, `resume`, `undo`, `editAttempt`, `deleteAttempt`, `finish`
- `protocol SeshRepository`
- `protocol TrickRepository`
- `protocol SeshMetricCalculating`
- `ActiveSeshStore` as the live observable state owner
- `SeshEngine` as a pure state machine / reducer responsible for validating actions, applying state changes, and deriving completion and progress

## Phase Plan

### Phase 1: Product Skeleton And Project Foundation

- `Goal:` create the app shell and lock the foundational architecture so later agents work against stable boundaries.
- `Scope:`
  - create the Xcode project and app target
  - set minimum deployment to `iOS 17`
  - establish folder structure and naming conventions
  - create the root navigation shell
  - define app-wide design tokens for spacing, typography, color, and control sizing
  - wire SwiftData container into app startup
  - seed a starter trick catalog on first launch
- `Deliverables:`
  - compilable app that launches into a placeholder `Home`, `History`, and `Tricks` tab structure
  - SwiftData stack initialized and testable
  - seed path for built-in tricks
  - architecture README or inline code conventions documenting layer responsibilities
- `Agent boundary:`
  - may create app shell, data container bootstrapping, navigation scaffolding, and seed logic
  - must not implement sesh engine logic, attempt logging, summaries, or history features
- `Dependencies:` none
- `Acceptance criteria:`
  - app launches cleanly
  - tabs render
  - seeded tricks appear locally
  - no domain logic is embedded in views
  - project structure clearly separates `Features`, `Domain`, and `Data`

### Phase 2: Domain Model And Sesh Engine

- `Goal:` implement the core product logic once, independent of UI.
- `Scope:`
  - define all domain types for seshes, attempts, tricks, summaries, and rules
  - implement `SeshEngine`
  - implement metric calculators for all 3 sesh types
  - enforce rating validation and rule validation
  - define completion rules and progress semantics
  - define persistence-facing models needed by the engine
- `Deliverables:`
  - pure domain layer with no SwiftUI dependency
  - deterministic engine API that can be driven by tests
  - summary calculation logic for attempt-based, timed, and consistency seshes
- `Agent boundary:`
  - owns all domain entities, reducers/state transitions, calculators, and validation rules
  - must not build screens or navigation
- `Dependencies:` Phase 1 folder structure
- `Acceptance criteria:`
  - engine supports start, record land, record miss, undo, edit, delete, pause, resume, finish
  - attempt-based seshes stop at `maxAttempts`
  - timed seshes enforce countdown and completion
  - consistency seshes track streaks correctly
  - average cleanliness is derived from landed attempts only
  - all logic is covered by unit tests and runs without UI

### Phase 3: Persistence Model And Repository Layer

- `Goal:` make the domain durable and queryable locally.
- `Scope:`
  - define SwiftData models for seshes, attempts, tricks, and summary snapshots
  - map between SwiftData records and domain entities
  - implement repositories for active sesh persistence, sesh history, and trick access
  - persist one active sesh and support restore after relaunch
  - define summary snapshot update strategy after every mutation
- `Deliverables:`
  - repository implementations backing the domain layer
  - reliable save/load/edit/delete flows
  - restore logic for in-progress seshes
- `Agent boundary:`
  - owns persistence schema and repository implementations
  - must not build feature UIs except minimal harnesses needed for verification
- `Dependencies:` Phase 2 public types and engine contracts
- `Acceptance criteria:`
  - a sesh can be created, appended to, resumed, finished, edited, and deleted through repositories
  - app relaunch restores the active sesh correctly
  - history queries by trick, sesh type, and date range return correct summaries
  - persisted summary snapshots remain consistent with raw attempt data

### Phase 4: New Sesh Flow And Attempt-Based Logging

- `Goal:` deliver the first end-to-end usable vertical slice.
- `Scope:`
  - build `Home` actions and `New Sesh` flow
  - allow user to choose trick, choose sesh type, and configure attempt-based rules
  - build active attempt-based logging screen
  - implement one-tap miss logging
  - implement landed-attempt flow requiring `cleanliness 1-6`
  - show live counts, progress, streak, and finish state
- `Deliverables:`
  - fully usable attempt-based sesh flow from creation through summary
  - fast active sesh UX optimized for skating conditions
- `Agent boundary:`
  - owns attempt-based UI flow only
  - may use existing engine and repositories but must not alter domain contracts unless blocked
- `Dependencies:` Phases 2 and 3
- `Acceptance criteria:`
  - user can create an attempt-based sesh and complete it without developer tooling
  - `Miss` is one tap
  - `Land` requires cleanliness before save
  - summary matches raw attempts exactly
  - undo works from the active screen

### Phase 5: Timed And Consistency Active Sesh Flows

- `Goal:` complete the shared engine’s initial product surface by adding the other 2 sesh types.
- `Scope:`
  - build timed sesh configuration and active UI
  - build consistency sesh configuration and active UI
  - implement countdown, pause, resume, timeout completion, and timer display
  - adapt the shared active screen model to different rule sets without duplicating engine logic
- `Deliverables:`
  - timed and consistency seshes fully operational
  - one cohesive active sesh presentation model across all sesh kinds
- `Agent boundary:`
  - owns timed and consistency feature UIs and their presentation logic
  - must reuse existing engine and repositories; no parallel engine rewrite
- `Dependencies:` Phase 4 patterns and all prior phases
- `Acceptance criteria:`
  - timed sesh auto-finishes at zero
  - pause/resume is stable across app background/foreground transitions
  - consistency sesh shows current streak and best streak live
  - all three sesh types behave consistently in creation, logging, finish, and summary flows

### Phase 6: Editing, Recovery, And Error Correction

- `Goal:` make the app usable in real-world skating conditions where mis-taps are common.
- `Scope:`
  - build edit attempt UI
  - build delete attempt flow
  - implement undo last event across all sesh types
  - support correction of cleanliness ratings on landed attempts
  - ensure summaries and PBs recompute after every mutation
- `Deliverables:`
  - robust correction workflow for active and completed seshes
  - safe recalculation behavior after any attempt mutation
- `Agent boundary:`
  - owns correction workflows and recalculation integration
  - must not redesign summary logic or PB rules outside accepted contracts
- `Dependencies:` prior phases complete
- `Acceptance criteria:`
  - user can change a miss to a land and vice versa
  - user can change cleanliness on a landed attempt
  - deleting attempts updates streaks, success rate, and averages correctly
  - undo never leaves the sesh in an invalid state

### Phase 7: Summary, History, And Trick Progress

- `Goal:` turn recorded sessions into useful feedback and review.
- `Scope:`
  - build sesh summary screen
  - build history list with filters by date, trick, and sesh type
  - build trick detail screen with recent seshes and aggregate stats
  - compute and display personal bests using like-for-like comparison rules
  - expose average cleanliness in summaries and trick history
- `Deliverables:`
  - browsable history and trick-centric progress views
  - PB labeling and key aggregate metrics
- `Agent boundary:`
  - owns read-only review surfaces and filtering UX
  - must consume repository/query interfaces rather than recalculate ad hoc in views
- `Dependencies:` persistence and full sesh logging
- `Acceptance criteria:`
  - completed seshes appear in history immediately
  - filters return correct results
  - trick view shows lifetime totals and recent sesh performance
  - PB badges only appear when the comparison rule is valid

### Phase 8: Polish, Accessibility, And Release Hardening

- `Goal:` make the app reliable enough for daily use.
- `Scope:`
  - add haptics for key logging actions
  - refine button sizing, spacing, and contrast for outdoor use
  - add VoiceOver labels and Dynamic Type checks
  - validate empty states, interrupted sessions, and persistence failure handling
  - profile the most active screens for lag or excessive re-rendering
- `Deliverables:`
  - polished v1 candidate build
  - release checklist and known-issues list
- `Agent boundary:`
  - owns UX polish and hardening
  - should not add new product scope
- `Dependencies:` all feature phases complete
- `Acceptance criteria:`
  - logging interactions stay responsive
  - accessibility labels are present on critical controls
  - no obvious state loss on interruption or relaunch
  - all core flows pass smoke testing on device or simulator

## Cross-Phase Rules For Agent Work

- Each phase should be assigned only after its dependencies are accepted.
- Agents must treat earlier public types and interfaces as contracts unless the phase explicitly authorizes contract changes.
- Any contract change proposed by a later phase must be surfaced before implementation because it can invalidate downstream agent work.
- Feature agents may add tests for their phase, but they must not silently widen scope into adjacent phases.
- Repositories, engine logic, and views should remain separate so multiple agents can work without merging conflicting architectural approaches.
- No phase should introduce `confidence`, placeholder confidence fields, or future-proofing hooks for it.

## Test Program

- `Domain tests:` rule enforcement, streak logic, completion logic, average cleanliness, first-land metrics, timer transitions
- `Persistence tests:` create, reload, restore active sesh, edit, delete, query history, update summary snapshots
- `Feature tests:` new sesh setup, one-tap miss, landed attempt rating entry, pause/resume, finish flow, edit flow
- `Acceptance scenarios:`
  - create and finish an attempt-based sesh with 20 attempts
  - create and finish a timed sesh with pause/resume and timeout completion
  - create and finish a consistency sesh with streak resets
  - edit a past attempt and verify summary and PB recalculation
  - relaunch mid-sesh and restore without data loss

## Assumptions

- v1 is intentionally not a custom sesh builder; sesh rules are strongly typed and limited to the 3 supported kinds.
- `Cleanliness` applies only to landed attempts. Misses may carry an optional note only.
- `Freeform`, `challenge builder`, `cloud sync`, `video`, `social`, and `AI insights` are explicitly out of scope for this master plan.
- This plan assumes a single-repo Swift app starting from scratch, not an existing production codebase.
