# GoSkate — Product and Architecture Concept

## Overview

GoSkate is an iPhone app built in **Swift** for recording and reviewing skateboarding practice.

The core unit in the app is a **Sesh**.

A **Sesh** is the parent term for any skateboarding exercise, challenge, drill, or event that records performance. A sesh can track outcomes **quantitatively** or **qualitatively** depending on the format.

Examples:
- A fixed-attempt challenge where the skater gets **10 attempts** and tries to land as many as possible.
- A timed challenge where the skater tries to land as many tricks as possible in **X minutes**.

The product should help users answer questions like:
- Am I getting better over time in a measurable way?
- How much time does it take to learn or improve upon tricks?

But it's main philosophy is that if you measure something, you will get better at the thing you measure.

---

## Core Domain Concept

### Sesh
A **Sesh** is the top-level container for recording a practice event.

A sesh should define:
- **Type** of sesh
- **Rules** of the sesh
- **Trick or trick category** being practiced
- **Recorded outcomes**
- **Summary metrics**

A sesh is not only a timer or attempt counter. It is a structured model for measuring performance in a specific skateboarding context.

---

## Possible Sesh Types

### 1. Attempt-Based Sesh
User gets a fixed number of tries.

Example:
- “Land kickflip 10 times out of 20 attempts.”
- “Best of 10 attempts.”

Useful metrics:
- Total attempts
- Total lands
- Land percentage
- Best streak
- Miss streak
- First land attempt number

### 2. Time-Based Sesh
User tries to land as many as possible within a time window.

Example:
- “5-minute heel flip challenge.”

Useful metrics:
- Session duration
- Attempts per minute
- Lands per minute
- Total lands
- Land percentage
- Best streak during time window

### 3. Repetition / Volume Sesh
User aims to accumulate total makes.

Example:
- “Land 50 ollies.”

Useful metrics:
- Total makes
- Total attempts
- Time to completion
- Breaks or pauses

### 4. Consistency Sesh
User repeats one trick to measure stability and reliability.

Example:
- “How many clean pop shuvits can I land in a row?”

Useful metrics:
- Longest streak
- Average streak length
- Failed-after-land frequency
- Clean vs sketchy landings

### 5. Challenge Sesh
A rule-driven mini game.

Example:
- “Land one trick every 30 seconds.”
- “10 attempts, switch only.”

Useful metrics depend on the challenge rules, which suggests the architecture should support configurable sesh definitions.

---

## Quantitative Tracking Concepts

These are the hard-measurement concepts the app can capture.

### Attempt Metrics
- Attempt count
- Make count
- Bail count
- Retry count
- Success rate
- Time between attempts

### Time Metrics
- Total session duration
- Active time
- Rest time
- Time to first land
- Time to goal completion

### Streak Metrics
- Current streak
- Best streak
- Average streak
- Consecutive misses

### Trick Progress Metrics
- Total lifetime attempts for a trick
- Total lifetime lands for a trick
- Personal bests by sesh type
- Trend over time

### Goal Metrics
- Goal reached or not reached
- Percentage of goal completed
- Distance from best result

---

## Qualitative Tracking Concepts

Skateboarding is not only about counts. A trick can be landed badly, confidently, cleanly, or with poor form.

The app should support subjective inputs such as:
- Cleanliness rating
- Confidence rating
- Difficulty rating
- Notes about technique

---

## Recommended Core Software Concepts

## 1. Domain-Driven Model
The app should be built around explicit domain concepts rather than generic trackers.

Key models:
- **Sesh**
- **SeshType**
- **Trick**
- **Attempt**
- **Outcome**
- **Goal**
- **MetricSummary**
- **QualitativeRating**

This keeps the product language aligned with the skateboarding idea.

## 2. Configurable Sesh Definitions
Rather than hardcoding every sesh format separately, use a system where a sesh type defines:
- input rules
- win conditions
- measurement style
- summary calculations

This allows the app to grow from a few sesh types into many challenges without rewriting the whole architecture.

## 3. Event Recording Model
A sesh can be understood as a timeline of recorded events.

Examples:
- attempt started
- attempt ended
- landed
- failed
- timer started
- timer paused
- note added

This makes it easier to:
- reconstruct what happened
- calculate summaries later
- support undo/editing
- generate history views and analytics

## 4. Derived Metrics
Some values should not be stored directly as the source of truth.

Examples:
- land percentage
- best streak
- average attempts per minute

These should be **derived** from raw session events or attempt records. That avoids inconsistent data and makes recalculation possible if rules change.

## 5. Local-First Mobile Data
Because the app is for skating, users may be outdoors with unreliable connection.

The app should work well offline:
- create seshes offline
- record attempts offline
- calculate summaries on-device
- sync later if cloud features are added

This has strong architectural implications for persistence and sync.

---

## Suggested Data Model

### Sesh
Represents one recorded skateboarding session unit.

Possible fields:
- id
- seshTypeId
- title
- trickId or trickCategoryId
- startedAt
- endedAt
- goalDefinition
- notes
- status

### SeshType
Defines the structure and rules of a sesh.

Possible fields:
- id
- name
- measurementMode
- supportsTimer
- supportsAttemptCounting
- supportsQualitativeRatings
- ruleDefinition

Examples:
- Fixed Attempts
- Timed Challenge
- Consistency Run
- Freeform Session

### Attempt
Represents a single try inside a sesh.

Possible fields:
- id
- seshId
- timestamp
- outcome
- landedCleanly
- note
- confidenceRating

### Trick
Represents a skateboard trick.

Possible fields:
- id
- name
- stance
- category
- difficultyLevel

### GoalDefinition
Represents what the user is trying to achieve.

Examples:
- land 5 of 10
- get highest streak
- land as many as possible in 3 minutes

### MetricSummary
Stores computed values for display and querying.

Possible fields:
- seshId
- attempts
- lands
- successRate
- bestStreak
- durationSeconds
- personalBestFlags

---

## Architectural Implications

## 1. The App Needs a Flexible Session Engine
Because a sesh can represent different game formats, the app needs a reusable engine that can:
- interpret sesh rules
- accept user input events
- update session state
- compute live progress
- finalize results

This suggests separating:
- **domain logic** from
- **UI screens**

The session engine should not live inside SwiftUI views.

## 2. Different Sesh Types Share a Common Shape
Even though sesh types differ, they all still have:
- a start
- a rule set
- tracked events
- a result summary

This suggests a shared protocol or abstraction for sesh behaviors.

For example, each sesh type could define:
- how it starts
- what inputs are valid
- how completion is determined
- how metrics are calculated

## 3. Real-Time Feedback Matters
The user may be actively skating while using the app, so the app should show fast, simple feedback:
- attempts remaining
- current makes
- streak count
- timer countdown
- goal progress

This means the app should maintain an in-memory active sesh state while persisting checkpoints to storage.

## 4. Editing and Corrections Must Be Supported
A user may accidentally tap the wrong result while skating.

The architecture should support:
- undo last event
- edit attempt result
- delete bad input
- recalculate summary after changes

This is another reason raw events plus derived metrics is a strong design.

## 5. Historical Analysis Should Be Easy
Over time, the app should help users see progress trends.

That means the data model should make it easy to query:
- performance by trick
- performance by sesh type
- performance over time
- personal bests
- consistency trends

If this is not considered early, analytics screens become much harder later.

---

## Suggested High-Level iOS Architecture

### Presentation Layer
Built with **SwiftUI**.

Responsibilities:
- render active sesh UI
- render summaries and history
- capture user input
- display progress in real time

### Application / Use Case Layer
Coordinates user actions.

Examples:
- start sesh
- record attempt
- pause sesh
- finish sesh
- edit attempt
- fetch history

This layer keeps UI thin and moves business rules into testable units.

### Domain Layer
Contains the core skate logic.

Examples:
- sesh rules
- scoring
- streak calculation
- completion conditions
- summary calculation

This should be the most stable and reusable part of the codebase.

### Data Layer
Handles storage and retrieval.

Examples:
- local database persistence
- cloud sync later
- user settings
- cached summaries

---

## Suggested Initial Feature Set

### Version 1
- Create a sesh
- Choose sesh type
- Select trick
- Record attempts
- Record simple qualitative note
- View sesh summary
- View trick history

### Version 1.5
- Timers and countdowns
- Streak tracking
- Personal best badges
- Charts for progress
- Filters by trick and sesh type

### Version 2
- Custom sesh builders
- Saved templates
- Cloud sync
- Social sharing
- Video attachment to attempts or seshes
- AI-generated progress insights

---

## Potential Swift Implementation Ideas

### Protocol-Oriented Design
Swift is a strong fit for modeling sesh behaviors using protocols and value types.

Example conceptual roles:
- `SeshDefinition`
- `SeshState`
- `SeshEvent`
- `SeshMetricCalculator`

This would allow each sesh type to plug into a shared system while still keeping unique rule logic.

### State Management
The active sesh likely needs a dedicated observable state object that:
- tracks the current live session
- applies input events
- recalculates summaries
- publishes updates to SwiftUI

### Persistence
A local persistence solution should exist early, even if simple.

Possible directions:
- SwiftData for modern Apple-first persistence
- Core Data if more mature control is needed
- A lightweight custom persistence layer if the early app stays small

For this product idea, a local-first persistence system is important from the start.

---

## Risks and Design Challenges

### 1. Overly Rigid Session Models
If sesh types are hardcoded too tightly, adding new challenge formats later will become expensive.

### 2. Too Much Input Friction
Skaters need quick logging while moving. The UI must be minimal and fast.

### 3. Mixing Source Data with Calculated Data
If the app stores percentages and streak values as primary data without clear recalculation logic, data consistency problems will appear.

### 4. Poor Offline Support
A skating app that depends too much on connectivity will feel unreliable in the real world.

### 5. Weak Domain Language
If the codebase uses generic names like `Session`, `Entry`, or `Record` everywhere, it may lose the product meaning that makes GoSkate distinct. Using `Sesh` as a first-class concept helps keep the code aligned with the product.

---

## Recommended Product Principle

**Treat skate practice as structured play, not just exercise logging.**

That means GoSkate should feel:
- lightweight during use
- flexible in format
- measurable over time
- expressive enough to capture the feel of skating

The best version of the app will combine:
- simple event logging
- meaningful summaries
- progression tracking
- enough flexibility to support many sesh formats

---

## Summary

GoSkate should be built around a strong domain model where **Sesh** is the main product concept.

A sesh is a structured skateboarding practice unit that can represent different challenge formats, including fixed attempts, time trials, consistency drills, and qualitative sessions.

From a software architecture perspective, this implies:
- a flexible rule-driven session system
- clear separation between UI, domain logic, and persistence
- support for both quantitative and qualitative measurements
- derived metric calculation from raw input data
- local-first mobile architecture for reliable outdoor use

This gives the app a clean foundation for both an MVP and future expansion.
