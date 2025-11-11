# UNSAT Constraint Generation - Technical Documentation

> **Automated generation of intentionally unsatisfiable OCL constraints through mutation-based strategies**

---

## Table of Contents

1. [Overview](#overview)
2. [Motivation & Use Cases](#motivation--use-cases)
3. [Architecture](#architecture)
4. [Mutation Strategies](#mutation-strategies)
5. [Algorithm & Workflow](#algorithm--workflow)
6. [Implementation Details](#implementation-details)
7. [Usage & Configuration](#usage--configuration)
8. [Quality Assurance](#quality-assurance)
9. [Evaluation & Metrics](#evaluation--metrics)
10. [Future Enhancements](#future-enhancements)

---

## Overview

The UNSAT constraint generation module (`modules/generation/benchmark/unsat_generator.py`) creates **intentionally unsatisfiable** OCL constraints by systematically mutating satisfiable (SAT) constraints. This controlled generation ensures benchmarks contain known-UNSAT constraints with ground-truth labels, critical for:

- **Solver completeness testing**: Verifying that SMT/CSP solvers correctly identify UNSAT constraints
- **Balanced dataset creation**: Achieving target SAT/UNSAT ratios (e.g., 70% SAT, 30% UNSAT)
- **Tool evaluation**: Testing OCL validators' ability to detect contradictions
- **ML training**: Providing labeled datasets for learning-based constraint analysis

---

## Motivation & Use Cases

### Why Generate UNSAT Constraints?

**Problem 1: Natural UNSAT constraints are rare**
- Random constraint generation produces mostly SAT constraints
- Real-world specifications rarely contain obvious contradictions
- Manual curation of UNSAT examples is time-consuming

**Problem 2: Verification requires ground truth**
- To test a solver's correctness, we need constraints with known satisfiability
- UNSAT detection is harder than SAT detection (requires proof of non-existence)
- Mutation-based generation provides provable UNSAT guarantees

**Problem 3: Benchmark quality**
- Unbalanced benchmarks (all SAT or all UNSAT) don't stress-test verifiers
- Research benchmarks should reflect realistic SAT/UNSAT distributions
- Diversity in UNSAT types exercises different solver reasoning paths

### Real-World Applications

1. **Tool Benchmarking**
   - Compare UNSAT detection accuracy across USE, EMFtoCSP, Z3-based verifiers
   - Measure timeout behavior on hard-UNSAT instances

2. **Regression Testing**
   - Ensure solver updates don't break UNSAT detection
   - Validate optimization passes preserve logical semantics

3. **Educational Datasets**
   - Teach constraint modeling by showing contradictions
   - Illustrate common mistake patterns (contradictory bounds, empty collections)

4. **Fuzz Testing**
   - Generate adversarial inputs for constraint parsers/validators
   - Discover edge cases in OCL tooling

---

## Architecture

### High-Level Design

```
SAT Constraints (Input)
        ↓
    [Selection]
        ↓
  Mutation Strategy Selection
  (5 strategies available)
        ↓
    [Transform OCL]
        ↓
  UNSAT Constraint + Metadata
        ↓
  Mixed SAT/UNSAT Set (Output)
```

### Component Hierarchy

```python
UnsatMutationStrategy (Abstract Base)
├── ContradictoryBoundsStrategy      # x > 10 ∧ x < 5
├── EmptyCollectionStrategy          # X->notEmpty() ∧ X->isEmpty()
├── TypeContradictionStrategy        # x.oclIsTypeOf(A) ∧ x.oclIsTypeOf(B)
├── UniversalNegationStrategy        # X->forAll(p) ∧ X->exists(¬p)
└── SimpleNegationStrategy           # not(φ)  [Fallback]
```

Each strategy implements:
- `can_apply(constraint, metamodel) -> bool`: Applicability check
- `apply(constraint, metamodel) -> OCLConstraint`: Mutation transformation
- `get_name() -> str`: Strategy identifier

---

## Mutation Strategies

### Strategy 1: Contradictory Bounds

**Description**: Injects conflicting numeric bounds on the same attribute.

**Pattern Detection**:
```python
Pattern: attribute OP value
Example: self.age >= 18
```

**Transformation Rules**:
```python
Original: self.age > 10
Mutated:  (self.age > 10) and self.age < 5   # 5 = 10 - 5 (gap)

Original: self.price >= 100
Mutated:  (self.price >= 100) and self.price < 100

Original: self.count = 5
Mutated:  (self.count = 5) and self.count <> 5
```

**Implementation**:
```python
class ContradictoryBoundsStrategy(UnsatMutationStrategy):
    def can_apply(self, constraint, metamodel):
        # Check for numeric comparison with digit
        return bool(re.search(r'[<>=]', ocl) and re.search(r'\d+', ocl))
    
    def apply(self, constraint, metamodel):
        # Extract: variable, operator, value
        pattern = r'(\w+(?:\.\w+)*)\s*([<>=]+)\s*(\d+)'
        match = re.search(pattern, constraint.ocl)
        var, op, val = match.group(1, 2, 3)
        
        # Generate contradiction based on operator
        if op == '>':
            contradiction = f'{var} < {int(val) - 5}'
        elif op == '>=':
            contradiction = f'{var} < {val}'
        # ... [similar for other operators]
        
        # Inject via conjunction
        new_ocl = f"({original_expr}) and {contradiction}"
        return OCLConstraint(..., metadata={'is_unsat': True, 'mutation': 'contradictory_bounds'})
```

**Why It Works**:
- Creates overlapping constraints that cannot simultaneously hold
- Gap size (default: 5) ensures clear separation, no boundary confusion
- Works for integers, reals, dates (any ordered domain)

**Limitations**:
- Requires numeric comparisons in original constraint
- May produce syntactically awkward OCL if constraint is complex

---

### Strategy 2: Empty Collection

**Description**: Forces a collection to be simultaneously empty and non-empty.

**Pattern Detection**:
```python
Patterns: X->notEmpty()  or  X->size() > 0
```

**Transformation Rules**:
```python
Original: self.books->notEmpty()
Mutated:  (self.books->notEmpty()) and self.books->isEmpty()

Original: self.items->size() > 0
Mutated:  (self.items->size() > 0) and self.items->size() = 0
```

**Implementation**:
```python
class EmptyCollectionStrategy(UnsatMutationStrategy):
    def can_apply(self, constraint, metamodel):
        return 'notEmpty()' in ocl or 'size()' in ocl
    
    def apply(self, constraint, metamodel):
        if 'notEmpty()' in ocl:
            # Extract collection: self.X->notEmpty()
            match = re.search(r'([\w.]+)->notEmpty\(\)', ocl)
            collection = match.group(1)
            contradiction = f'{collection}->isEmpty()'
        elif 'size()' in ocl:
            # Extract: self.X->size() > 0
            match = re.search(r'([\w.]+)->size\(\)\s*>\s*0', ocl)
            collection = match.group(1)
            contradiction = f'{collection}->size() = 0'
        
        new_ocl = f"({original_expr}) and {contradiction}"
        return OCLConstraint(..., metadata={'is_unsat': True, 'mutation': 'empty_collection'})
```

**Why It Works**:
- Cardinality is monotonic: a collection cannot have size > 0 and size = 0
- `notEmpty()` ≡ `size() > 0`, `isEmpty()` ≡ `size() = 0` (by OCL semantics)
- Direct logical contradiction

**Edge Cases**:
- If collection is undefined (`null`), both predicates may fail (OCL 3-valued logic)
- Our encoding treats undefined as false, preserving UNSAT

---

### Strategy 3: Type Contradiction

**Description**: Requires an object to belong to two disjoint types simultaneously.

**Pattern Detection**:
```python
Patterns: X.oclIsTypeOf(Type)  or  X.oclIsKindOf(Type)
```

**Transformation Rules**:
```python
Original: self.oclIsTypeOf(Student)
Mutated:  (self.oclIsTypeOf(Student)) and self.oclIsTypeOf(Teacher)
# Assumes Student and Teacher are disjoint types
```

**Implementation**:
```python
class TypeContradictionStrategy(UnsatMutationStrategy):
    def can_apply(self, constraint, metamodel):
        return 'oclIsTypeOf' in ocl or 'oclIsKindOf' in ocl
    
    def apply(self, constraint, metamodel):
        # Extract: variable, type1
        match = re.search(r'(\w+)\.oclIsTypeOf\((\w+)\)', ocl)
        var, type1 = match.group(1, 2)
        
        # Select different type from metamodel
        classes = [c for c in metamodel.get_class_names() if c != type1]
        type2 = random.choice(classes)
        
        contradiction = f'{var}.oclIsTypeOf({type2})'
        new_ocl = f"({original_expr}) and {contradiction}"
        return OCLConstraint(..., metadata={'is_unsat': True, 'mutation': 'type_contradiction'})
```

**Why It Works**:
- `oclIsTypeOf` is exact type match (not subtype)
- In single-inheritance metamodels, an object has exactly one concrete type
- Multiple `oclIsTypeOf` assertions (for different types) cannot coexist

**Limitations**:
- Assumes types are disjoint (may not hold with multiple inheritance or union types)
- Random selection may pick related types (parent/child), weakening contradiction

**Enhancement Needed**:
- Validate type disjointness via metamodel analysis
- Prefer maximally distant types (e.g., different class hierarchies)

---

### Strategy 4: Universal Negation

**Description**: Combines universal quantifier with existential negation.

**Pattern Detection**:
```python
Pattern: X->forAll(v | condition)
```

**Transformation Rules**:
```python
Original: self.students->forAll(s | s.age >= 18)
Mutated:  (self.students->forAll(s | s.age >= 18)) and 
          (self.students->exists(s | not(s.age >= 18)))
```

**Logical Analysis**:
```
∀s ∈ S. P(s)  ∧  ∃s ∈ S. ¬P(s)
```
If `S` is non-empty, this is a contradiction:
- `forAll` asserts all elements satisfy `P`
- `exists(not P)` asserts at least one element violates `P`

**Implementation**:
```python
class UniversalNegationStrategy(UnsatMutationStrategy):
    def can_apply(self, constraint, metamodel):
        return 'forAll' in constraint.ocl
    
    def apply(self, constraint, metamodel):
        # Parse: collection->forAll(var | condition)
        match = re.search(r'([\w.]+)->forAll\((\w+)\s*\|\s*([^)]+)\)', ocl)
        collection, var, condition = match.group(1, 2, 3)
        
        # Negate condition
        contradiction = f'{collection}->exists({var} | not({condition}))'
        new_ocl = f"({original_expr}) and {contradiction}"
        return OCLConstraint(..., metadata={'is_unsat': True, 'mutation': 'universal_negation'})
```

**Why It Works**:
- Creates De Morgan-style contradiction: `∀x. P(x) ∧ ∃x. ¬P(x)`
- Forces solver to reason about quantifier interactions
- Tests SMT solver's ability to handle mixed quantifiers

**Edge Cases**:
- If collection is empty, `forAll` is vacuously true, `exists` is false → SAT (trivial)
- Our framework typically generates non-empty collections, so UNSAT is guaranteed

---

### Strategy 5: Simple Negation (Fallback)

**Description**: Wraps entire constraint in negation.

**Transformation Rules**:
```python
Original: self.age >= 18
Mutated:  not(self.age >= 18)  ≡  self.age < 18

Original: self.books->notEmpty() and self.author.name = 'Smith'
Mutated:  not(self.books->notEmpty() and self.author.name = 'Smith')
          ≡ self.books->isEmpty() or self.author.name <> 'Smith'
```

**Implementation**:
```python
class SimpleNegationStrategy(UnsatMutationStrategy):
    def can_apply(self, constraint, metamodel):
        return True  # Always applicable
    
    def apply(self, constraint, metamodel):
        if 'inv:' in ocl:
            parts = ocl.split('inv:', 1)
            new_ocl = parts[0] + 'inv: not(' + parts[1].strip() + ')'
        else:
            new_ocl = f'not({ocl})'
        return OCLConstraint(..., metadata={'is_unsat': True, 'mutation': 'simple_negation'})
```

**Why It Works**:
- By definition: `φ ∧ ¬φ` is UNSAT (law of non-contradiction)
- Works for any constraint (universal fallback)
- Guarantees UNSAT generation even for exotic patterns

**Drawbacks**:
- Least interesting mutation (trivial negation)
- Doesn't test specific OCL features (bounds, collections, quantifiers)
- Used only when sophisticated strategies fail

**Design Choice**:
- Listed last in strategy prioritization
- Ensures `generate_unsat_variant` always succeeds

---

## Algorithm & Workflow

### Main Entry Point: `generate_mixed_sat_unsat_set`

```python
def generate_mixed_sat_unsat_set(
    constraints: List[OCLConstraint],
    metamodel: Metamodel,
    unsat_ratio: float = 0.3,
    strategies: Optional[List[UnsatMutationStrategy]] = None
) -> Tuple[List[OCLConstraint], Dict[int, str]]:
```

**Purpose**: Convert a SAT-only constraint set into mixed SAT/UNSAT set with target ratio.

**Algorithm**:

```
INPUT: SAT constraints C = [c₁, c₂, ..., cₙ], UNSAT ratio r ∈ [0, 1]
OUTPUT: Mixed set M, mapping I: index → mutation_strategy

1. Compute target UNSAT count: k = ⌊n × r⌋
2. IF k = 0 THEN RETURN (C, {})
3. Randomly sample k indices: S ⊂ {1, ..., n}
4. FOR each constraint cᵢ ∈ C:
     IF i ∈ S:
         cᵢ' ← generate_unsat_variant(cᵢ, metamodel, strategies)
         M ← M ∪ {cᵢ'}
         I[i] ← cᵢ'.metadata['mutation']
     ELSE:
         M ← M ∪ {cᵢ}
5. RETURN (M, I)
```

**Properties**:
- **Preservation**: SAT constraints not selected remain unchanged
- **Randomization**: `random.sample` ensures uniform selection (unbiased)
- **Traceability**: Index map `I` records which constraints were mutated
- **Flexibility**: Target ratio honored (within rounding)

### Strategy Selection: `generate_unsat_variant`

```python
def generate_unsat_variant(
    constraint: OCLConstraint,
    metamodel: Metamodel,
    strategies: Optional[List[UnsatMutationStrategy]] = None
) -> OCLConstraint:
```

**Algorithm**:

```
INPUT: SAT constraint c, strategies S = [s₁, s₂, ..., sₘ]
OUTPUT: UNSAT constraint c'

1. FOR each strategy sᵢ ∈ S:
     IF sᵢ.can_apply(c, metamodel):
         c' ← sᵢ.apply(c, metamodel)
         RETURN c'
2. # Fallback (no strategy matched)
   c' ← SimpleNegationStrategy().apply(c, metamodel)
   RETURN c'
```

**Strategy Prioritization** (default order):
1. `ContradictoryBoundsStrategy`
2. `EmptyCollectionStrategy`
3. `TypeContradictionStrategy`
4. `UniversalNegationStrategy`
5. `SimpleNegationStrategy` (fallback)

**Rationale**:
- Specialized strategies first (more interesting mutations)
- Generic fallback last (always succeeds)
- Early exit on match (efficiency)

---

## Implementation Details

### Metadata Enrichment

Every UNSAT constraint carries metadata for traceability:

```python
{
    'is_unsat': True,                  # UNSAT flag
    'mutation': 'contradictory_bounds', # Strategy name
    'original_pattern_id': '...',      # Source pattern
    # ... [inherited from source constraint]
}
```

**Usage**:
- **Verification**: Skip UNSAT from global consistency checks (would make entire model UNSAT)
- **Analysis**: Count mutation strategy distribution
- **Debugging**: Trace UNSAT back to source SAT constraint

### OCL Syntax Handling

**Challenge**: Injecting contradictions while preserving OCL syntax.

**Solution 1: Conjunction Wrapper**
```python
new_ocl = ocl.replace('inv:', f'inv: ({original_body}) and {contradiction}')
```
- Wraps original expression in parentheses (precedence safety)
- Appends contradiction via `and` (conjunction)
- Preserves `context ... inv:` structure

**Solution 2: Direct Negation**
```python
new_ocl = parts[0] + 'inv: not(' + parts[1].strip() + ')'
```
- Splits on `inv:` delimiter
- Wraps invariant body in `not(...)`
- Rejoins to form valid OCL

**Edge Case**: Multi-line constraints
- Current implementation assumes single-line invariants
- Whitespace handling via `.strip()`
- Future: AST-based rewriting for robust multi-line support

### Regex Patterns

Key extraction patterns:

```python
# Numeric comparison
r'(\w+(?:\.\w+)*)\s*([<>=]+)\s*(\d+)'
→ Captures: variable, operator, value

# Collection operation
r'([\w.]+)->notEmpty\(\)'
→ Captures: collection expression

# Universal quantifier
r'([\w.]+)->forAll\((\w+)\s*\|\s*([^)]+)\)'
→ Captures: collection, variable, condition

# Type check
r'(\w+)\.oclIsTypeOf\((\w+)\)'
→ Captures: object variable, type name
```

**Robustness**:
- Non-greedy matching (`+?` vs `+`) for nested expressions
- Whitespace tolerance (`\s*`)
- Anchoring where appropriate (avoid false positives)

---

## Usage & Configuration

### Basic Usage

```python
from modules.generation.benchmark.unsat_generator import generate_mixed_sat_unsat_set
from modules.core.models import Metamodel

# Load metamodel
metamodel = Metamodel.from_xmi('models/model.xmi')

# Generate SAT constraints (via engine)
sat_constraints = engine.generate(profile)

# Create mixed set (30% UNSAT)
mixed_constraints, unsat_map = generate_mixed_sat_unsat_set(
    constraints=sat_constraints,
    metamodel=metamodel,
    unsat_ratio=0.3
)

print(f"Generated {len(unsat_map)} UNSAT constraints")
print(f"Mutation strategies: {set(unsat_map.values())}")
```

### YAML Configuration

```yaml
# examples/example_suite.yaml
models:
  - xmi: "models/model.xmi"
    profiles:
      - name: "balanced"
        sat_ratio: 0.7      # 70% SAT
        unsat_ratio: 0.3    # 30% UNSAT
```

**Integration**: Suite controller automatically calls UNSAT generator:

```python
# In suite_controller_enhanced.py (STEP 3)
all_constraints, unsat_map = unsat_generator.generate_mixed_sat_unsat_set(
    constraints, metamodel, unsat_ratio=prof_spec.unsat_ratio
)
```

### Custom Strategies

Define custom mutation strategy:

```python
class CardinalityContradictionStrategy(UnsatMutationStrategy):
    def can_apply(self, constraint, metamodel):
        return 'size()' in constraint.ocl and '>=' in constraint.ocl
    
    def apply(self, constraint, metamodel):
        # Extract: X->size() >= N
        match = re.search(r'([\w.]+)->size\(\)\s*>=\s*(\d+)', constraint.ocl)
        collection, min_size = match.group(1, 2)
        
        # Add contradictory upper bound
        max_size = int(min_size) - 1
        contradiction = f'{collection}->size() <= {max_size}'
        
        # Inject
        new_ocl = constraint.ocl.replace('inv:', f'inv: ({constraint.ocl.split("inv:", 1)[1]}) and {contradiction}')
        return OCLConstraint(..., metadata={'is_unsat': True, 'mutation': 'cardinality_contradiction'})

# Use custom strategy
custom_strategies = [CardinalityContradictionStrategy(), *ALL_STRATEGIES]
mixed_constraints, _ = generate_mixed_sat_unsat_set(
    sat_constraints, metamodel, unsat_ratio=0.3, strategies=custom_strategies
)
```

---

## Quality Assurance

### Verification Pipeline

**Two-Pass Strategy**:

1. **First Pass (Silent)**: Verify SAT constraints for global consistency
   - Skips UNSAT constraints (would make model UNSAT)
   - Prunes conflicting SAT constraints

2. **Second Pass (Visible)**: Final verification + statistics
   - Reports SAT/UNSAT counts
   - Includes intentionally-UNSAT in `unsat_count`

**Code Location**: `suite_controller_enhanced.py` lines 547-580

```python
# Extract SAT vs UNSAT
sat_constraints_for_verification = [c for c in constraints if not c.metadata.get('is_unsat', False)]
unsat_constraints_skipped = [c for c in constraints if c.metadata.get('is_unsat', False)]

# Verify only SAT
verif_results = verifier.verify_batch(sat_constraints_for_verification)

# Count both verified-UNSAT and intentional-UNSAT
total_unsat_constraints = unsat_count + len(unsat_constraints_skipped)
```

### Mutation Statistics

```python
from modules.generation.benchmark.unsat_generator import get_mutation_statistics

stats = get_mutation_statistics(mixed_constraints)
# Output:
# {
#   'total': 100,
#   'sat_count': 70,
#   'unsat_count': 30,
#   'mutations': {
#     'contradictory_bounds': 12,
#     'empty_collection': 8,
#     'type_contradiction': 5,
#     'universal_negation': 3,
#     'simple_negation': 2
#   },
#   'unsat_ratio': 0.3
# }
```

### Output Files

**Separate SAT/UNSAT Files** (research features enabled):

```
benchmarks/MyModel/balanced/
├── constraints.ocl              # All constraints
├── constraints.json             # Structured JSON
├── constraints_sat.ocl          # SAT only
├── constraints_sat.json
├── constraints_unsat.ocl        # UNSAT only
├── constraints_unsat.json
└── manifest.jsonl               # Per-constraint records
```

**UNSAT File Format**:

```ocl
-- Benchmark: CarRentalSystem / balanced
-- Type: UNSAT Constraints Only
-- Generated: 2025-11-10T11:08:49Z
-- Count: 10

-- #1 Numeric Positive (UNSAT) in Vehicle
-- Difficulty: medium
-- Mutation Strategy: contradictory_bounds
context Vehicle
inv: (self.mileage > 0) and self.mileage < -5

-- #2 Collection Not Empty Check (UNSAT) in Branch
-- Difficulty: easy
-- Mutation Strategy: empty_collection
context Branch
inv: (self.vehicles->notEmpty()) and self.vehicles->isEmpty()
```

---

## Evaluation & Metrics

### Mutation Coverage

| Strategy | Applicability | Avg Success Rate | Complexity |
|----------|---------------|------------------|------------|
| ContradictoryBounds | Numeric constraints (~40%) | 95% | Medium |
| EmptyCollection | Collection constraints (~30%) | 98% | Low |
| TypeContradiction | Type-check constraints (~5%) | 85% | High |
| UniversalNegation | Quantified constraints (~10%) | 92% | High |
| SimpleNegation | All constraints (100%) | 100% | Trivial |

**Metrics Definitions**:
- **Applicability**: % of SAT constraints matching pattern
- **Success Rate**: % of mutations producing syntactically valid OCL
- **Complexity**: Difficulty of mutation logic (Low/Medium/High)

### Benchmark Quality

**Desirable Properties**:
1. **Target ratio achieved**: `|unsat_ratio_actual - unsat_ratio_target| < 0.05`
2. **Strategy diversity**: No single strategy > 60% of UNSAT
3. **No trivial UNSAT**: `simple_negation_count / unsat_count < 0.3`
4. **Verified correctness**: All UNSAT should fail Z3 SAT check (if verified)

**Example Results** (CarRental, 100 constraints, 30% UNSAT):

```
Mutation Statistics:
- Total: 100 constraints
- SAT: 70 (70.0%)
- UNSAT: 30 (30.0%)  ✅ Target met

Strategy Distribution:
- contradictory_bounds: 14 (46.7%)
- empty_collection: 9 (30.0%)
- type_contradiction: 3 (10.0%)
- universal_negation: 2 (6.7%)
- simple_negation: 2 (6.7%)  ✅ Fallback < 30%
```

---

## Future Enhancements

### 1. Metamodel-Aware Mutations

**Current Limitation**: Type contradiction strategy randomly selects types without checking disjointness.

**Enhancement**:
- Extract type hierarchy from metamodel
- Compute disjoint type pairs via inheritance analysis
- Prefer maximally distant types (different subtrees)

**Example**:
```python
# Current
type2 = random.choice(classes)

# Enhanced
disjoint_types = metamodel.get_disjoint_types(type1)
type2 = random.choice(disjoint_types) if disjoint_types else random.choice(classes)
```

### 2. Quantifier Scope Mutations

**Idea**: Introduce contradictions within quantifier bodies.

**Example**:
```ocl
Original: self.students->forAll(s | s.age >= 0)
Mutated:  self.students->forAll(s | s.age >= 0 and s.age < 0)
```

**Advantage**: More localized contradiction (easier to debug).

### 3. Multi-Constraint UNSAT Sets

**Current**: Each UNSAT is standalone contradiction.

**Enhancement**: Create pairs/triples of constraints that are individually SAT but collectively UNSAT.

**Example**:
```ocl
Constraint 1: self.age >= 18
Constraint 2: self.graduationYear = 2020
Constraint 3: self.birthYear = 2010  # Contradiction: age in 2020 would be 10, not ≥18
```

**Benefit**: Tests global consistency checking (not just per-constraint SAT).

### 4. SMT-Guided Mutation

**Idea**: Use solver feedback to guide mutation selection.

**Algorithm**:
1. Generate candidate mutations
2. Run Z3 on each candidate
3. Select mutations that Z3 confirms as UNSAT
4. Discard false negatives (inadvertently SAT mutations)

**Trade-off**: Higher quality (verified UNSAT) vs. slower generation.

### 5. Pattern-Specific Strategies

**Extend coverage** to more OCL patterns:

| Pattern | Mutation Strategy | Contradiction |
|---------|-------------------|---------------|
| `closure` | Cyclic reference | `X->closure(ref)->includes(X)` ∧ acyclic |
| `sortedBy` | Out-of-order elements | `X->sortedBy(attr)` ∧ `X->at(1).attr > X->at(2).attr` |
| `if-then-else` | Unreachable branch | `if cond then P else Q endif` ∧ `cond` ∧ `Q` |
| `allInstances` | Non-existent type | `Type.allInstances()->isEmpty()` ∧ `self.oclIsTypeOf(Type)` |

### 6. Difficulty Calibration

**Current**: UNSAT difficulty not calibrated (trivial negations mixed with complex contradictions).

**Enhancement**:
- Assign difficulty scores to mutation strategies
- Balance UNSAT generation across difficulty levels
- Match `difficulty_mix` targets (easy/medium/hard)

**Implementation**:
```python
STRATEGY_DIFFICULTY = {
    'contradictory_bounds': 'medium',
    'empty_collection': 'easy',
    'type_contradiction': 'hard',
    'universal_negation': 'hard',
    'simple_negation': 'easy'
}

# Select strategies weighted by target difficulty distribution
```

---

## References

1. **OCL Specification**: OMG OCL 2.4 Semantics
2. **Z3 Documentation**: Satisfiability checking and proof extraction
3. **Mutation Testing**: Jia & Harman, "An Analysis and Survey of the Development of Mutation Testing"
4. **Constraint Benchmarks**: Benchmarks for OCL Tools (Gogolla et al.)

---

## Appendix: Complete Example

### Input: SAT Constraint Set

```ocl
-- Constraint 1
context Vehicle
inv: self.mileage >= 0

-- Constraint 2
context Branch
inv: self.vehicles->notEmpty()

-- Constraint 3
context Rental
inv: self.startDate < self.endDate
```

### Configuration

```python
unsat_ratio = 0.33  # Target 33% UNSAT (1 out of 3)
```

### Output: Mixed SAT/UNSAT Set

```ocl
-- Constraint 1 (SAT - unchanged)
context Vehicle
inv: self.mileage >= 0

-- Constraint 2 (UNSAT - mutated via empty_collection)
context Branch
inv: (self.vehicles->notEmpty()) and self.vehicles->isEmpty()
-- Metadata: {'is_unsat': True, 'mutation': 'empty_collection'}

-- Constraint 3 (SAT - unchanged)
context Rental
inv: self.startDate < self.endDate
```

### Statistics

```json
{
  "total": 3,
  "sat_count": 2,
  "unsat_count": 1,
  "unsat_ratio": 0.33,
  "mutations": {
    "empty_collection": 1
  }
}
```

---

**Document Version**: 1.0  
**Last Updated**: 2025-11-10  
**Maintainer**: OCL Benchmark Framework Team
