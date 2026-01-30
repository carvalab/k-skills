# DRY Detection & Code Reuse

**MANDATORY: Read this before creating new files or functions.**

## The Core Problem

> "Duplication is the root of all evil in software." — The Pragmatic Programmer

DRY is NOT about avoiding copy-paste. DRY means: **Every piece of KNOWLEDGE has ONE authoritative source.**

When AI generates code, it often can't "see" all relevant files, leading to accidental duplication. Your job is to be the bridge - find existing code and ensure reuse.

## The Cost of Duplication

| Problem | Impact |
|---------|--------|
| Same logic in 2+ places | Bug fix needed in N places |
| Copied calculations | Results diverge over time |
| Duplicated business rules | Inconsistent behavior |
| Similar mappers/transformers | Maintenance nightmare |

## 🔴 Pre-Implementation GATE (Cannot Skip)

**Before writing ANY new code, complete this checklist:**

- [ ] Ran comprehensive search (verbs, nouns, types)?
- [ ] Read ALL potentially relevant files found?
- [ ] Documented reuse analysis with similarity %?
- [ ] Applied decision matrix (REUSE/EXTEND/EXTRACT/NEW)?
- [ ] If >50% similar found → refactoring plan created?

## How to Detect Hidden Duplication

### 1. Search by Function Purpose

```bash
# What you're building: "Map X to Event"
grep -rn "Map.*To.*Event\|Convert.*To.*Event\|Build.*Event" --include="*.go" --include="*.ts" .

# What you're building: "Calculate differences"
grep -rn "calculate.*Diff\|compute.*Difference\|diff.*Offers" --include="*.go" --include="*.ts" .

# What you're building: "Send to external API"
grep -rn "Send.*API\|Post.*Event\|Publish.*Event" --include="*.go" --include="*.ts" .
```

### 2. Search by Data Types

```bash
# If you're working with "GrowthPulseInput"
grep -rn "GrowthPulseInput\|PriceAnchoringEvent" --include="*.go" --include="*.ts" .

# If you're working with simulations/quotations
grep -rn "QuotationSimulation\|SimulationModel" --include="*.go" --include="*.ts" .
```

### 3. Search by Domain Concepts

```bash
# Search broad domain terms
grep -rn "pulse\|quotation\|event.*tracker\|repricing" --include="*.go" . | head -50
```

## Signs You're About to Duplicate

| Red Flag                                               | What It Means                                |
| ------------------------------------------------------ | -------------------------------------------- |
| "I need a function similar to X but for Y"             | Extract common logic into shared helper      |
| "This struct has the same fields as that one"          | Use composition or interface                 |
| "The calculation is the same, just different input"    | Create adapter/transformer                   |
| "New file looks 80% like existing file"                | **STOP** - refactor to share code            |

## Refactoring Patterns to Avoid Duplication

### Pattern 1: Extract Shared Helper

**Before (duplicated):**

```go
// file1.go - 100 lines of offer difference calculations
func MapModelAToEvent(a *ModelA) *Event {
    // ... 80 lines of calculations ...
    diff := newest.Offer - latest.Offer
    // ... more calculations ...
}

// file2.go - 100 lines of SAME offer difference calculations
func MapModelBToEvent(b *ModelB) *Event {
    // ... same 80 lines of calculations ...
    diff := newest.Offer - latest.Offer
    // ... same calculations ...
}
```

**After (shared helper):**

```go
// shared.go - calculations in ONE place
type OfferData struct {
    InstantOffer    float64
    ConsignmentOffer float64
    // ... common fields
}

func CalculateOfferDifferences(latest, newest *OfferData) *Differences {
    return &Differences{
        InstantDiff: newest.InstantOffer - latest.InstantOffer,
        // ... all calculations in ONE place
    }
}

// file1.go - just extraction + shared call
func MapModelAToEvent(a *ModelA) *Event {
    offers := extractOffersFromA(a) // 10 lines
    diffs := CalculateOfferDifferences(offers.Previous, offers.Current)
    return buildEvent(a.UserID, diffs) // 10 lines
}

// file2.go - just extraction + shared call
func MapModelBToEvent(b *ModelB) *Event {
    offers := extractOffersFromB(b) // 10 lines
    diffs := CalculateOfferDifferences(offers.Previous, offers.Current)
    return buildEvent(b.UserID, diffs) // 10 lines
}
```

**Result:** 200 lines → 50 lines. Calculations in ONE place.

### Pattern 2: Interface Adapter

**Before (duplicated):**

```go
func ProcessAudienceModel(a *AudienceModel) { /* 100 lines */ }
func ProcessMembershipModel(m *MembershipModel) { /* same 100 lines */ }
```

**After (interface):**

```go
type EventSource interface {
    GetUserUUID() string
    GetSKU() string
    GetOffers() []Offer
}

// Make both models implement the interface
func (a *AudienceModel) GetUserUUID() string { return a.User.UUID }
func (m *MembershipModel) GetUserUUID() string { return m.Member.User.UUID }

// ONE function handles both
func ProcessEventSource(src EventSource) { /* 100 lines, once */ }
```

### Pattern 3: Composition Over Copy

**Before:**

```go
type EventContentA struct {
    UserID string
    SKU    string
    // ... 50 common fields ...
    SpecificFieldA string
}

type EventContentB struct {
    UserID string
    SKU    string
    // ... same 50 fields copied ...
    SpecificFieldB string
}
```

**After:**

```go
type CommonEventContent struct {
    UserID string
    SKU    string
    // ... 50 fields in ONE place ...
}

type EventContentA struct {
    CommonEventContent
    SpecificFieldA string
}

type EventContentB struct {
    CommonEventContent
    SpecificFieldB string
}
```

## Decision Tree

```
Found similar existing code?
│
├─ NO → Proceed with new implementation
│
└─ YES → How similar?
         │
         ├─ <30% similar → Proceed (different enough)
         │
         ├─ 30-50% similar → Consider extracting common parts
         │
         └─ >50% similar → MUST refactor
                          │
                          ├─ Same calculation logic? → Extract helper function
                          ├─ Same struct fields? → Use composition
                          ├─ Same method signatures? → Create interface
                          └─ Same overall flow? → Template method pattern
```

## Code Review Checklist for DRY

When reviewing your own code before committing:

- [ ] Did I create a new file >100 lines? Search for similar files.
- [ ] Did I copy-paste ANY code? Extract it.
- [ ] Does my new function look like an existing one? Refactor to share.
- [ ] Are there calculations that appear twice? Extract helper.
- [ ] Could this logic change? Is it in ONE place?

## Examples of DRY Violations to Avoid

### ❌ BAD: New mapper that duplicates existing

```go
// WRONG: 364 lines that duplicate 90% of existing mapper.go
func MapPulseMembershipToEvent(...) (*GrowthPulseInput, error) {
    // Same calculations as MapModelsToEvent
    // Same struct population
    // Same helper calls
    // Only different: input model type
}
```

### ✅ GOOD: Extend existing with minimal new code

```go
// RIGHT: Extract common logic, add new entry point
func MapToEvent(src EventSource, prev, curr *Simulation) (*GrowthPulseInput, error) {
    // Shared calculation logic
    diffs := calculateOfferDifferences(prev, curr)
    return &GrowthPulseInput{
        UserUUID: src.GetUserUUID(),
        // ... populate from interface
    }, nil
}

// New entry point - just adapts input type
func MapPulseMembershipToEvent(pm *PulseMembershipModel, ...) (*GrowthPulseInput, error) {
    return MapToEvent(pm, prev, curr) // pm implements EventSource
}
```

## Summary

| Rule                               | Action                        |
| ---------------------------------- | ----------------------------- |
| **Before creating**                | Search for similar code       |
| **If >50% similar**                | MUST refactor to share        |
| **Calculations**                   | Extract to ONE helper         |
| **Struct fields**                  | Use composition               |
| **Multiple sources, same logic**   | Use interfaces                |
| **Every change**                   | Ask: "Is this in ONE place?"  |

**The goal:** Any business logic change requires editing ONE file, not N files.
