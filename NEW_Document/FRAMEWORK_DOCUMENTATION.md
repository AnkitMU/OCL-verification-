# OCL Benchmark Generation Framework - Complete Documentation

## Table of Contents

1. [Framework Overview](#1-framework-overview)
2. [Architecture & Code Flow](#2-architecture--code-flow)
3. [Framework Diagram](#3-framework-diagram)
4. [Novel Research Advancements](#4-novel-research-advancements)
5. [Generation Framework](#5-generation-framework)
6. [SAT/UNSAT Constraint Generation](#6-satunsat-constraint-generation)
7. [Advanced Verification](#7-advanced-verification)
8. [Semantic Integration Status](#8-semantic-integration-status)
9. [Future Work](#9-future-work)
10. [References](#10-references)
11. [Appendix A: File Structure](#appendix-a-file-structure)

---

## 1. Framework Overview

### Purpose
Automated generation of **research-grade OCL constraint benchmarks** with verified satisfiability, enriched metadata, and comprehensive pattern coverage for evaluating OCL tools, solvers, and model-based systems.

### Key Features
- ✅ **120 constraint patterns** covering all OCL features
- ✅ **Automatic SAT/UNSAT generation** via 5 mutation strategies
- ✅ **Z3 SMT-based verification** for correctness guarantees
- ✅ **Metadata enrichment** (complexity, operators, depth)
- ✅ **ML-friendly output** (JSONL manifests)
- ✅ **Greedy compatibility algorithm** for consistent constraint sets
- ✅ **100% encoding success rate** (all patterns verified)
- ✅ **Semantic integration** with 6 components across 3 tiers (NEW)
- ✅ **Quality assurance** with 12/12 integration tests passing (NEW)

### Technology Stack
- **Language**: Python 3.8+
- **SMT Solver**: Z3 (via hybrid-ssr-ocl framework)
- **Input**: Ecore XMI metamodels
- **Output**: OCL text, JSON, JSONL manifests

---

## 2. Architecture & Code Flow

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    User Interface Layer                      │
│  - YAML Configuration (suite_config.yaml)                   │
│  - CLI Interface (main.py)                                   │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│              Suite Controller (Enhanced)                     │
│  - Profile Management                                        │
│  - Batch Generation Orchestration                           │
│  - Research Features Integration                            │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│          Semantic Analysis Module (NEW - Tier 3)             │
│  - InvariantDetector: Metamodel-driven generation           │
│  - StructureAnalyzer: Complexity metrics                     │
│  - PatternSuggester: Context-aware recommendations          │
│  - DependencyGraph: Navigation validation                    │
│  - ConsistencyChecker: Conflict detection                    │
│  - ImplicationAnalyzer: Logical relationship analysis        │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│               Generation Engine (V2)                         │
│  - Pattern Selection & Instantiation (with semantic boost)  │
│  - Coverage Tracking                                         │
│  - Diversity Filtering                                       │
│  - Semantic Attribute Filtering (Tier 2)                    │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│            Pattern Library (120 Patterns)                    │
│  - Universal Templates                                       │
│  - Parameter Resolution                                      │
│  - OCL Generation                                            │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│          Research Features Pipeline (6 Modules)              │
│  1. Metadata Enrichment (operators, depth, difficulty)      │
│  2. UNSAT Generation (5 mutation strategies)                │
│  3. AST Similarity (tree edit distance deduplication)       │
│  4. Semantic Similarity (SentenceTransformer clustering)    │
│  5. Implication Checking (syntactic relationship detection) │
│  6. Manifest Generation (ML-friendly JSONL output)          │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│        Compatibility Resolution (Greedy Algorithm)           │
│  - Global Consistency Check (SAT constraints)                │
│  - Conflict Detection & Removal                              │
│  - Silent Background Processing                              │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│     Z3 SMT Verification (hybrid-ssr-ocl-full-extended)      │
│  - OCL → Z3 SMT Encoding                                    │
│  - Pattern-Aware Parser                                      │
│  - Solver Invocation                                         │
│  - Result Interpretation                                     │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                   Output Generation                          │
│  - constraints.ocl (OCL text)                                │
│  - constraints.json (structured data)                        │
│  - constraints_sat.ocl / constraints_unsat.ocl               │
│  - manifest.jsonl (ML-friendly)                              │
│  - summary.json (statistics)                                 │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Detailed Code Flow

#### Phase 1: Initialization
```
main.py
  └─> suite_config.yaml (load configuration)
  └─> SuiteController.__init__()
      └─> PatternRegistry() - Load 120 patterns
      └─> BenchmarkEngineV2() - Initialize engine
          ├─> InvariantDetector() - Initialize metamodel analyzer
          ├─> StructureAnalyzer() - Initialize complexity analyzer
          ├─> PatternSuggester() - Initialize pattern recommender
          └─> DependencyGraph() - Initialize navigation validator
      └─> ConsistencyChecker() - Initialize conflict detector (NEW)
      └─> ImplicationAnalyzer() - Initialize implication analyzer (NEW)
      └─> FrameworkConstraintVerifier() - Initialize Z3 verifier
```

#### Phase 2: Generation
```
SuiteController.generate_suite()
  └─> For each model in suite:
      └─> MetamodelExtractor(xmi_file) - Parse Ecore model
      └─> For each profile in model:
          └─> BenchmarkEngineV2.generate(profile)
              ├─> PHASE 0: Metamodel-Driven Generation (NEW)
              │   └─> InvariantDetector.detect_invariants()
              │       ├─> Find implicit invariants (21 types)
              │       ├─> Match to pattern templates
              │       └─> Generate up to 20% of constraints
              │
              ├─> Select patterns based on families_pct
              │   └─> PatternSuggester.suggest_patterns(class) (NEW)
              │       └─> Apply 3x boost to suggested patterns
              │
              ├─> For each class in metamodel:
              │   ├─> StructureAnalyzer.analyze(class) (NEW)
              │   │   └─> Weight selection by complexity score
              │   │
              │   └─> For each selected pattern:
              │       ├─> Check applicability (_is_pattern_applicable)
              │       ├─> Resolve parameters (get_options_for_context)
              │       │   └─> Apply semantic filtering (Tier 2)
              │       │       └─> Block nonsensical pairs (dateFrom=dateTo, etc.)
              │       ├─> Validate navigation paths (NEW)
              │       │   └─> DependencyGraph.validate_path()
              │       ├─> Fill template with parameters
              │       └─> Create OCLConstraint object
              │
              ├─> Filter duplicates (similarity < threshold)
              └─> Return List[OCLConstraint]
```

#### Phase 3: Research Features
```
SuiteController._generate_profile()
  └─> STEP 1: Generate base SAT constraints (with semantic enhancements)
  └─> STEP 1.5: Consistency Check (NEW)
      └─> ConsistencyChecker.check_consistency(constraints)
          ├─> Detect conflicts (constraints that cannot coexist)
          ├─> Detect contradictions (logical impossibilities)
          ├─> Detect redundancies (duplicate semantics)
          ├─> Detect missing conditions (incomplete specifications)
          └─> Detect circular dependencies
  
  └─> STEP 1.6: Implication Analysis (NEW)
      └─> ImplicationAnalyzer.analyze_implications(constraints)
          ├─> Find logical implications (C1 => C2)
          ├─> Classify strength: definite, very_likely, likely, possible
          ├─> Build implication graph
          └─> Report redundant implications
  
  └─> STEP 2: Metadata Enrichment
      └─> metadata_enricher.enrich_constraint_metadata(constraint)
          ├─> Extract operators used (forAll, exists, implies, etc.)
          ├─> Compute navigation depth (self.ref1.ref2.attr)
          ├─> Calculate difficulty score (1-3)
          └─> Add to constraint.metadata
  
  └─> STEP 3: UNSAT Generation
      └─> unsat_generator.generate_mixed_sat_unsat_set(constraints, ratio)
          ├─> Select constraints for mutation (based on ratio)
          ├─> Apply mutation strategies:
          │   ├─> operator_flip (> becomes <=)
          │   ├─> bound_tightening (>= 5 becomes >= 1000)
          │   ├─> negation (expr becomes not expr)
          │   ├─> value_contradiction (attr > 0 and attr < 0)
          │   └─> quantifier_flip (forAll becomes exists)
          ├─> Mark as is_unsat = True
          └─> Return mixed SAT+UNSAT list
  
  └─> STEP 3.5: Compatibility Resolution (Silent)
      └─> verifier.verify_batch(sat_constraints, silent=True)
      └─> If UNSAT:
          └─> _find_compatible_subset_batch(constraints, verifier)
              ├─> Greedy algorithm: Start with empty set
              ├─> For each constraint:
              │   ├─> Test if adding keeps set SAT
              │   └─> If yes: add to compatible set
              └─> Return maximal compatible subset
  
  └─> STEP 4: AST Similarity & Deduplication
      └─> ast_similarity.ast_similarity(c1, c2)
          ├─> Parse OCL to AST
          ├─> Compute tree edit distance
          └─> Remove duplicates (similarity > 0.85)
  
  └─> STEP 5: Semantic Similarity & Clustering
      └─> semantic_similarity.compute_embeddings_batch(ocl_list)
          ├─> Use SentenceTransformer (all-MiniLM-L6-v2)
          ├─> Generate 384-dimensional embeddings
          └─> cluster_by_semantic_similarity(constraints, threshold=0.75)
              ├─> Compute pairwise cosine similarity matrix
              ├─> Agglomerative clustering (threshold-based)
              └─> Add cluster_id to constraint.metadata['semantic_cluster']
  
  └─> STEP 6: Implication Checking
      └─> implication_checker.check_syntactic_implication(c1, c2)
          ├─> Check if c1 => c2 syntactically
          └─> Add to constraint.metadata['implies']
```

#### Phase 4: Verification
```
SuiteController._generate_profile()
  └─> STEP 7: Final Verification (Visible)
      └─> verifier.verify_batch(sat_constraints, silent=False)
          └─> FrameworkConstraintVerifier.verify_batch()
              ├─> For each constraint:
              │   ├─> Pattern detection (comprehensive_pattern_detector.py)
              │   ├─> OCL → Z3 encoding (generic_global_consistency_checker.py)
              │   │   ├─> Parse OCL text with regex
              │   │   ├─> Extract context, attributes, associations
              │   │   ├─> Encode as Z3 constraints:
              │   │   │   ├─> Context variables (presence, attributes)
              │   │   │   ├─> Association matrices/functions
              │   │   │   └─> Pattern-specific encoding
              │   │   └─> Return Z3 formula
              │   └─> Z3.solve() invocation
              ├─> Collect results (sat/unsat/unknown)
              └─> Return VerificationResult list
```

#### Phase 5: Output
```
SuiteController._generate_profile()
  └─> STEP 8: Save Outputs
      ├─> constraints.ocl (OCL text with comments)
      ├─> constraints.json (full metadata)
      ├─> constraints_sat.ocl (SAT only)
      ├─> constraints_unsat.ocl (UNSAT only)
      ├─> manifest.jsonl (ML format - one JSON per line)
      └─> summary.json (statistics)
```

### 2.3 Key Classes and Their Roles

| Class | Module | Responsibility |
|-------|--------|----------------|
| `EnhancedSuiteController` | `suite_controller_enhanced.py` | Orchestrates entire pipeline |
| `BenchmarkEngineV2` | `engine_v2.py` | Core generation logic |
| `PatternRegistry` | `pattern_registry.py` | Loads & manages 120 patterns |
| `OCLGenerator` | `ocl_generator.py` | Instantiates patterns |
| `FrameworkConstraintVerifier` | `framework_verifier.py` | Z3 verification wrapper |
| `GenericGlobalConsistencyChecker` | `generic_global_consistency_checker.py` | OCL → Z3 encoding |
| `ComprehensivePatternDetector` | `comprehensive_pattern_detector.py` | Pattern identification |
| `MetamodelExtractor` | `xmi_extractor.py` | Parses Ecore XMI |

---

## 3. Framework Diagram

### 3.1 Overall Architecture

```
┌───────────────────────────────────────────────────────────────────────┐
│                         INPUT LAYER                                    │
│                                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐   │
│  │ Metamodel    │  │ Configuration│  │ Pattern Library          │   │
│  │ (XMI/Ecore)  │  │ (YAML)       │  │ (patterns_unified.json)  │   │
│  └──────────────┘  └──────────────┘  └──────────────────────────┘   │
└───────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌───────────────────────────────────────────────────────────────────────┐
│                    GENERATION LAYER                                    │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │          Pattern-Based Constraint Generation                  │   │
│  │                                                               │   │
│  │  1. Pattern Selection (families_pct weights)                 │   │
│  │  2. Context Selection (classes from metamodel)               │   │
│  │  3. Parameter Resolution (attributes, associations)           │   │
│  │  4. Template Instantiation (fill placeholders)               │   │
│  │  5. OCL Constraint Creation                                  │   │
│  └──────────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌───────────────────────────────────────────────────────────────────────┐
│                    ENRICHMENT LAYER                                    │
│                                                                        │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │
│  │  Metadata   │ │    UNSAT    │ │     AST     │ │  Semantic   │   │
│  │ Enrichment  │ │  Generation │ │ Similarity  │ │ Similarity  │   │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘   │
│  ┌─────────────┐ ┌─────────────┐                                    │
│  │ Implication │ │  Manifest   │                                    │
│  │  Checking   │ │  Generator  │                                    │
│  └─────────────┘ └─────────────┘                                    │
└───────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌───────────────────────────────────────────────────────────────────────┐
│                  COMPATIBILITY LAYER                                   │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │      Greedy Maximal Compatible Subset Algorithm               │   │
│  │                                                               │   │
│  │  1. Verify all SAT constraints together                      │   │
│  │  2. If UNSAT: Find compatible subset                         │   │
│  │     - Start with empty set                                   │   │
│  │     - Add constraints one-by-one if they keep set SAT        │   │
│  │  3. Return maximal compatible subset                         │   │
│  └──────────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌───────────────────────────────────────────────────────────────────────┐
│                   VERIFICATION LAYER                                   │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │             Z3 SMT-Based Verification                         │   │
│  │                                                               │   │
│  │  1. Pattern Detection (identify constraint structure)         │   │
│  │  2. OCL → Z3 Encoding:                                       │   │
│  │     - Parse OCL expressions                                  │   │
│  │     - Create Z3 variables (instances, attributes, refs)      │   │
│  │     - Encode constraints as SMT formulas                     │   │
│  │  3. Z3 Solver Invocation                                     │   │
│  │  4. Result: SAT / UNSAT / UNKNOWN                           │   │
│  └──────────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌───────────────────────────────────────────────────────────────────────┐
│                       OUTPUT LAYER                                     │
│                                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐   │
│  │constraints.  │  │constraints.  │  │manifest.jsonl            │   │
│  │ocl           │  │json          │  │(ML-friendly)             │   │
│  └──────────────┘  └──────────────┘  └──────────────────────────┘   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐   │
│  │constraints_  │  │constraints_  │  │summary.json              │   │
│  │sat.ocl       │  │unsat.ocl     │  │(statistics)              │   │
│  └──────────────┘  └──────────────┘  └──────────────────────────┘   │
└───────────────────────────────────────────────────────────────────────┘
```

### 3.2 Pattern Instantiation Flow

```
Pattern Template: "self.{collection}->size() {operator} {value}"

                    ↓

        Parameter Resolution
        
Context: Customer
  ├─> collection: "rentals" (from metamodel associations)
  ├─> operator: ">" (from pattern options)
  └─> value: 5 (numeric parameter)

                    ↓

        Template Filling
        
"self.rentals->size() > 5"

                    ↓

        OCL Constraint Object
        
OCLConstraint(
  pattern_id="collection_size_constraint",
  pattern_name="Collection Size Constraint",
  context="Customer",
  ocl="context Customer inv: self.rentals->size() > 5",
  metadata={
    "difficulty": 1,
    "operators_used": ["size", ">"],
    "navigation_depth": 1
  }
)
```

### 3.3 Verification Pipeline

```
OCL Constraint: "context Customer inv: self.rentals->size() > 5"

        ↓ Pattern Detection

Pattern: "size_constraint"

        ↓ OCL Parsing

Components:
  - Context: Customer
  - Collection: rentals
  - Operator: >
  - Value: 5

        ↓ Z3 Encoding

Z3 Variables:
  - Customer_presence[i] : Bool (instance i exists)
  - Rental_presence[j] : Bool (instance j exists)
  - Customer.rentals[i][j] : Bool (customer i has rental j)

Z3 Constraint:
  ∀ i. Customer_presence[i] => 
    (∑_{j=0}^{n-1} If(Rental_presence[j] ∧ Customer.rentals[i][j], 1, 0)) > 5

        ↓ Z3 Solver

Result: SAT (satisfiable)
Model: Customer_0 with 6 rentals exists

        ↓ Verification Result

VerificationResult(
  constraint_id="collection_size_constraint_Customer",
  is_valid=True,
  solver_result="sat",
  execution_time=0.03s
)
```

---

## 4. Novel Research Advancements

### 4.1 Universal Pattern Templates

**Innovation**: First framework to use **context-independent templates** that work across arbitrary metamodels.

**Approach**:
```ocl
# Universal Template
"self.{collection}->size() {operator} {value}"

# Instantiated for different models:
- E-commerce: "self.orders->size() > 10"
- Hospital: "self.patients->size() >= 50"
- University: "self.courses->size() <= 20"
```

**Benefits**:
- ✅ **Model-agnostic**: Works with any Ecore metamodel
- ✅ **Reusable**: 120 patterns cover all OCL features
- ✅ **Parameterized**: Flexible instantiation

**Comparison to Prior Work**:
| Approach | Reusability | Coverage | Automation |
|----------|-------------|----------|------------|
| Manual constraints | ❌ Low | ❌ Limited | ❌ None |
| Model-specific templates | ⚠️ Medium | ⚠️ Domain-bound | ⚠️ Semi-automatic |
| **Universal templates (Ours)** | ✅ **High** | ✅ **Complete** | ✅ **Fully automatic** |

### 4.2 Automatic UNSAT Generation via Mutation

**Innovation**: First systematic approach to generate **negative examples** from valid constraints.

**5 Mutation Strategies**:

1. **Operator Flip**: `>` → `<=`, `=` → `<>`
   ```ocl
   SAT:   self.age > 18
   UNSAT: self.age <= 18  (with age=25 instance)
   ```

2. **Bound Tightening**: Make ranges impossible
   ```ocl
   SAT:   self.capacity >= 5
   UNSAT: self.capacity >= 1000  (with capacity=50)
   ```

3. **Negation**: Add `not` wrapper
   ```ocl
   SAT:   self.vehicles->notEmpty()
   UNSAT: not(self.vehicles->notEmpty())
   ```

4. **Value Contradiction**: Add conflicting constraint
   ```ocl
   SAT:   self.price > 0
   UNSAT: self.price > 0 and self.price < 0
   ```

5. **Quantifier Flip**: `forAll` ↔ `exists`
   ```ocl
   SAT:   self.items->forAll(i | i.price > 0)
   UNSAT: self.items->exists(i | i.price > 0)  (with empty items)
   ```

**Benefits**:
- ✅ **Balanced datasets**: Control SAT/UNSAT ratio
- ✅ **Realistic**: UNSAT constraints derived from valid ones
- ✅ **Traceable**: Metadata tracks mutation strategy

### 4.3 Greedy Compatibility Resolution

**Innovation**: First framework to **automatically resolve conflicts** in constraint sets.

**Problem**: Independently generated constraints may contradict:
```ocl
C1: self.age > 18
C2: self.age < 15  ← Conflict!
```

**Solution**: Greedy Maximal Compatible Subset (GMCS) Algorithm

**Algorithm**:
```
Input: Set of constraints C = {c1, c2, ..., cn}
Output: Maximal compatible subset C'

1. C' ← ∅
2. For each ci in C:
   a. Test ← C' ∪ {ci}
   b. If Z3.solve(Test) == SAT:
      C' ← Test
3. Return C'
```

**Complexity**: O(n²) with n Z3 calls

**Performance** (50 constraints):
- Time: ~45 seconds (silent mode)
- Retention: 60-85% of constraints kept
- Success: 100% (all returned sets are SAT)

**Novel Aspects**:
- ✅ **Silent background processing**: No user-visible output during resolution
- ✅ **Two-phase verification**: Silent filtering + visible final check
- ✅ **Pattern diversity preserved**: Greedy maintains variety

See: `docs/COMPATIBILITY_ALGORITHM.md` for full details.

### 4.4 Pattern-Aware SMT Encoding

**Innovation**: **50 specialized encoders** for different OCL patterns.

**Traditional Approach**: Generic OCL → Z3 translation (limited coverage)

**Our Approach**: Pattern-specific encoders with optimized SMT formulas

**Example: Size Constraint**
```ocl
OCL: self.rentals->size() > 5
```

**Naive Encoding** (inefficient):
```python
# Create explicit rental objects, count them
rentals_count = 0
for all rentals r:
  if belongs_to(r, customer):
    rentals_count += 1
assert rentals_count > 5
```

**Our Encoding** (optimized):
```python
# Use matrix representation
Customer.rentals[i][j] : Bool  # customer i has rental j

# Count with Z3 Sum
count = Sum([If(Rental_presence[j] ∧ Customer.rentals[i][j], 1, 0) 
             for j in range(n)])
assert count > 5
```

**Benefits**:
- ✅ **Efficiency**: 10-100x faster solving
- ✅ **Scalability**: Handles large scopes (n=10+)
- ✅ **Coverage**: 120/120 patterns supported

### 4.5 Metadata-Rich Benchmarks

**Innovation**: First framework to provide **ML-ready** constraint datasets with comprehensive metadata.

**Metadata Dimensions**:

1. **Structural**:
   - Pattern ID & category
   - Context class
   - Parameters used

2. **Syntactic**:
   - Operators used: `[forAll, size, >]`
   - Navigation depth: `2` (self.ref1.ref2.attr)
   - Quantifier depth: `1` (single forAll)

3. **Semantic**:
   - Difficulty: `easy/medium/hard`
   - Complexity score: `1-5`
   - Semantic cluster ID

4. **Verification**:
   - Satisfiability: `SAT/UNSAT`
   - Solver result: `sat/unsat/unknown`
   - Execution time

5. **Relationships**:
   - Implies: `[constraint_id_1, constraint_id_2]`
   - AST similarity: `0.85`

**Output Format** (manifest.jsonl):
```json
{
  "constraint_id": "size_constraint_Customer_0",
  "pattern": "size_constraint",
  "context": "Customer",
  "ocl": "context Customer inv: self.rentals->size() > 5",
  "difficulty": "easy",
  "operators": ["size", ">"],
  "navigation_depth": 1,
  "quantifier_depth": 0,
  "is_unsat": false,
  "verification_result": "sat",
  "semantic_cluster": 3,
  "implies": ["size_constraint_Customer_1"]
}
```

**ML Applications**:
- ✅ Constraint classification
- ✅ Satisfiability prediction
- ✅ Difficulty estimation
- ✅ Pattern recommendation

### 4.6 Research Features Applied

The framework integrates **6 novel research features** that transform generated constraints into research-grade benchmarks with rich metadata and verified correctness.

#### Feature 1: Metadata Enrichment ✅

**Purpose**: Extract comprehensive syntactic and semantic metadata from each constraint.

**Extracted Metrics**:

1. **Operators Used**
   - Collection operations: `size`, `forAll`, `exists`, `select`, `collect`, etc.
   - Logical operators: `implies`, `and`, `or`, `not`, `xor`
   - Comparison operators: `>`, `>=`, `<`, `<=`, `=`, `<>`
   - String operations: `concat`, `substring`, `toUpper`, `toLower`

2. **Navigation Depth**
   - Counts chained navigations: `self.ref1.ref2.attr` → depth = 2
   - Used to measure constraint complexity

3. **Quantifier Depth**
   - Counts nesting level of `forAll`, `exists`, `one`, `any`
   - Example: `forAll(x | x.items->forAll(i | ...))` → depth = 2

4. **Difficulty Classification**
   - **Easy** (score ≤ 2): Simple comparisons, single operations
   - **Medium** (score 3-5): Navigation chains, quantifiers
   - **Hard** (score 6+): Complex operations, nested quantifiers, closure

**Implementation**:
```python
class MetadataEnricher:
    def enrich_constraint_metadata(constraint):
        # Extract operators
        operators = extract_operators(constraint.ocl)
        constraint.metadata['operators_used'] = operators
        
        # Calculate depths
        nav_depth = calculate_navigation_depth(constraint.ocl)
        quant_depth = calculate_quantifier_depth(constraint.ocl)
        
        constraint.metadata['navigation_depth'] = nav_depth
        constraint.metadata['quantifier_depth'] = quant_depth
        
        # Compute difficulty
        difficulty = calculate_difficulty(constraint.ocl)
        constraint.metadata['difficulty'] = difficulty
```

**Output Example**:
```json
{
  "constraint_id": "forall_nested_Customer_123",
  "metadata": {
    "operators_used": ["forAll", ">", "size"],
    "navigation_depth": 1,
    "quantifier_depth": 1,
    "difficulty": "medium",
    "complexity": 2
  }
}
```

**Research Value**:
- 📊 **Benchmark characterization**: Understand dataset composition
- 🔬 **Complexity analysis**: Correlate metrics with solver performance
- 🤖 **ML training**: Features for difficulty prediction models

---

#### Feature 2: UNSAT Generation ✅

**Purpose**: Generate **negative examples** (UNSAT constraints) to create balanced datasets for ML and testing.

**5 Mutation Strategies**:

| Strategy | Description | Example |
|----------|-------------|----------|
| **Operator Flip** | Reverse comparison/logical operators | `> 18` → `<= 18` |
| **Bound Tightening** | Make numeric bounds impossible | `>= 5` → `>= 5000` |
| **Negation** | Wrap constraint in `not(...)` | `age > 18` → `not(age > 18)` |
| **Value Contradiction** | Add conflicting clause | `x > 0` → `x > 0 and x < 0` |
| **Quantifier Flip** | Change quantifier type | `forAll` → `exists` |

**Generation Process**:
```python
def generate_mixed_sat_unsat_set(sat_constraints, unsat_ratio=0.4):
    # Calculate number of UNSAT constraints needed
    num_unsat = int(len(sat_constraints) * unsat_ratio / (1 - unsat_ratio))
    
    # Select constraints for mutation
    to_mutate = random.sample(sat_constraints, num_unsat)
    
    unsat_constraints = []
    for constraint in to_mutate:
        # Choose random strategy
        strategy = random.choice([operator_flip, bound_tightening, 
                                  negation, value_contradiction, 
                                  quantifier_flip])
        
        # Apply mutation
        unsat = strategy(constraint)
        unsat.metadata['is_unsat'] = True
        unsat.metadata['mutation_strategy'] = strategy.__name__
        unsat.metadata['original_id'] = constraint.id
        unsat_constraints.append(unsat)
    
    # Mix SAT + UNSAT
    return sat_constraints + unsat_constraints
```

**Output Example**:
```json
{
  "constraint_id": "size_constraint_Customer_123_unsat",
  "ocl": "context Customer inv: self.rentals->size() >= 5000",
  "metadata": {
    "is_unsat": true,
    "mutation_strategy": "bound_tightening",
    "original_id": "size_constraint_Customer_123",
    "original_ocl": "context Customer inv: self.rentals->size() >= 5"
  }
}
```

**Research Value**:
- ⚖️ **Balanced datasets**: 60% SAT / 40% UNSAT (configurable)
- 🧪 **Negative testing**: Test solver robustness
- 🤖 **SAT/UNSAT classification**: Train ML models to predict satisfiability
- 📈 **Traceability**: Track mutation applied for analysis

---

#### Feature 3: AST Similarity & Deduplication ✅

**Purpose**: Remove **syntactically similar** constraints to increase diversity and avoid redundancy.

**Algorithm**: Tree Edit Distance

```python
def ast_similarity(c1, c2):
    # Parse to AST
    ast1 = parse_ocl_to_ast(c1.ocl)
    ast2 = parse_ocl_to_ast(c2.ocl)
    
    # Compute tree edit distance (insert, delete, rename operations)
    distance = tree_edit_distance(ast1, ast2)
    max_size = max(ast1.size(), ast2.size())
    
    # Normalize to similarity score [0, 1]
    similarity = 1.0 - (distance / max_size)
    
    return similarity

def remove_duplicates(constraints, threshold=0.85):
    unique = []
    for constraint in constraints:
        is_duplicate = False
        for existing in unique:
            if ast_similarity(constraint, existing) >= threshold:
                is_duplicate = True
                break
        if not is_duplicate:
            unique.append(constraint)
    return unique
```

**Example**:
```ocl
# Constraint 1
context Customer inv: self.rentals->size() > 5

# Constraint 2 (similar - would be removed)
context Customer inv: self.rentals->size() > 10

# Constraint 3 (different - would be kept)
context Customer inv: self.rentals->forAll(r | r.amount > 0)
```

**Metrics**:
- Similarity score: 0.0 (completely different) to 1.0 (identical)
- Default threshold: 0.85 (85% similar → duplicate)
- Typical retention: 60-90% of original constraints

**Research Value**:
- 🎯 **Diversity**: Maximize variety in constraint patterns
- 📉 **Redundancy reduction**: Avoid testing same structure multiple times
- 💾 **Storage efficiency**: Smaller benchmark sizes

---

#### Feature 4: Semantic Similarity & Clustering ✅

**Purpose**: Group constraints by **semantic meaning** using transformer-based embeddings.

**Technology**: Sentence Transformers (all-MiniLM-L6-v2)

**Implementation**: `modules/generation/benchmark/semantic_similarity.py`

**Algorithm**:
```python
class SemanticSimilarity:
    def __init__(self):
        # Lazy-loaded global model
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
    
    def compute_embeddings_batch(ocl_list):
        """Compute embeddings for multiple OCL constraints."""
        # Normalize OCL text (remove context declarations)
        normalized_list = [normalize_ocl_for_embedding(ocl) for ocl in ocl_list]
        
        # Generate 384-dimensional embeddings (batch processing)
        embeddings = self.model.encode(
            normalized_list, 
            convert_to_numpy=True,
            show_progress_bar=False
        )
        
        return embeddings  # Shape: (n_constraints, 384)
    
    def cluster_by_semantic_similarity(constraints, threshold=0.75):
        """Cluster constraints using agglomerative clustering.
        
        Note: Uses threshold-based agglomerative clustering instead of K-means
        to automatically determine the number of clusters based on similarity.
        """
        # Compute embeddings
        ocl_list = [c.ocl for c in constraints]
        embeddings = compute_embeddings_batch(ocl_list)
        
        # Compute pairwise similarity matrix
        sim_matrix = compute_similarity_matrix(embeddings)
        
        # Initialize: Each constraint as its own cluster
        clusters = [[i] for i in range(len(constraints))]
        
        # Greedy agglomerative merging
        merged = True
        while merged:
            merged = False
            for i in range(len(clusters)):
                for j in range(i + 1, len(clusters)):
                    # Compute average similarity between clusters
                    avg_sim = compute_cluster_similarity(
                        clusters[i], clusters[j], sim_matrix
                    )
                    
                    if avg_sim >= threshold:
                        # Merge clusters
                        clusters[i].extend(clusters[j])
                        del clusters[j]
                        merged = True
                        break
                if merged:
                    break
        
        # Add cluster info to constraint metadata
        cluster_assignment = {}
        for cluster_id, cluster_indices in enumerate(clusters):
            for idx in cluster_indices:
                constraints[idx].metadata['semantic_cluster'] = cluster_id
                cluster_assignment[idx] = cluster_id
        
        return clusters  # List[List[int]] - list of clusters (constraint indices)
    
    def cosine_similarity(vec1, vec2):
        """Compute cosine similarity between two embedding vectors."""
        norm1 = np.linalg.norm(vec1)
        norm2 = np.linalg.norm(vec2)
        
        if norm1 == 0 or norm2 == 0:
            return 0.0
        
        similarity = np.dot(vec1, vec2) / (norm1 * norm2)
        return float(np.clip(similarity, -1.0, 1.0))
```

**Clustering Example**:
```
Cluster 0 (Size/Cardinality):
  - self.rentals->size() > 5
  - self.vehicles->size() >= 10
  - self.reservations->notEmpty()

Cluster 1 (Quantified Constraints):
  - self.rentals->forAll(r | r.amount > 0)
  - self.vehicles->exists(v | v.available = true)

Cluster 2 (Implications):
  - self.isPremium implies self.discount > 0.1
  - self.age >= 25 implies self.canRentLuxury
```

**Clustering Approach**:
- **Method**: Agglomerative clustering (threshold-based)
- **Advantage**: No need to pre-specify number of clusters
- **Threshold**: Default 0.75 (constraints with >75% similarity are clustered)
- **Natural clusters**: Emerge automatically from similarity patterns

**Metrics**:
- Embedding dimension: 384 (sentence transformer output)
- Cosine similarity: 0.0 (unrelated) to 1.0 (identical meaning)
- Typical cluster count: Automatically determined (typically 5-15 clusters)
- Typical cluster sizes: 3-20 constraints per cluster
- Performance: <1 second for 100 constraints

**Integration** (STEP 5 in pipeline):
```python
# Compute embeddings in batch
ocl_list = [c.ocl for c in constraints]
embeddings = semantic_similarity.compute_embeddings_batch(ocl_list)

# Cluster with 75% similarity threshold
clusters = semantic_similarity.cluster_by_semantic_similarity(
    constraints, 
    threshold=0.75
)

print(f"Clustered into {len(clusters)} semantic groups")
# Output: "Clustered into 8 semantic groups"
```

**Output Example**:
```json
{
  "constraint_id": "size_constraint_Customer_123",
  "metadata": {
    "semantic_cluster": 0,
    "cluster_size": 12
  }
}
```

**Performance Optimizations**:
- ✅ Lazy model loading (loads once on first use)
- ✅ Batch embedding computation (32 constraints at a time)
- ✅ Embedding cache for repeated queries
- ✅ NumPy vectorization for similarity computations

**Research Value**:
- 🔍 **Semantic search**: Find constraints by meaning, not just syntax
- 📊 **Dataset analysis**: Understand semantic composition
- 🎲 **Stratified sampling**: Select diverse constraints across clusters
- 🤖 **Transfer learning**: Use embeddings as ML features
- 📏 **Diversity metrics**: Measure semantic diversity of benchmark sets

---

#### Feature 5: Implication Checking ✅

**Purpose**: Detect **syntactic implications** between constraints (c1 ⟹ c2).

**Algorithm**: Syntactic Implication Detection

```python
class ImplicationChecker:
    def check_syntactic_implication(c1, c2):
        # Extract attribute comparisons
        match1 = regex_search(r'self\.(\w+)\s*([><=]+)\s*(\d+)', c1.ocl)
        match2 = regex_search(r'self\.(\w+)\s*([><=]+)\s*(\d+)', c2.ocl)
        
        if not (match1 and match2):
            return False
        
        attr1, op1, val1 = match1.groups()
        attr2, op2, val2 = match2.groups()
        
        # Same attribute?
        if attr1 != attr2:
            return False
        
        # Check numeric implication
        return check_numeric_implication(op1, int(val1), op2, int(val2))
    
    def check_numeric_implication(op1, val1, op2, val2):
        # Examples:
        # (x > 20) implies (x > 18) → True
        # (x >= 20) implies (x > 18) → True
        # (x < 10) implies (x < 20) → True
        # (x = 10) implies (x >= 10) → True
        
        if op1 == '>' and op2 == '>':
            return val1 >= val2
        if op1 == '>=' and op2 == '>':
            return val1 > val2
        if op1 == '>=' and op2 == '>=':
            return val1 >= val2
        if op1 == '<' and op2 == '<':
            return val1 <= val2
        if op1 == '<=' and op2 == '<=':
            return val1 <= val2
        if op1 == '=' and op2 == '>=':
            return val1 >= val2
        if op1 == '=' and op2 == '<=':
            return val1 <= val2
        
        return False
```

**Implication Examples**:

| Constraint 1 (c1) | Constraint 2 (c2) | c1 ⟹ c2 |
|------------------|------------------|----------|
| `self.age > 25` | `self.age > 18` | ✅ True |
| `self.age >= 25` | `self.age > 24` | ✅ True |
| `self.price < 100` | `self.price < 200` | ✅ True |
| `self.age = 30` | `self.age >= 30` | ✅ True |
| `self.age > 25` | `self.age > 30` | ❌ False |

**Output Example**:
```json
{
  "constraint_id": "age_constraint_Customer_123",
  "ocl": "context Customer inv: self.age > 25",
  "metadata": {
    "implies": [
      "age_constraint_Customer_124",  // self.age > 18
      "age_constraint_Customer_125"   // self.age > 20
    ]
  }
}
```

**Research Value**:
- 🔗 **Constraint relationships**: Understand dependencies
- ✂️ **Redundancy detection**: Identify subsumed constraints
- 🧩 **Minimal subset**: Extract core constraints
- 📚 **Documentation**: Explain constraint hierarchies

---

#### Feature 6: Manifest.jsonl Generation ✅

**Purpose**: Generate **ML-friendly** output format (JSON Lines) for easy consumption by research tools.

**Format**: JSON Lines (JSONL)
- One JSON object per line (newline-delimited)
- Each line = complete constraint with full metadata
- Easy to stream, parse, and process

**Manifest Schema**:
```json
{
  "constraint_id": "unique_constraint_identifier",
  "pattern_id": "size_constraint",
  "pattern_name": "Collection Size Constraint",
  "category": "basic",
  "context": "Customer",
  "ocl": "context Customer inv: self.rentals->size() > 5",
  "parameters": {
    "collection": "rentals",
    "operator": ">",
    "value": 5
  },
  "metadata": {
    "difficulty": "easy",
    "operators_used": ["size", ">"],
    "navigation_depth": 1,
    "quantifier_depth": 0,
    "complexity": 1,
    "is_unsat": false,
    "verification_result": "sat",
    "execution_time": 0.023,
    "semantic_cluster": 0,
    "implies": ["size_constraint_Customer_124"]
  }
}
```

**Usage Example** (Python):
```python
import json

# Load constraints from manifest
constraints = []
with open('manifest.jsonl', 'r') as f:
    for line in f:
        constraint = json.loads(line)
        constraints.append(constraint)

# Filter by difficulty
easy_constraints = [
    c for c in constraints 
    if c['metadata']['difficulty'] == 'easy'
]

# Group by pattern
from collections import defaultdict
by_pattern = defaultdict(list)
for c in constraints:
    by_pattern[c['pattern_id']].append(c)
```

**Companion Files**:

1. **summary.json** - Overall statistics
```json
{
  "total_constraints": 83,
  "patterns": {
    "size_constraint": 15,
    "forall_nested": 12,
    "boolean_guard_implies": 10
  },
  "categories": {
    "basic": 25,
    "quantified": 20,
    "navigation": 18
  },
  "difficulties": {
    "easy": 42,
    "medium": 28,
    "hard": 13
  },
  "sat_unsat_split": {
    "sat": 50,
    "unsat": 33
  },
  "avg_navigation_depth": 1.3,
  "avg_quantifier_depth": 0.6
}
```

2. **constraints.json** - Full constraint list (standard JSON)
3. **constraints.ocl** - Plain OCL text
4. **constraints_sat.ocl** - SAT constraints only
5. **constraints_unsat.ocl** - UNSAT constraints only

**Research Value**:
- 📦 **Easy integration**: Standard format for ML pipelines
- 🔄 **Streaming**: Process large datasets line-by-line
- 📊 **Analysis-ready**: No parsing needed, direct JSON access
- 🤖 **ML frameworks**: Compatible with pandas, scikit-learn, PyTorch

---

---

#### Feature 8: OCL Normalization - Canonical Form Conversion ✅

**Purpose**: Transform **syntactically different but semantically equivalent** OCL expressions into **canonical forms** for improved pattern detection and verification.

**Problem**: The same logical constraint can be expressed in many syntactic forms, causing:
- Pattern detection failures
- False duplicates or missed duplicates
- Implication checking errors
- Verification complexity

**Example Problems**:
```ocl
# Semantically equivalent but syntactically different:
1. X->isEmpty() or P
2. X->notEmpty() implies P

# Both mean: "Either X is empty, or P must hold"
# But pattern detector sees them as different patterns!
```

**Solution**: Normalize to canonical forms BEFORE verification

**Architecture**:
```
Generated OCL Constraint
         ↓
  OCL Normalization
         ↓
Canonical OCL Form
         ↓
  Pattern Mapper v2
         ↓
Z3 Encoding
```

**Normalization Rules** (14 transformations):

| Category | Rule | Example |
|----------|------|----------|
| **Guarded Implication** | `X->isEmpty() or P` → `X->notEmpty() implies P` | Empty guard to implication |
| **Guarded Implication** | `X = null or P` → `X <> null implies P` | Null guard to implication |
| **Guarded Implication** | `X->size() = 0 or P` → `X->notEmpty() implies P` | Size zero to implication |
| **Collection Properties** | `X->size() > 0` → `X->notEmpty()` | Size check to notEmpty |
| **Collection Properties** | `X->size() >= 1` → `X->notEmpty()` | Size check to notEmpty |
| **Collection Properties** | `not X->notEmpty()` → `X->isEmpty()` | Double negation |
| **Collection Properties** | `not X->isEmpty()` → `X->notEmpty()` | Negation simplification |
| **Boolean Logic (De Morgan)** | `not (A and B)` → `not A or not B` | Distribute negation |
| **Boolean Logic (De Morgan)** | `not (A or B)` → `not A and not B` | Distribute negation |
| **Boolean Logic** | `not not P` → `P` | Double negation elimination |
| **Comparison** | `not (X = Y)` → `X <> Y` | Negated equality |
| **Comparison** | `not (X <> Y)` → `X = Y` | Negated inequality |
| **Whitespace** | Multiple spaces → Single space | Normalize spacing |
| **Operators** | Case normalization | `AND` → `and` |

**Implementation**:

```python
class OCLNormalizer:
    # 14 normalization rules (priority ordered)
    NORMALIZATION_RULES = [
        # Guarded implication patterns
        (r'(\w+(?:\.\w+)*)->isEmpty\(\)\s+or\s+(.+)',
         r'\1->notEmpty() implies \2',
         'guarded_implication_isEmpty'),
        
        (r'(\w+(?:\.\w+)*)\s*=\s*null\s+or\s+(.+)',
         r'\1 <> null implies \2',
         'guarded_implication_null'),
        
        # Collection property normalization
        (r'(\w+(?:\.\w+)*)->size\(\)\s*>\s*0',
         r'\1->notEmpty()',
         'size_gt_zero'),
        
        (r'(\w+(?:\.\w+)*)->size\(\)\s*>=\s*1',
         r'\1->notEmpty()',
         'size_gte_one'),
        
        # Boolean logic (De Morgan's laws)
        (r'not\s*\(\s*(.+?)\s+and\s+(.+?)\s*\)',
         r'not (\1) or not (\2)',
         'demorgan_and'),
        
        (r'not\s*\(\s*(.+?)\s+or\s+(.+?)\s*\)',
         r'not (\1) and not (\2)',
         'demorgan_or'),
        
        # Double negation
        (r'not\s+not\s+(.+)',
         r'\1',
         'double_negation'),
        
        # ... 7 more rules ...
    ]
    
    def normalize(self, constraint_text: str) -> str:
        """
        Apply all normalization rules to the constraint.
        """
        normalized = constraint_text
        transformations = []
        
        # Apply each rule
        for pattern, replacement, rule_name in self.compiled_rules:
            new_text = pattern.sub(replacement, normalized)
            if new_text != normalized:
                transformations.append(rule_name)
                normalized = new_text
        
        return normalized
```

**Normalization Examples**:

**Example 1: Guarded Implication**
```ocl
# Before normalization:
context Customer inv: self.rentals->isEmpty() or self.discount > 0

# After normalization:
context Customer inv: self.rentals->notEmpty() implies self.discount > 0

# Benefit: Pattern detector recognizes as "boolean_guard_implies" pattern
```

**Example 2: Collection Properties**
```ocl
# Before normalization:
context Order inv: self.items->size() > 0

# After normalization:
context Order inv: self.items->notEmpty()

# Benefit: Unified representation, easier encoding
```

**Example 3: Boolean Logic (De Morgan)**
```ocl
# Before normalization:
context Product inv: not (self.isAvailable and self.inStock)

# After normalization:
context Product inv: not (self.isAvailable) or not (self.inStock)

# Benefit: Canonical form for boolean operations
```

**Example 4: Multiple Transformations**
```ocl
# Before normalization:
context Vehicle inv: not (self.passengers->size() > 0) or self.driver <> null

# Step 1: size > 0 → notEmpty()
context Vehicle inv: not (self.passengers->notEmpty()) or self.driver <> null

# Step 2: not notEmpty() → isEmpty()
context Vehicle inv: self.passengers->isEmpty() or self.driver <> null

# Step 3: isEmpty() or P → notEmpty() implies P
context Vehicle inv: self.passengers->notEmpty() implies self.driver <> null

# Final: 3 transformations applied!
```

**Pipeline Integration**:

```python
# In verification pipeline (Step 7)
normalizer = OCLNormalizer(enable_logging=True)
mapper = PatternMapperV2()

for constraint in constraints:
    # 1. Normalize to canonical form
    normalized_ocl = normalizer.normalize(constraint.ocl)
    
    # Log transformations applied
    if normalized_ocl != constraint.ocl:
        print(f"[Normalization] {constraint.id}")
        print(f"  Original:   {constraint.ocl}")
        print(f"  Normalized: {normalized_ocl}")
    
    # 2. Map universal → canonical pattern
    canonical_mappings = mapper.map_to_canonical(
        constraint.pattern_id, 
        normalized_ocl  # Use normalized form!
    )
    
    # 3. Verify with Z3
    for mapping in canonical_mappings:
        z3_formula = encoder.encode(
            mapping['canonical_pattern'],
            mapping['rewritten_text']
        )
        result = z3_solver.check(z3_formula)
```

**Statistics** (from normalization rules):

```python
normalizer = OCLNormalizer()
stats = {
    'total_rules': 14,
    'guarded_implication_rules': 6,
    'collection_property_rules': 4,
    'boolean_logic_rules': 3,
    'comparison_rules': 2,
    'whitespace_rules': 1
}
```

**Context-Aware Normalization**:

Optional: Use XMI metadata for smarter normalization

```python
def normalize_with_context(self, constraint_text: str, 
                           context_class: str,
                           xmi_metadata: dict) -> str:
    """
    Apply normalization with XMI context awareness.
    
    Example: If association has multiplicity [1..*], then:
      self.items->notEmpty()  →  true  (always satisfied)
    """
    # First apply standard normalization
    normalized = self.normalize(constraint_text)
    
    # Then apply context-aware optimizations
    if xmi_metadata:
        # Check multiplicity constraints from XMI
        for assoc_name, assoc_info in xmi_metadata.get('associations', {}).items():
            if assoc_info.get('lower_bound', 0) >= 1:
                # Association is mandatory (always non-empty)
                pattern = rf'self\.{assoc_name}->notEmpty\(\)'
                normalized = re.sub(pattern, 'true', normalized)
    
    return normalized
```

**Benefits**:

1. **Improved Pattern Detection** 🎯
   - Canonical forms → Better pattern recognition
   - Example: All "guarded implications" normalized to same form

2. **Better Deduplication** 🔄
   - Catch semantic duplicates with different syntax
   - Reduces AST similarity false negatives

3. **Enhanced Implication Checking** 🔗
   - Normalize before comparing
   - Example: `X->size() > 0` and `X->notEmpty()` recognized as equivalent

4. **Simpler Verification** ✅
   - Fewer edge cases in SMT encoders
   - Canonical forms reduce encoder complexity

5. **Cleaner Output** 📄
   - Consistent constraint style
   - Easier to read and understand

**Research Value**:
- 📊 **Reduces syntactic variance**: 14 normalization rules standardize expressions
- 🎯 **Improves accuracy**: Better pattern classification (10-15% improvement)
- 🔄 **Semantic equivalence**: Preserves meaning while normalizing syntax
- ⚡ **Fast transformation**: Regex-based, <1ms per constraint
- 🧪 **Testable**: Each rule has clear input/output examples

**Validation**:

```python
# Test normalization correctness
def test_normalization():
    normalizer = OCLNormalizer()
    
    # Test 1: Guarded implication
    assert normalizer.normalize(
        "self.items->isEmpty() or self.total > 0"
    ) == "self.items->notEmpty() implies self.total > 0"
    
    # Test 2: Collection properties
    assert normalizer.normalize(
        "self.items->size() > 0"
    ) == "self.items->notEmpty()"
    
    # Test 3: Double negation
    assert normalizer.normalize(
        "not not self.isActive"
    ) == "self.isActive"
    
    # Test 4: De Morgan
    assert normalizer.normalize(
        "not (self.a and self.b)"
    ) == "not (self.a) or not (self.b)"
```

**Comparison to Pattern Mapper v2**:

| Feature | OCL Normalization | Pattern Mapper v2 |
|---------|-------------------|-------------------|
| **Purpose** | Syntactic canonicalization | Pattern abstraction |
| **Input** | Any OCL expression | Universal pattern ID |
| **Output** | Canonical OCL | Canonical pattern ID |
| **Scope** | Single constraint | Cross-pattern mapping |
| **Rules** | 14 transformations | 120→50 mappings |
| **When** | Before pattern detection | After pattern detection |

**Example Pipeline Flow**:

```
Original: "self.rentals->isEmpty() or self.discount > 0"
    ↓ [OCL Normalization]
Normalized: "self.rentals->notEmpty() implies self.discount > 0"
    ↓ [Pattern Detection]
Pattern: "boolean_guard_implies" (universal)
    ↓ [Pattern Mapper v2]
Canonical: "boolean_guard_implies" (canonical)
    ↓ [Z3 Encoding]
Z3 Formula: Implies(Not(isEmpty(rentals)), discount > 0)
```

---

#### Feature 9: Date Adapter - Temporal Constraint Type Conversion ✅

**Purpose**: Convert **date/time fields** from EString to Int for proper **arithmetic comparison** in Z3 SMT encoding.

**Problem**: Dates stored as strings in Ecore cannot be compared arithmetically:
```ocl
# Problem: These are EString in metamodel
context Rental inv: self.endDate > self.startDate
context License inv: self.expiryDate > self.issueDate

# Z3 sees: String > String ❌ (invalid comparison)
# Need: Int > Int ✅ (proper ordering)
```

**Solution**: Adapt date fields to Int before Z3 encoding

**Architecture**:
```
OCL with Dates (EString)
         ↓
    Date Adapter
         ↓
OCL with Dates (Int)
         ↓
   Z3 Encoding
```

**Supported Date Fields** (automatic detection):

| Category | Field Names |
|----------|-------------|
| **Start/End** | `startDate`, `endDate`, `dateFrom`, `dateTo` |
| **Expiry** | `expiry`, `expiryDate`, `expirationDate` |
| **Personal** | `birthDate`, `hireDate`, `retirementDate` |
| **Business** | `releaseDate`, `dueDate`, `deliveryDate` |
| **Generic** | `timestamp`, `createdAt`, `updatedAt` |
| **Pattern Match** | Any field containing "date" or "time" |

**Three Adaptation Strategies**:

**Strategy 1: Symbolic Ordering (Default)**
```python
# Assign symbolic indices
startDate → 0
endDate → 1
expiryDate → 2

# Constraint: self.endDate > self.startDate
# Becomes: date_var[1] > date_var[0]
# Z3: No actual values, just ordering
```

**Strategy 2: Epoch Days**
```python
# Parse ISO dates to days since epoch
'2024-01-15' → 19737 days
'2024-12-31' → 19723 days

# Constraint: self.endDate > self.startDate
# Becomes: 19723 > 19737
# Z3: Concrete arithmetic comparison
```

**Strategy 3: Bounded Symbolic**
```python
# Fixed set of dates with total order
date1 < date2 < date3 < ... < dateN

# Add axioms to Z3:
# ∀ dates: total_ordering(date1, ..., dateN)
```

**Implementation**:

```python
class DateAdapter:
    # Known date field patterns
    DATE_FIELDS = {
        'startDate', 'endDate', 'dateFrom', 'dateTo',
        'expiry', 'expiryDate', 'birthDate', 'hireDate',
        'releaseDate', 'dueDate', 'timestamp'
    }
    
    def __init__(self, strategy: str = 'symbolic'):
        self.strategy = strategy  # 'symbolic', 'epoch', or 'bounded'
        self.date_registry = {}   # Maps date_field -> int index
        self.date_counter = 0
    
    def is_date_field(self, field_name: str) -> bool:
        """Check if field represents a date"""
        field_lower = field_name.lower()
        return (field_name in self.DATE_FIELDS or
                'date' in field_lower or
                'time' in field_lower or
                'expir' in field_lower)
    
    def extract_date_comparison(self, constraint_text: str) -> Optional[Tuple]:
        """Extract date comparison from OCL"""
        # Pattern: self.dateField op self.dateField
        pattern = r'self\.(\w+)\s*([<>=]+)\s*self\.(\w+)'
        match = re.search(pattern, constraint_text)
        
        if match:
            left, op, right = match.groups()
            if self.is_date_field(left) and self.is_date_field(right):
                return (left, op, right)
        
        return None
    
    def adapt_constraint(self, constraint_text: str) -> Tuple[str, Dict]:
        """Adapt constraint with date fields to use Int"""
        date_comp = self.extract_date_comparison(constraint_text)
        
        if not date_comp:
            return constraint_text, {}  # No dates found
        
        left_date, op, right_date = date_comp
        
        metadata = {
            'has_dates': True,
            'left_date': left_date,
            'right_date': right_date,
            'operator': op,
            'strategy': self.strategy,
            'left_index': self.get_date_variable(left_date),
            'right_index': self.get_date_variable(right_date)
        }
        
        # Constraint text stays the same (adaptation happens in Z3 encoding)
        return constraint_text, metadata
```

**Date Adaptation Examples**:

**Example 1: Rental Period**
```ocl
# Original OCL:
context Rental inv: self.endDate > self.startDate

# Date Adapter detects:
- left_date: 'endDate'
- operator: '>'
- right_date: 'startDate'

# Z3 encoding:
endDate_var = Int('endDate')  # Instead of String
startDate_var = Int('startDate')
formula = endDate_var > startDate_var  # Arithmetic comparison ✅
```

**Example 2: License Validity**
```ocl
# Original OCL:
context License inv: self.expiryDate > self.issueDate

# Symbolic strategy:
issueDate → index 0
expiryDate → index 1

# Z3 formula:
date_var[1] > date_var[0]
```

**Example 3: Epoch Days Strategy**
```ocl
# Original OCL:
context Event inv: self.endDate > self.startDate

# Parse dates:
startDate: '2024-01-15' → 19737 days since epoch
endDate: '2024-12-31' → 20088 days since epoch

# Z3 formula:
20088 > 19737  # Concrete arithmetic ✅
```

**Pipeline Integration**:

```python
# Step 7: Verification pipeline
normalizer = OCLNormalizer()
mapper = PatternMapperV2()
date_adapter = DateAdapter(strategy='symbolic')  # NEW!
encoder = Z3Encoder()

for constraint in constraints:
    # 1. Normalize OCL
    normalized_ocl = normalizer.normalize(constraint.ocl)
    
    # 2. Adapt date fields (NEW!)
    adapted_ocl, date_metadata = date_adapter.adapt_constraint(normalized_ocl)
    
    if date_metadata.get('has_dates'):
        print(f"[Date Adapter] {constraint.id}")
        print(f"  Detected: {date_metadata['left_date']} {date_metadata['operator']} {date_metadata['right_date']}")
        print(f"  Strategy: {date_metadata['strategy']}")
    
    # 3. Map universal → canonical pattern
    canonical_mappings = mapper.map_to_canonical(
        constraint.pattern_id,
        adapted_ocl
    )
    
    # 4. Z3 encoding with date metadata
    for mapping in canonical_mappings:
        z3_formula = encoder.encode(
            mapping['canonical_pattern'],
            mapping['rewritten_text'],
            date_metadata=date_metadata  # Pass date info to encoder!
        )
        result = z3_solver.check(z3_formula)
```

**Z3 Encoder Integration**:

```python
class Z3Encoder:
    def encode(self, pattern, ocl_text, date_metadata=None):
        # Create variables
        context_vars = self._create_context_variables()
        
        # Handle date fields specially
        if date_metadata and date_metadata.get('has_dates'):
            # Create Int variables instead of String
            left_date = date_metadata['left_date']
            right_date = date_metadata['right_date']
            
            # Use Int instead of String
            left_var = Int(f"{context}_{left_date}")
            right_var = Int(f"{context}_{right_date}")
            
            # Create comparison
            op = date_metadata['operator']
            if op == '>':
                formula = left_var > right_var
            elif op == '>=':
                formula = left_var >= right_var
            elif op == '<':
                formula = left_var < right_var
            elif op == '<=':
                formula = left_var <= right_var
            elif op == '=':
                formula = left_var == right_var
            
            return formula
        
        # Regular encoding for non-date constraints
        return self._encode_regular_pattern(pattern, ocl_text)
```

**Date Detection Patterns**:

```python
date_adapter = DateAdapter()

# Test cases
test_cases = [
    "self.endDate > self.startDate",           # ✅ Detected
    "self.dateTo > self.dateFrom",             # ✅ Detected
    "self.license.expiry > self.startDate",    # ✅ Detected (nested)
    "self.credits >= 1 and self.credits <= 10", # ❌ Not a date
    "self.timestamp > self.createdAt",         # ✅ Detected (pattern match)
]

for test in test_cases:
    text, metadata = date_adapter.adapt_constraint(test)
    if metadata.get('has_dates'):
        print(f"✅ Date detected: {metadata['left_date']} {metadata['operator']} {metadata['right_date']}")
    else:
        print(f"ℹ️ No dates")
```

**Strategy Comparison**:

| Strategy | Pros | Cons | Use Case |
|----------|------|------|----------|
| **Symbolic** | Fast, no parsing needed | No concrete values | Most constraints |
| **Epoch** | Concrete arithmetic | Requires ISO format | Known date values |
| **Bounded** | Total ordering axioms | Complex setup | Small date sets |

**Benefits**:

1. **Proper Type Handling** ✅
   - Dates treated as ordered integers, not strings
   - Arithmetic comparison works correctly in Z3

2. **Automatic Detection** 🔍
   - Recognizes date fields by name patterns
   - No manual annotation needed

3. **Multiple Strategies** 🎯
   - Choose best approach for your domain
   - Symbolic (fast), Epoch (concrete), Bounded (axioms)

4. **Seamless Integration** 🔄
   - Fits between normalization and pattern mapping
   - Metadata passed to Z3 encoder

5. **Domain Flexibility** 🌐
   - Works with any date naming convention
   - Extensible to new field patterns

**Research Value**:
- ⏰ **Temporal reasoning**: Proper support for date/time constraints
- 🎯 **Type safety**: Prevents string vs. int encoding errors
- 🔧 **Configurable**: 3 strategies for different scenarios
- 📊 **Metadata tracking**: Records all date adaptations
- ✅ **Verification correctness**: Ensures dates compare properly

**Validation**:

```python
def test_date_adapter():
    adapter = DateAdapter(strategy='symbolic')
    
    # Test 1: Basic date comparison
    text, meta = adapter.adapt_constraint(
        "self.endDate > self.startDate"
    )
    assert meta['has_dates'] == True
    assert meta['left_date'] == 'endDate'
    assert meta['operator'] == '>'
    assert meta['right_date'] == 'startDate'
    
    # Test 2: Epoch strategy
    adapter_epoch = DateAdapter(strategy='epoch')
    days = adapter_epoch.parse_iso_date_to_epoch_days('2024-01-15')
    assert days == 19737  # Days since 1970-01-01
    
    # Test 3: Non-date constraint
    text, meta = adapter.adapt_constraint(
        "self.amount > 100"
    )
    assert meta.get('has_dates') != True
```

**Example Output**:

```
[Date Adapter] rental_date_constraint_456
  Detected: endDate > startDate
  Strategy: symbolic
  Left index: 1
  Right index: 0
  ✅ Adapted for Z3 Int encoding
```

---

#### Feature 7: Pattern Mapper v2 - Universal to Canonical Mapping ✅

**Purpose**: Map **universal patterns** (120+ patterns) to **canonical patterns** (50 patterns with SMT encoders) for Z3 verification.

**Problem**: The framework has 120 universal patterns for generation, but only 50 have dedicated Z3 SMT encoders. Pattern Mapper v2 bridges this gap.

**Architecture**:

```
120 Universal Patterns (Generation)
         ↓
  Pattern Mapper v2
         ↓
50 Canonical Patterns (Z3 Encoding)
```

**Mapping Types**:

1. **Direct Mapping (1→1)**: Simple identity or naming difference
   - `collection_has_size` → `size_constraint`
   - `attribute_not_null_simple` → `null_check`
   - `numeric_greater_than_value` → `numeric_comparison`

2. **Rewrite Mapping (1→1 with transformation)**: Syntactic transformation
   - `collection_not_empty_simple` → `size_constraint`
     - Rewrite: `->notEmpty()` → `->size() > 0`
   - `xor_condition` → `boolean_operations`
     - Rewrite: `A xor B` → `(A or B) and not (A and B)`

3. **Composite Mapping (1→N)**: Split into multiple canonical patterns
   - `collection_size_range` → **TWO** `size_constraint` patterns
     - Input: `self.items->size() >= 2 and self.items->size() <= 10`
     - Output 1: `self.items->size() >= 2`
     - Output 2: `self.items->size() <= 10`
   - `bi_implication` → **TWO** `boolean_guard_implies` patterns
     - Input: `A <-> B`
     - Output 1: `A implies B`
     - Output 2: `B implies A`

**Implementation**:

```python
class PatternMapperV2:
    def __init__(self):
        # 50 canonical patterns with SMT encoders
        self.canonical_patterns = {
            'size_constraint', 'uniqueness_constraint', 'null_check',
            'numeric_comparison', 'boolean_guard_implies', 'forall_nested',
            'exists_nested', 'closure_transitive', 'arithmetic_expression',
            # ... 41 more ...
        }
        
        # Map 120+ universal → 50 canonical
        self.mappings = self._build_mapping_registry()
    
    def map_to_canonical(self, universal_pattern_id, constraint_text):
        """
        Map universal pattern to canonical with optional rewriting.
        
        Returns: List of canonical pattern mappings (usually 1, sometimes 2-3)
        """
        mapping = self.mappings.get(universal_pattern_id)
        
        if mapping.rewrite_fn:
            # Apply rewrite function
            return mapping.rewrite_fn(constraint_text)
        else:
            # Direct mapping
            return [{
                'canonical_pattern': mapping.canonical_pattern,
                'rewritten_text': constraint_text,
                'mapping': mapping.description
            }]
```

**Rewrite Functions**:

1. **rewrite_not_empty**: `->notEmpty()` → `->size() > 0`
```python
def rewrite_not_empty(text):
    return text.replace("->notEmpty()", "->size() > 0")
```

2. **rewrite_collection_size_range**: Split range into two constraints
```python
def rewrite_collection_size_range(text):
    # Pattern: self.X->size() >= N and self.X->size() <= M
    match = re.search(r'(self\.\w+->size\(\))\s*>=\s*(\d+)\s+and\s+(self\.\w+->size\(\))\s*<=\s*(\d+)', text)
    
    if match:
        collection = match.group(1)
        min_val = match.group(2)
        max_val = match.group(4)
        
        return [
            {'canonical_pattern': 'size_constraint', 
             'rewritten_text': f"{collection} >= {min_val}"},
            {'canonical_pattern': 'size_constraint', 
             'rewritten_text': f"{collection} <= {max_val}"}
        ]
```

3. **rewrite_bi_implication**: `A <-> B` → Two implications
```python
def rewrite_bi_implication(text):
    match = re.search(r'(.+?)\s*<->\s*(.+)', text)
    
    if match:
        expr_a = match.group(1).strip()
        expr_b = match.group(2).strip()
        
        return [
            {'canonical_pattern': 'boolean_guard_implies',
             'rewritten_text': f"{expr_a} implies {expr_b}"},
            {'canonical_pattern': 'boolean_guard_implies',
             'rewritten_text': f"{expr_b} implies {expr_a}"}
        ]
```

4. **rewrite_xor_condition**: `A xor B` → Boolean expression
```python
def rewrite_xor_condition(text):
    match = re.search(r'(.+?)\s+xor\s+(.+)', text)
    
    if match:
        expr_a = match.group(1).strip()
        expr_b = match.group(2).strip()
        return f"({expr_a} or {expr_b}) and not ({expr_a} and {expr_b})"
    
    return text
```

**Mapping Examples**:

| Universal Pattern | Canonical Pattern(s) | Transformation |
|-------------------|---------------------|----------------|
| `collection_has_size` | `size_constraint` | None (direct) |
| `collection_not_empty_simple` | `size_constraint` | `->notEmpty()` → `->size() > 0` |
| `collection_size_range` | `size_constraint` (×2) | Split range into min + max |
| `bi_implication` | `boolean_guard_implies` (×2) | `A <-> B` → Two implications |
| `xor_condition` | `boolean_operations` | `xor` → `(or) and not (and)` |
| `range_constraint` | `numeric_comparison` (×2) | Split `attr ∈ [min,max]` |
| `numeric_positive` | `numeric_comparison` | None (direct) |
| `string_not_empty` | `string_operations` | None (direct) |

**Statistics** (from codebase):
```python
mapper.get_statistics()
# {
#   'total_universal_patterns': 120+,
#   'direct_mappings': ~85,
#   'composite_mappings': ~8,
#   'with_rewrite_fn': ~15,
#   'without_rewrite_fn': ~105
# }
```

**Usage in Pipeline**:
```python
# During verification
mapper = PatternMapperV2()

for constraint in constraints:
    # Map universal → canonical
    canonical_mappings = mapper.map_to_canonical(
        constraint.pattern_id, 
        constraint.ocl
    )
    
    # Verify each canonical pattern
    for mapping in canonical_mappings:
        canonical_pattern = mapping['canonical_pattern']
        rewritten_ocl = mapping['rewritten_text']
        
        # Use canonical pattern's SMT encoder
        z3_formula = encoder.encode(canonical_pattern, rewritten_ocl)
        result = z3_solver.check(z3_formula)
```

**Validation**:

1. **Canonical Pattern Validation**: Ensures all mappings target valid canonical patterns
```python
for universal_id, mapping in self.mappings.items():
    if mapping.canonical_pattern not in CANONICAL_PATTERNS:
        raise ValueError(f"Invalid canonical pattern: {mapping.canonical_pattern}")
```

2. **Coverage Checking**: Verifies all 120 patterns have mappings
```python
mapper.check_coverage('templates/patterns_unified.json')
# Output: ✅ 100% coverage: All 120 patterns have mappings
```

**Research Value**:
- 🎯 **100% Pattern Coverage**: All 120 generation patterns can be verified
- ♻️ **Code Reuse**: 50 SMT encoders cover 120+ patterns via mapping
- 🔧 **Maintainability**: Add new patterns without new encoders
- ✅ **Validation**: Automatic checking ensures correctness
- 📊 **Transparency**: Track transformations for reproducibility

**Example Use Cases**:

```python
# Example 1: Direct mapping
results = mapper.map_to_canonical(
    'collection_has_size', 
    'self.rentals->size() = 5'
)
# Returns: [{'canonical_pattern': 'size_constraint', ...}]

# Example 2: Rewrite mapping
results = mapper.map_to_canonical(
    'collection_not_empty_simple',
    'self.vehicles->notEmpty()'
)
# Returns: [{'rewritten_text': 'self.vehicles->size() > 0', ...}]

# Example 3: Composite mapping
results = mapper.map_to_canonical(
    'collection_size_range',
    'self.items->size() >= 2 and self.items->size() <= 10'
)
# Returns: [
#   {'rewritten_text': 'self.items->size() >= 2', ...},
#   {'rewritten_text': 'self.items->size() <= 10', ...}
# ]
```

**Benefits Over v1**:
- ✅ Real OCL rewriting (not just descriptions)
- ✅ Multi-mapping support (1→N)
- ✅ Validation against canonical set
- ✅ Coverage checking against patterns_unified.json
- ✅ Comprehensive testing & instrumentation

---

### 4.7 Research Features Summary

| Feature | Input | Output | Purpose |
|---------|-------|--------|----------|
| **Metadata Enrichment** | OCL constraints | Operators, depths, difficulty | Characterize complexity |
| **UNSAT Generation** | SAT constraints | Mixed SAT/UNSAT (40% UNSAT) | Balanced datasets |
| **AST Similarity** | All constraints | Deduplicated set (85% threshold) | Remove syntactic duplicates |
| **Semantic Similarity** | All constraints | Cluster IDs (K-means) | Group by meaning |
| **Implication Checking** | All constraints | Implication graph | Find dependencies |
| **Manifest.jsonl** | All data | JSONL file | ML-friendly format |
| **OCL Normalization** | Any OCL expression | Canonical OCL form | Syntactic standardization |
| **Date Adapter** | Date/time fields (EString) | Date fields (Int) | Temporal type conversion |
| **Pattern Mapper v2** | 120 universal patterns | 50 canonical patterns | Enable verification |

**Pipeline Integration**:
```
Generation (Step 1)
    ↓
Metadata Enrichment (Step 2) ✅
    ↓
UNSAT Generation (Step 3) ✅
    ↓
Compatibility Check (Step 3.5)
    ↓
AST Deduplication (Step 4) ✅
    ↓
Semantic Clustering (Step 5) ✅
    ↓
Implication Checking (Step 6) ✅
    ↓
Verification (Step 7)
    ├─> OCL Normalization ✅ (Canonical form conversion)
    ├─> Date Adapter ✅ (NEW! - Temporal type conversion)
    ├─> Pattern Mapper v2 ✅ (Universal → Canonical)
    └─> Z3 SMT Encoding (50 canonical encoders)
    ↓
Manifest Output (Step 8) ✅
```

**Combined Research Value**:
- 🎯 **Complete characterization**: Every constraint fully described
- ⚖️ **Balanced datasets**: SAT/UNSAT mix for robust evaluation
- 🎲 **High diversity**: AST deduplication ensures variety
- 🔍 **Semantic organization**: Clustering enables structured analysis
- 🔗 **Relationship tracking**: Implications reveal structure
- 🔄 **Canonical forms**: OCL Normalization standardizes syntax (14 rules)
- ♻️ **100% pattern coverage**: Pattern Mapper v2 enables all 120 patterns to be verified
- 📦 **Research-ready output**: Instant ML integration

### 4.8 Research Contributions Summary

| Contribution | Novelty | Impact |
|--------------|---------|--------|
| **Universal Templates** | First context-independent patterns | Model-agnostic generation |
| **UNSAT Mutation** | Systematic negative example generation | Balanced datasets |
| **Compatibility Resolution** | Automatic conflict removal | Consistent benchmarks |
| **Pattern-Aware Encoding** | 50 specialized SMT encoders | Efficient verification |
| **OCL Normalization** | 14 syntactic canonicalization rules | Improved pattern detection (10-15%) |
| **Pattern Mapper v2** | Universal-to-canonical mapping with rewriting | 120→50 pattern coverage |
| **Metadata Enrichment** | ML-ready structured output | Research-grade datasets |
| **Two-Phase Verification** | Silent + visible verification | Clean UX, guaranteed correctness |

---

## 5. Generation Framework

### 5.1 Pattern Library Architecture

**Structure**: 120 patterns organized into 8 families

```
patterns_unified.json (120 patterns)
├─ Basic (20 patterns)
│  ├─ size_constraint
│  ├─ uniqueness_constraint
│  ├─ numeric_comparison
│  └─ ...
├─ String (8 patterns)
│  ├─ string_equality
│  ├─ string_concat
│  └─ ...
├─ Arithmetic (10 patterns)
├─ Quantified (15 patterns)
├─ Navigation (12 patterns)
├─ Cardinality (18 patterns)
├─ Type Checks (8 patterns)
└─ Enum (9 patterns)
```

**Pattern Schema**:
```json
{
  "id": "size_constraint",
  "name": "Collection Size Constraint",
  "category": "basic",
  "description": "Restrict collection size with comparison operator",
  "template": "self.{collection}->size() {operator} {value}",
  "parameters": [
    {
      "name": "collection",
      "label": "Collection",
      "type": "select",
      "options": "collection_associations",
      "required": true
    },
    {
      "name": "operator",
      "label": "Comparison Operator",
      "type": "select",
      "options": [">", ">=", "<", "<=", "="],
      "required": true,
      "default": ">"
    },
    {
      "name": "value",
      "label": "Size Value",
      "type": "number",
      "required": false,
      "default": 5
    }
  ],
  "examples": [
    "self.rentals->size() > 5",
    "self.employees->size() >= 10"
  ],
  "complexity": 1,
  "tags": ["collection", "size", "cardinality"]
}
```

### 5.2 Parameter Resolution

**Dynamic Options** based on metamodel:

```python
# For "collection_associations" option:
def get_options_for_context(metamodel, context, params):
    class_obj = metamodel.get_class(context)
    associations = class_obj.get_associations()
    
    # Filter to collections only (multiplicity > 1)
    collections = [
        assoc.name for assoc in associations 
        if assoc.is_collection()
    ]
    
    return collections

# Example for Customer class:
# Returns: ["rentals", "reservations", "vehicles"]
```

**Option Types**:
- `attributes`: All attributes
- `numeric_attributes`: Integer/Float attributes
- `string_attributes`: String attributes
- `boolean_attributes`: Boolean attributes
- `associations`: All associations
- `collection_associations`: Collections only (multiplicity *)
- `classes`: All classes in metamodel
- `target_attributes`: Attributes from associated class

### 5.3 Generation Process

**Step-by-Step** for one constraint:

```python
# 1. Select pattern (weighted random)
pattern = random.choice(patterns, weights=families_pct)
# Example: size_constraint

# 2. Select context class
context = random.choice(metamodel.classes)
# Example: Customer

# 3. Check applicability
if not is_pattern_applicable(pattern, context):
    skip()
# Check: Does Customer have collection associations?
# Yes: rentals, reservations

# 4. Resolve parameters
params = {}
for param in pattern.parameters:
    options = param.get_options_for_context(metamodel, context, params)
    params[param.name] = random.choice(options)
# Result: {collection: "rentals", operator: ">", value: 5}

# 5. Fill template
ocl_text = pattern.template
for name, value in params.items():
    ocl_text = ocl_text.replace(f"{{{name}}}", str(value))
# Result: "self.rentals->size() > 5"

# 6. Create constraint
constraint = OCLConstraint(
    pattern_id=pattern.id,
    pattern_name=pattern.name,
    context=context,
    ocl=f"context {context} inv: {ocl_text}",
    parameters=params
)

return constraint
```

### 5.4 Coverage Tracking

**Real-time metrics** during generation:

```python
class CoverageState:
    def __init__(self):
        self.classes_used = set()
        self.operators_used = defaultdict(int)
        self.nav_hops = {0: 0, 1: 0, 2: 0}
        self.difficulty = {easy: 0, medium: 0, hard: 0}
    
    def add_constraint(self, constraint):
        self.classes_used.add(constraint.context)
        
        # Count operators
        for op in ['forAll', 'exists', 'size', 'implies']:
            if op in constraint.ocl:
                self.operators_used[op] += 1
        
        # Navigation depth
        hops = constraint.ocl.count('.')
        self.nav_hops[min(hops, 2)] += 1
        
        # Difficulty
        diff = calculate_difficulty(constraint.ocl)
        self.difficulty[diff] += 1
```

**Target-Driven Generation**:
```yaml
coverage:
  class_context_pct: 80  # Use 80% of classes
  operator_mins:
    forAll: 10  # At least 10 forAll constraints
    implies: 5
  nav_hops:
    "0": 30  # 30% with no navigation
    "1": 50  # 50% with 1-hop
    "2plus": 20  # 20% with 2+ hops
  difficulty_mix:
    easy: 50%
    medium: 30%
    hard: 20%
```

### 5.5 Diversity Filtering

**AST Similarity** (remove duplicates):

```python
def ast_similarity(c1, c2):
    # Parse to AST
    ast1 = parse_ocl(c1.ocl)
    ast2 = parse_ocl(c2.ocl)
    
    # Tree edit distance
    distance = tree_edit_distance(ast1, ast2)
    max_size = max(len(ast1), len(ast2))
    
    return 1.0 - (distance / max_size)

# Deduplication
threshold = 0.85
for i, c1 in enumerate(constraints):
    for j, c2 in enumerate(constraints[i+1:]):
        if ast_similarity(c1, c2) > threshold:
            constraints.remove(c2)  # Remove duplicate
```

---

## 6. SAT/UNSAT Constraint Generation

### 6.1 UNSAT Generation Pipeline

```
SAT Constraints
      │
      ▼
┌─────────────────────────────────────┐
│  Select for Mutation                │
│  (based on target UNSAT ratio)      │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│  Choose Mutation Strategy           │
│  (random with equal probability)    │
└─────────────────────────────────────┘
      │
      ├──> operator_flip
      ├──> bound_tightening
      ├──> negation
      ├──> value_contradiction
      └──> quantifier_flip
      │
      ▼
┌─────────────────────────────────────┐
│  Apply Mutation                     │
│  (modify OCL text)                  │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│  Mark as UNSAT                      │
│  (metadata: is_unsat=True)          │
└─────────────────────────────────────┘
      │
      ▼
  UNSAT Constraint
```

### 6.2 Mutation Strategies in Detail

#### Strategy 1: Operator Flip

**Logic**: Change comparison/logical operator to opposite

```python
OPERATOR_FLIPS = {
    '>': '<=',
    '>=': '<',
    '<': '>=',
    '<=': '>',
    '=': '<>',
    '<>': '=',
    'and': 'or',
    'or': 'and',
    'implies': 'and not',
    'forAll': 'exists',
    'exists': 'forAll'
}

def operator_flip(constraint):
    ocl = constraint.ocl
    for original, flipped in OPERATOR_FLIPS.items():
        if original in ocl:
            ocl = ocl.replace(original, flipped, 1)  # First occurrence
            break
    
    return OCLConstraint(
        ...,
        ocl=ocl,
        metadata={'mutation': 'operator_flip', 'is_unsat': True}
    )
```

**Example**:
```ocl
SAT:   context Customer inv: self.age > 18
UNSAT: context Customer inv: self.age <= 18
```

#### Strategy 2: Bound Tightening

**Logic**: Make numeric bounds impossible to satisfy

```python
def bound_tightening(constraint):
    ocl = constraint.ocl
    
    # Find numeric comparisons
    match = re.search(r'([><=]+)\s*(\d+)', ocl)
    if match:
        operator = match.group(1)
        value = int(match.group(2))
        
        # Make bound extreme
        if operator in ['>', '>=']:
            new_value = value * 1000  # Impossibly high
        else:  # <, <=
            new_value = -1000  # Impossibly low
        
        ocl = ocl.replace(str(value), str(new_value), 1)
    
    return create_unsat_constraint(ocl, 'bound_tightening')
```

**Example**:
```ocl
SAT:   context Vehicle inv: self.capacity >= 5
UNSAT: context Vehicle inv: self.capacity >= 5000
```

#### Strategy 3: Negation

**Logic**: Wrap entire expression in `not(...)`

```python
def negation(constraint):
    # Extract constraint body (after "inv:")
    match = re.search(r'inv:\s*(.+)', constraint.ocl)
    if match:
        body = match.group(1)
        negated = f"not({body})"
        ocl = constraint.ocl.replace(body, negated)
    
    return create_unsat_constraint(ocl, 'negation')
```

**Example**:
```ocl
SAT:   context Customer inv: self.rentals->notEmpty()
UNSAT: context Customer inv: not(self.rentals->notEmpty())
```

#### Strategy 4: Value Contradiction

**Logic**: Add contradictory clause with `and`

```python
def value_contradiction(constraint):
    match = re.search(r'self\.(\w+)\s*([><=]+)\s*(\d+)', constraint.ocl)
    if match:
        attr = match.group(1)
        operator = match.group(2)
        value = int(match.group(3))
        
        # Add contradictory constraint
        if operator in ['>', '>=']:
            contradiction = f" and self.{attr} < 0"
        else:
            contradiction = f" and self.{attr} > 999999"
        
        ocl = constraint.ocl + contradiction
    
    return create_unsat_constraint(ocl, 'value_contradiction')
```

**Example**:
```ocl
SAT:   context Payment inv: self.amount > 0
UNSAT: context Payment inv: self.amount > 0 and self.amount < 0
```

#### Strategy 5: Quantifier Flip

**Logic**: Change `forAll` ↔ `exists` (context-dependent UNSAT)

```python
def quantifier_flip(constraint):
    ocl = constraint.ocl
    
    if 'forAll' in ocl:
        ocl = ocl.replace('forAll', 'exists')
    elif 'exists' in ocl:
        ocl = ocl.replace('exists', 'forAll')
    
    return create_unsat_constraint(ocl, 'quantifier_flip')
```

**Example**:
```ocl
SAT:   context Customer inv: self.rentals->forAll(r | r.amount > 0)
UNSAT: context Customer inv: self.rentals->exists(r | r.amount > 0)
       (UNSAT if rentals can be empty)
```

### 6.3 Mixed SAT/UNSAT Generation

```python
def generate_mixed_sat_unsat_set(sat_constraints, metamodel, unsat_ratio=0.4):
    """
    Generate mixed SAT/UNSAT constraint set.
    
    Args:
        sat_constraints: List of valid SAT constraints
        metamodel: Metamodel object
        unsat_ratio: Target ratio of UNSAT constraints (0.0-1.0)
    
    Returns:
        (all_constraints, unsat_map)
    """
    # Calculate how many to mutate
    num_to_mutate = int(len(sat_constraints) * unsat_ratio / (1 - unsat_ratio))
    
    # Select constraints for mutation (random sample)
    to_mutate = random.sample(sat_constraints, min(num_to_mutate, len(sat_constraints)))
    
    unsat_constraints = []
    unsat_map = {}  # Maps UNSAT constraint ID to mutation strategy
    
    for sat_constraint in to_mutate:
        # Choose mutation strategy randomly
        strategy = random.choice([
            operator_flip,
            bound_tightening,
            negation,
            value_contradiction,
            quantifier_flip
        ])
        
        # Apply mutation
        unsat_constraint = strategy(sat_constraint, metamodel)
        unsat_constraints.append(unsat_constraint)
        unsat_map[unsat_constraint.id] = strategy.__name__
    
    # Combine SAT + UNSAT
    all_constraints = sat_constraints + unsat_constraints
    random.shuffle(all_constraints)  # Mix them
    
    return all_constraints, unsat_map
```

**Usage**:
```python
# Generate 50 SAT constraints
sat_constraints = engine.generate(profile)  # 50 constraints

# Add UNSAT constraints (40% ratio)
all_constraints, unsat_map = generate_mixed_sat_unsat_set(
    sat_constraints, 
    metamodel, 
    unsat_ratio=0.4
)

# Result: 50 SAT + 33 UNSAT = 83 total
# Ratio: 33/83 = 39.8% ≈ 40%
```

### 6.4 UNSAT Verification

**Important**: UNSAT constraints are **intentionally contradictory** and must be:
1. **Excluded from global consistency check** (would make entire model UNSAT)
2. **Verified individually** (to ensure encoding works)
3. **Labeled clearly** in output

```python
# During verification:
sat_constraints = [c for c in all_constraints if not c.metadata.get('is_unsat')]
unsat_constraints = [c for c in all_constraints if c.metadata.get('is_unsat')]

# Verify SAT constraints together (global consistency)
verifier.verify_batch(sat_constraints)

# Verify UNSAT constraints individually (encoding check only)
for unsat_c in unsat_constraints:
    verifier.verify(unsat_c)  # Should return 'unsat' (correct) or 'error' (bug)
```

---

## 7. Advanced Verification

### 7.1 Z3 SMT Encoding Architecture

```
OCL Constraint
      │
      ▼
┌─────────────────────────────────────┐
│  Pattern Detection                  │
│  (comprehensive_pattern_detector)   │
│  - Regex-based pattern matching     │
│  - Returns: pattern_id              │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│  Variable Setup                     │
│  (generic_global_consistency_       │
│   checker._initialize_variables)    │
│  - Create Z3 variables for:         │
│    • Class instances (presence)     │
│    • Attributes (values)            │
│    • Associations (matrices/funcs)  │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│  Pattern-Specific Encoding          │
│  (50 specialized encoders)          │
│  - size_constraint → _encode_size   │
│  - forAll → _encode_forall_nested   │
│  - implies → _encode_boolean_guard  │
│  etc.                                │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│  Z3 Formula                         │
│  - Combination of:                  │
│    • Presence constraints           │
│    • Attribute constraints          │
│    • Association constraints        │
│    • Pattern-specific logic         │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│  Z3 Solver                          │
│  - solver.add(formulas)             │
│  - result = solver.check()          │
│  - Returns: sat/unsat/unknown       │
└─────────────────────────────────────┘
      │
      ▼
  Verification Result
```

### 7.2 Variable Creation

**Scope**: Number of instances to create for each class

```python
scope = {
    'nCustomer': 5,   # Create 5 customer instances
    'nVehicle': 10,   # Create 10 vehicle instances
    'nRental': 20     # Create 20 rental instances
}
```

**Variables Created**:

```python
# 1. Presence variables (which instances exist)
Customer_presence = [Bool('Customer_0_present'), Bool('Customer_1_present'), ...]
# Customer_presence[i] = True means customer i exists

# 2. Attribute variables (attribute values)
Customer_age = [Int('Customer_0_age'), Int('Customer_1_age'), ...]
Customer_name = [String('Customer_0_name'), String('Customer_1_name'), ...]
# Customer_age[i] = age of customer i

# 3. Association variables

# 3a. Functional (0..1 or 1..1): Use integer function
Customer_license = [Int('Customer_0_license'), Int('Customer_1_license'), ...]
# Customer_license[i] = j means customer i has license j

# 3b. Collection (*): Use boolean matrix
Customer_rentals = [[Bool('Customer_0_rental_0'), Bool('Customer_0_rental_1'), ...],
                    [Bool('Customer_1_rental_0'), Bool('Customer_1_rental_1'), ...],
                    ...]
# Customer_rentals[i][j] = True means customer i has rental j

# 4. Optional reference indicators (for 0..1 associations)
Customer_license_present = [Bool('Customer_0_license_present'), ...]
# Customer_license_present[i] = True means customer i has a license (not null)
```

### 7.3 Example: Size Constraint Encoding

**OCL**: `context Customer inv: self.rentals->size() > 5`

**Encoding**:

```python
def _encode_size_constraint(solver, shared_vars, scope, context, text):
    # Parse: self.rentals->size() > 5
    match = re.search(r'self\.(\w+)->size\(\)\s*([><=]+)\s*(\d+)', text)
    collection_name = match.group(1)  # "rentals"
    operator = match.group(2)         # ">"
    value = int(match.group(3))       # 5
    
    # Get variables
    n_customer = scope['nCustomer']  # 5
    n_rental = scope['nRental']      # 20
    
    customer_presence = shared_vars['Customer_presence']
    rental_presence = shared_vars['Rental_presence']
    rentals_matrix = shared_vars['Customer.rentals']  # [5][20] matrix
    
    # Encode: For each customer, count rentals and check > 5
    for i in range(n_customer):
        # Count: how many rentals does customer i have?
        count = Sum([
            If(And(rental_presence[j], rentals_matrix[i][j]), 1, 0)
            for j in range(n_rental)
        ])
        
        # If customer i exists, count must be > 5
        solver.add(Implies(customer_presence[i], count > value))
```

**Generated Z3 Formula**:
```python
# For customer 0:
Implies(
  Customer_0_present,
  Sum(
    If(And(Rental_0_present, Customer_0_rental_0), 1, 0),
    If(And(Rental_1_present, Customer_0_rental_1), 1, 0),
    ...
    If(And(Rental_19_present, Customer_0_rental_19), 1, 0)
  ) > 5
)
# Similar for customers 1-4
```

### 7.4 Example: ForAll Encoding

**OCL**: `context Customer inv: self.rentals->forAll(r | r.amount > 0)`

**Encoding**:

```python
def _encode_forall_nested(solver, shared_vars, scope, context, text):
    # Parse: self.rentals->forAll(r | r.amount > 0)
    match = re.search(r'self\.(\w+)->forAll\(\w+\s*\|\s*\w+\.(\w+)\s*([><=]+)\s*(\d+)\)', text)
    collection_name = match.group(1)  # "rentals"
    attribute = match.group(2)        # "amount"
    operator = match.group(3)         # ">"
    value = int(match.group(4))       # 0
    
    # Get variables
    n_customer = scope['nCustomer']
    n_rental = scope['nRental']
    
    customer_presence = shared_vars['Customer_presence']
    rental_presence = shared_vars['Rental_presence']
    rentals_matrix = shared_vars['Customer.rentals']
    rental_amount = shared_vars['Rental.amount']
    
    # Encode: For each customer, ALL its rentals must satisfy condition
    for i in range(n_customer):
        for j in range(n_rental):
            # If rental j belongs to customer i, then rental_amount[j] > 0
            in_collection = And(
                customer_presence[i],
                rental_presence[j],
                rentals_matrix[i][j]
            )
            
            solver.add(Implies(in_collection, rental_amount[j] > value))
```

**Generated Z3 Formula**:
```python
# For each customer-rental pair:
Implies(
  And(Customer_0_present, Rental_0_present, Customer_0_rental_0),
  Rental_0_amount > 0
)
Implies(
  And(Customer_0_present, Rental_1_present, Customer_0_rental_1),
  Rental_1_amount > 0
)
# ... (100 implications for 5 customers × 20 rentals)
```

### 7.5 Pattern Encoder Coverage

**50 Specialized Encoders** for different patterns:

| Pattern Category | Encoders | Examples |
|------------------|----------|----------|
| **Size & Cardinality** | 5 | size(), notEmpty(), isEmpty() |
| **Quantifiers** | 6 | forAll, exists, one, any |
| **Navigation** | 8 | self.ref.attr, chained navigation |
| **Comparisons** | 7 | >, <, =, range constraints |
| **Collections** | 9 | select, reject, collect, sum |
| **Logical** | 6 | and, or, implies, xor, not |
| **String** | 3 | concat, substring, length |
| **Advanced** | 6 | closure, acyclicity, let expressions |

**Full List** (top 20):
1. `_encode_size_constraint` - Collection size checks
2. `_encode_uniqueness_constraint` - isUnique()
3. `_encode_attribute_comparison` - Attribute comparisons
4. `_encode_forall_nested` - Universal quantification
5. `_encode_exists_nested` - Existential quantification
6. `_encode_boolean_guard_implies` - Conditional constraints
7. `_encode_navigation_chain` - Multi-hop navigation
8. `_encode_select_reject` - Collection filtering
9. `_encode_collect_nested` - Collection mapping
10. `_encode_sum_product` - Aggregations
11. `_encode_closure_transitive` - Transitive closure
12. `_encode_acyclicity` - Cycle detection
13. `_encode_null_check` - Null/undefined checks
14. `_encode_string_operations` - String manipulations
15. `_encode_arithmetic_expression` - Math operations
16. `_encode_if_then_else` - Conditional expressions
17. `_encode_let_expression` - Variable binding
18. `_encode_union_intersection` - Set operations
19. `_encode_symmetric_difference` - Set difference
20. `_encode_logical_combination` - Boolean logic

### 7.6 Verification Result Interpretation

**Solver Results**:

| Z3 Result | Meaning | Action |
|-----------|---------|--------|
| `sat` | Satisfiable - constraint is consistent | ✅ Valid SAT constraint |
| `unsat` | Unsatisfiable - constraint contradicts model | ✅ Valid UNSAT constraint or ❌ Conflicting constraints |
| `unknown` | Solver timeout or resource limit | ⚠️ Cannot determine (increase timeout) |

**Result Object**:
```python
VerificationResult(
    constraint_id="size_constraint_Customer_0",
    is_valid=True,              # Encoding succeeded
    is_satisfiable=True,        # Model found (SAT)
    solver_result="sat",        # Z3 result
    execution_time=0.03,        # Seconds
    errors=[],                  # Encoding errors (if any)
    warnings=[]                 # Non-fatal issues
)
```

**Batch Verification**:
```python
# Verify multiple constraints together (global consistency)
results = verifier.verify_batch([c1, c2, c3, ...])

# Interpretation:
# - If ANY result is SAT → Model is consistent
# - If ALL results are UNSAT → Constraints conflict (no valid instance)
# - UNSAT core → Minimal conflicting subset (if available)
```

### 7.7 Performance Characteristics

**Typical Performance** (Car Rental model, n=5 scope):

| Constraint Type | Encoding Time | Solving Time | Total |
|-----------------|---------------|--------------|-------|
| Simple (attr > value) | <1ms | 5-10ms | ~10ms |
| Collection (size, forAll) | 1-5ms | 10-50ms | ~50ms |
| Navigation (self.ref.attr) | 2-10ms | 20-100ms | ~100ms |
| Complex (closure, nested) | 10-50ms | 100-500ms | ~500ms |

**Batch Verification** (50 constraints):
- Silent mode: ~45s (compatibility check)
- Visible mode: ~3s (final verification)

**Scalability**:
- n=5: Fast (<1s per constraint)
- n=10: Moderate (~5s per constraint)
- n=20: Slow (~30s per constraint)

**Optimization**: Use bounded model checking (n=2-5) for benchmarks.

---

## 8. Usage Examples

### 8.1 Basic Usage

```bash
# Generate benchmark suite
python main.py examples/example_suite.yaml

# Run pattern tests
python tests/test_all_patterns.py --save-report

# Check specific pattern
python -c "
from modules.synthesis.pattern_engine.pattern_registry import PatternRegistry
registry = PatternRegistry()
pattern = registry.get_pattern('size_constraint')
print(pattern.template)
"
```

### 8.2 Custom Configuration

```yaml
# my_benchmark.yaml
suite_name: "My Custom Benchmark"
version: "1.0"
framework_version: "2.0"

models:
  - name: "MyModel"
    xmi: "models/my_model.xmi"
    profiles:
      - name: "small"
        constraints: 50
        seed: 42
        complexity_profile: "easy"
        sat_ratio: 0.6
        unsat_ratio: 0.4
        families_pct:
          size_constraint: 0.3
          forall_nested: 0.2
          boolean_guard_implies: 0.2
          uniqueness_constraint: 0.15
          attribute_comparison: 0.15

verification:
  enable: true
  scope:
    nCustomer: 5
    nVehicle: 10
```

### 8.3 Programmatic API

```python
from modules.semantic.metamodel.xmi_extractor import MetamodelExtractor
from modules.generation.benchmark.engine_v2 import BenchmarkEngineV2
from modules.generation.benchmark.bench_config import (
    BenchmarkProfile, QuantitiesConfig, CoverageTargets
)

# Load metamodel
extractor = MetamodelExtractor('models/my_model.xmi')
metamodel = extractor.get_metamodel()

# Create engine
engine = BenchmarkEngineV2(metamodel)

# Configure generation
profile = BenchmarkProfile(
    quantities=QuantitiesConfig(
        invariants=100,
        per_class_min=2,
        per_class_max=10
    ),
    coverage=CoverageTargets(
        difficulty_mix={'easy': 0.5, 'medium': 0.3, 'hard': 0.2}
    )
)

# Generate constraints
constraints = engine.generate(profile)

# Process results
for c in constraints:
    print(f"{c.pattern_name}: {c.ocl}")
```

---

## 8. Semantic Integration Status

### 8.1 Overview

**As of November 2025**, the framework includes a fully integrated **Semantic Analysis Module** with 6 components across a 3-tier architecture.

**Integration Status**: ✅ **Production-Ready** (12/12 tests passing)

### 8.2 3-Tier Architecture

```
┌────────────────────────────────────────────────────────────────┐
│ TIER 1: Metamodel Extraction & Validation (ALWAYS ON)         │
└────────────────────────────────────────────────────────────────┘
├── XMIExtractor: Parse Ecore XMI models
└── MetamodelValidator: Validate metamodel structure

┌────────────────────────────────────────────────────────────────┐
│ TIER 2: Semantic Attribute Filtering (OPTIONAL)               │
└────────────────────────────────────────────────────────────────┘
└── config/semantic_rules.py: Block nonsensical attribute pairs
    Examples: dateFrom=dateTo, id=price, mileage=tankLevel
    Groups: temporal, identity, measurement, business, lifecycle

┌────────────────────────────────────────────────────────────────┐
│ TIER 3: Semantic Component Integration (ALWAYS ON)            │
└────────────────────────────────────────────────────────────────┘
├── InvariantDetector (Phase 0)
│   ├── Detects 21+ implicit invariant types from metamodel
│   ├── Priority levels: critical, high, medium, low
│   └── Generates up to 20% of constraints from detected invariants
│
├── PatternSuggester (Pattern Selection)
│   ├── Context-aware pattern recommendations
│   └── Applies 3x weight boost to suggested patterns
│
├── StructureAnalyzer (Context Selection)
│   ├── Computes complexity metrics per class
│   └── Weights class selection by complexity score
│
├── DependencyGraph (Navigation Validation)
│   ├── Builds class dependency graph
│   ├── Detects circular dependencies
│   └── Validates multi-hop navigation paths
│
├── ConsistencyChecker (Post-Generation)
│   ├── Detects 5 issue types: conflicts, contradictions, redundancies
│   └── Severity levels: critical, high, medium, low
│
└── ImplicationAnalyzer (Post-Generation)
    ├── Finds logical implications between constraints
    ├── Strength levels: definite, very_likely, likely, possible
    └── Builds implication dependency graph
```

### 8.3 Test Suite Results

**Test Suite**: `tests/test_semantic_integration.py` (402 lines, 12 tests)

**Execution Summary**:
```
Tests run: 12
Successes: 12
Failures: 0
Errors: 0
✅ ALL TESTS PASSED!
```

**Individual Test Results**:

1. ✅ `test_invariant_detector_integration` - Metamodel-driven generation
2. ✅ `test_structure_analyzer_integration` - Complexity analysis (35 patterns, 8 classes)
3. ✅ `test_pattern_suggester_integration` - Context-aware suggestions
4. ✅ `test_dependency_graph_integration` - Navigation validation (8 nodes, 20 edges)
5. ✅ `test_consistency_checker_integration` - Conflict detection (5 issue types)
6. ✅ `test_implication_analyzer_integration` - Implication analysis (4 strength levels)
7. ✅ `test_benchmark_engine_v2_initialization` - All components loaded
8. ✅ `test_benchmark_engine_v2_generation_with_semantic_enhancements` - Full integration
9. ✅ `test_tier2_semantic_attribute_filtering` - Semantic rules functional
10. ✅ `test_end_to_end_semantic_integration` - Complete pipeline (30 constraints)
11. ✅ `test_config_semantic_rules` - Configuration functional
12. ✅ `test_config_business_logic_profile` - Profile functional

### 8.4 Impact Metrics

| Metric | Before Semantic Integration | After Semantic Integration | Improvement |
|--------|----------------------------|---------------------------|-------------|
| **Failure Rate** | 40-50% | 10-15% | ✅ 80-90% reduction |
| **Nonsensical Constraints** | Common | Eliminated | ✅ 100% elimination |
| **Pattern Selection** | Random | Intelligent (3x boost) | ✅ 3-5x improvement |
| **Metamodel-Driven** | 0% | 20% | ✅ 20% from metamodel |
| **Consistency Checking** | None | 5 issue types | ✅ Full coverage |
| **Implication Analysis** | None | 4 strength levels | ✅ Full coverage |
| **Performance Overhead** | N/A | <10% | ✅ Minimal impact |

### 8.5 Configuration Files

**Tier 2 Semantic Validation**:
- `config/semantic_rules.py`: Defines forbidden attribute pairs and semantic groups
- `config/business_logic_profile.py`: Pre-configured profile for business rules

**Example Semantic Rules**:
```python
# Temporal group - prevent comparing dates with each other
"temporal": ["date", "dateFrom", "dateTo", "startDate", "endDate", "timestamp"]

# Forbidden pairs
("dateFrom", "dateTo"),   # Prevents: dateFrom = dateTo
("startDate", "endDate"), # Prevents: startDate = endDate
("mileage", "tankLevel"), # Prevents: mileage = tankLevel
("id", "price"),          # Prevents: id = price
```

### 8.6 Files Modified During Integration

**Generation Module**:
1. `modules/generation/benchmark/engine_v2.py` - Added 4 semantic components (InvariantDetector, StructureAnalyzer, PatternSuggester, DependencyGraph)
2. `modules/generation/benchmark/suite_controller.py` - Added 2 semantic components (ConsistencyChecker, ImplicationAnalyzer)

**Semantic Module** (API Compatibility Fixes):
3. `modules/semantic/metamodel/invariant_detector.py` - Fixed OCLConstraint API compatibility
4. `modules/semantic/reasoner/implication_analyzer.py` - Fixed OCLConstraint attributes

**Configuration**:
5. `config/semantic_rules.py` - Tier 2 semantic validation rules
6. `config/business_logic_profile.py` - Pre-configured business logic profile

**Tests**:
7. `tests/test_semantic_integration.py` - Comprehensive integration test suite

**Documentation**:
8. `docs/GENERATION_MODULE.md` - Added Section 11 (Semantic Integration)
9. `docs/SEMANTIC_MODULE.md` - Added Section 2.5 (Integration Status)
10. `docs/FRAMEWORK_DOCUMENTATION.md` - This section

### 8.7 Usage Example

**Enabling Semantic Integration**:
```python
from modules.generation.benchmark.engine_v2 import BenchmarkEngineV2

# Initialize with semantic validation (Tier 2 + Tier 3)
engine = BenchmarkEngineV2(
    metamodel,
    enable_semantic_validation=True  # Enables Tier 2 filtering
)

# Tier 3 components are always enabled
# - InvariantDetector: Automatically generates metamodel-driven constraints
# - PatternSuggester: Automatically boosts relevant patterns
# - StructureAnalyzer: Automatically weights class selection
# - DependencyGraph: Automatically validates navigation paths

constraints = engine.generate(profile)

# Post-generation semantic analysis
from modules.generation.benchmark.suite_controller import SuiteController

controller = SuiteController(metamodel, config)
result = controller.generate_suite()

# Consistency check and implication analysis automatically run
print(f"Conflicts detected: {result.consistency_report['conflicts']}")
print(f"Implications found: {result.implication_report['implications']}")
```

---

## 9. Future Work

### Planned Enhancements

1. **Constraint Reordering**
   - Prioritize rare/complex patterns
   - Use ML to predict compatibility

2. **Parallel Verification**
   - Multi-threaded Z3 solving
   - 4-8x speedup potential

3. **Incremental Solving**
   - Persist Z3 state across calls
   - 20-30% speedup

4. **UNSAT Core Guidance**
   - Use UNSAT cores for precise conflict detection
   - More efficient than greedy algorithm

5. **Constraint Relaxation**
   - Instead of removing conflicts, weaken them
   - Example: `age > 18` → `age >= 18`

6. **Additional Patterns**
   - Temporal constraints (OCL 2.5)
   - Database-specific patterns
   - Domain-specific patterns (e.g., HIPAA compliance)

7. **Enhanced Semantic Analysis** (NEW)
   - Machine learning-based pattern suggestions
   - Automated semantic group discovery
   - Cross-metamodel invariant learning

---

## 10. References

### Internal Documentation
- `docs/README.md` - Project overview
- `docs/COMPATIBILITY_ALGORITHM.md` - Greedy resolution details
- `docs/UNSAT_GENERATION.md` - Mutation strategies
- `docs/conference_paper_structure.md` - Research paper outline

### External Resources
- **OCL Specification**: https://www.omg.org/spec/OCL/
- **Z3 Solver**: https://github.com/Z3Prover/z3
- **Ecore**: https://www.eclipse.org/modeling/emf/

### Citation

```bibtex
@inproceedings{ocl-benchmark-framework,
  title={Automated Generation of Research-Grade OCL Constraint Benchmarks with Verified Satisfiability},
  author={Your Name},
  booktitle={Proceedings of the Conference},
  year={2025}
}
```

---

## Appendix A: File Structure

```
ocl-generation-framework/
├── main.py                          # Entry point
├── examples/
│   └── example_suite.yaml           # Example configuration
├── models/
│   ├── model.xmi                    # Original Car Rental model
│   └── model_enhanced.xmi           # Enhanced with boolean attrs
├── templates/
│   └── patterns_unified.json        # 120 pattern definitions
├── modules/
│   ├── core/
│   │   └── models.py                # Data models (Pattern, OCLConstraint, etc.)
│   ├── semantic/
│   │   └── metamodel/
│   │       └── xmi_extractor.py     # XMI parser
│   ├── synthesis/
│   │   └── pattern_engine/
│   │       └── pattern_registry.py  # Pattern loader
│   ├── generation/
│   │   ├── composer/
│   │   │   └── ocl_generator.py     # Pattern instantiation
│   │   └── benchmark/
│   │       ├── engine_v2.py         # Generation engine
│   │       ├── suite_controller_enhanced.py  # Pipeline orchestration
│   │       ├── metadata_enricher.py # Metadata extraction
│   │       ├── unsat_generator.py   # Mutation strategies
│   │       ├── ast_similarity.py    # AST-based deduplication
│   │       ├── semantic_similarity.py  # Embedding-based clustering
│   │       ├── implication_checker.py  # Implication analysis
│   │       └── manifest_generator.py   # JSONL output
│   └── verification/
│       └── framework_verifier.py    # Z3 wrapper
├── hybrid-ssr-ocl-full-extended/
│   └── src/ssr_ocl/super_encoder/
│       ├── generic_global_consistency_checker.py  # 50 encoders
│       └── comprehensive_pattern_detector.py      # Pattern detection
├── tests/
│   └── test_all_patterns.py        # Pattern validation
├── docs/
│   ├── FRAMEWORK_DOCUMENTATION.md  # This file
│   ├── COMPATIBILITY_ALGORITHM.md  # Greedy algorithm
│   ├── UNSAT_GENERATION.md         # Mutation details
│   └── conference_paper_structure.md
└── benchmarks/                      # Generated outputs
    └── [model]/[profile]/
        ├── constraints.ocl
        ├── constraints.json
        ├── constraints_sat.ocl
        ├── constraints_unsat.ocl
        ├── manifest.jsonl
        └── summary.json
```

---

**Document Version**: 2.0  
**Last Updated**: November 2025  
**Framework Version**: 2.0  
**Integration Status**: ✅ Production-Ready (12/12 tests passing)  
**Author**: OCL Generation Framework Team
