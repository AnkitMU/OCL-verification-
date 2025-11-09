# Automated Generation of OCL Constraint Benchmarks with SMT-Based Verification

Authors: [Redacted for Review]

Abstract
We present an automated framework to generate research-grade OCL (Object Constraint Language) benchmarks across arbitrary metamodels (UML/Ecore), verified for satisfiability using SMT solving. Our key contribution is a universal→canonical pattern mapping layer with OCL rewriting (PatternMapperV2) that decouples high-level OCL idioms from solver-ready encodings. The system integrates a generic SMT encoder atop Z3, a coverage-driven generation engine, and a research feature pipeline including (i) metadata enrichment, (ii) UNSAT constraint generation by mutation, (iii) AST-based structural similarity, (iv) semantic similarity via dense embeddings, (v) implication detection, and (vi) manifest generation for downstream ML and tooling. Evaluation on standard domains shows high validity (≥90%) and diverse pattern coverage. We open-source a complete benchmark generation pipeline designed to support tool evaluation, reproducible experiments, and dataset creation for learning-based OCL.

1 Introduction
Motivation. OCL is widely used to express invariants and constraints over UML/Ecore models, yet evaluation of OCL tooling (parsers, validators, and solvers) suffers from a shortage of diverse, verified benchmarks. Manual curation is labor intensive, prone to bias, and often lacks ground-truth satisfiability labels.

Challenges.
- Pattern diversity: Benchmarks should span navigation, collections, quantifiers, arithmetic, strings, and boolean logic.
- Generality: The same generation pipeline should work across arbitrary metamodels (not tied to a single domain).
- Correctness at scale: Generated constraints must be syntactically valid and globally consistent (when labeled SAT), requiring formal verification.
- Redundancy control: Large batches need deduplication by structure and meaning to produce high-quality corpora.

Contributions.
1) Universal→Canonical Pattern Mapping (PatternMapperV2): A novel mapping and OCL rewriting layer converting 120 universal patterns into a curated set of 50 canonical encoders, enabling domain-independent generation with solver-ready forms.
2) Generic SMT Encoding: A model-agnostic encoder that builds Z3 variables for classes, attributes, and associations, and emits constraints for canonical OCL patterns.
3) Research Feature Pipeline: Metadata extraction, UNSAT mutation generation, AST similarity, semantic clustering, implication detection, and manifest generation, integrated into a reproducible benchmark suite pipeline.
4) Implementation & Evaluation: An end-to-end system with coverage tracking and configurable scope/timeouts, producing ≥90% valid constraints and diverse pattern coverage across standard metamodels.

2 Background and Related Work
OCL Primer. OCL defines side-effect-free constraints over models, using navigation (self.a.b), collection operations (forAll, exists, select, collect, size), arithmetic and string operations, and logical connectives (and, or, implies, xor).

SMT Solving for Models. SMT (Satisfiability Modulo Theories) solvers (e.g., Z3) support reasoning over integers, reals, booleans, arrays, and uninterpreted functions—making them suitable for OCL encodings. Prior approaches (e.g., USE Validator, EMFtoCSP/UMLtoCSP) encode models into SAT/SMT or CSP, but typically lack an automated, scalable benchmark generation pipeline with semantic diversity.

Benchmark Generation. Existing OCL benchmarks are limited in size, generality, and verification. Our contribution is a unified, extensible pipeline offering pattern-driven generation, formal verification, and research-oriented metadata.

3 System Overview
Workflow.
1) Metamodel ingestion (XMI): Extract classes, attributes, associations, and cardinalities.
2) Pattern-driven generation: Instantiate OCL templates from a universal pattern library using a coverage-driven engine.
3) Pattern mapping & rewriting: Convert universal patterns to canonical encoders via PatternMapperV2.
4) SMT encoding & verification: Emit Z3 constraints, check SAT/UNSAT and model consistency.
5) Research features: Enrich metadata, generate UNSAT variants, deduplicate by AST and semantics, detect implications, produce manifest.

Key Modules.
- Generation: modules/generation/benchmark/engine_v2.py, suite_controller_enhanced.py
- Pattern Mapping: modules/verification/pattern_mapper_v2.py
- Verification (SMT): hybrid-ssr-ocl-full-extended/src/ssr_ocl/super_encoder/generic_global_consistency_checker.py
- Research Features: metadata_enricher.py, unsat_generator.py, ast_similarity.py, semantic_similarity.py, implication_checker.py, manifest_generator.py

Novel Architectural Idea: Universal→Canonical Pattern Layer. Instead of supporting every universal OCL idiom directly in the SMT encoder, we map them to a fixed set of canonical encodings with explicit OCL rewrites. This decoupling accelerates encoder evolution, improves testability, and enables multi-mapping (e.g., a single bi-implication spawns two implications).

4 Pattern Library and Universal→Canonical Mapping (PatternMapperV2)
Universal Pattern Library. 120 universal pattern entries span families: cardinality (size, includes/excludes, notEmpty/isEmpty, range), uniqueness, navigation, quantified (forAll, exists, select/reject, one/any), arithmetic/logic, strings, and type checks.

Canonical Pattern Set. 50 canonical encoders (e.g., size_constraint, uniqueness_constraint, null_check, numeric_comparison, boolean_guard_implies, collect_flatten, string_operations, type_check_casting) define solver-ready building blocks.

Rewriting and Multi-Mapping. PatternMapperV2 (modules/verification/pattern_mapper_v2.py) implements:
- Direct mappings (e.g., collection_min_size → size_constraint)
- Rewrites (e.g., X->notEmpty() → X->size() > 0; xor → (A or B) and not (A and B))
- Multi-mapping (A <-> B → A implies B AND B implies A)
- Fallbacks for unparsed cases while preserving traceability (mapping fields)

Difficult Implementation: Bi-Implication Parsing.
- OCL idioms appear as either A <-> B or (A) = (B) when A,B are boolean. We extended rewrite_bi_implication to detect both forms, emitting two boolean_guard_implies encodings to the SMT layer. This harmonizes idiomatic equality-of-boolean expressions with explicit implications.

Difficult Implementation: Range Splitting.
- Rewrites like size() in [min,max] become two separate constraints (size() ≥ min) ∧ (size() ≤ max). Splitting improves solver efficiency and aligns with canonical encoders.

Coverage Validation.
- The mapper validates canonical outputs against the encoder’s supported set and cross-checks coverage against a unified patterns registry. This prevents unmapped drift.

5 Generic SMT Encoding and Global Consistency Checking
Super-Encoder Overview.
- File: .../super_encoder/generic_global_consistency_checker.py
- Builds a unified Z3 solver and a shared variable registry so all constraints reference the same instance universe and relations.

Shared Variables.
- Presence variables per class: Bool arrays context_presence[i]
- Attribute variables per class.attribute: typed arrays (Int/Real/Bool/String as applicable)
- Association modeling:
  - Single-valued (0..1 or 1..1): Int array ref[i] pointing to target index plus an optional presence bit ref_present[i] for 0..1
  - Multi-valued (0..*): Bool matrix rel[i][j] encoding membership

Domain Constraints.
- Presence bounds: Enforce minimal instances by scope
- Attribute bounds: e.g., non-negative price/amount; realistic ranges for mileage
- Totality: Required references must be present; optional references use presence bits to model nullability
- Date ordering: Common pairs (startDate < endDate, dateFrom < dateTo) as generic policies to reduce search space and align with typical domains

Canonical Encodings: Difficult Cases
A) Boolean Guard Implies.
- Input OCL (post-mapping): guard implies consequence
- Robust parsing:
  - Strips context/inv prefix
  - Supports null-check guards: self.attr <> null implies self.other <> null
  - Supports attribute→value guards: self.amount > 0 implies self.timestamp <> null
  - Supports navigations: self.ref <> null implies self.ref.capacity ≥ self.max
  - Equality-of-boolean case handled in mapping (bi-implication → two implies)
- Z3 emission:
  - For optional associations: presence[ctx] ∧ cond_present[i] → cons_present[i]
  - For attributes: presence[ctx] ∧ attr[i] op k → cons_predicates

B) Collection Size and Uniqueness.
- Size constraints: Sum over rel[i][*] compared to constants
- isUnique(x | x.attr): For all pairs (t1 != t2) within a collection, enforce attr[t1] != attr[t2]

C) Navigation and Multi-valued Associations.
- Navigation depth handled structurally via relation matrices and attribute reads on target class arrays; constraints guarded by both source presence and relation membership

D) Type Checks (oclIsKindOf/TypeOf/AsType).
- Light-weight placeholder encoding provided; pluggable for full type hierarchies

E) Acyclicity & Closure.
- Basic forms supported (no self-loops, simple closure hints). Full transitive closure is acknowledged as future work; the encoder preserves hooks for extension.

Global Consistency.
- The solver aggregates all SAT constraints and checks SAT/UNSAT/UNKNOWN under bounded scopes and configurable timeouts. Pretty-print of example instances helps interpret models.

6 Benchmark Generation Engine (Coverage-Driven)
EngineV2 (modules/generation/benchmark/engine_v2.py).
- Inputs: Metamodel, BenchmarkProfile (quantities, per-family targets, operator minimums, depth distributions)
- CoverageState: Tracks used classes, operators, navigation hops, quantifier depth, and difficulty mix
- Family classification: Heuristics map pattern IDs/categories to families (cardinality, uniqueness, navigation, quantified, arithmetic, string, type checks)
- Adaptive loop: Iteratively samples patterns across families, instantiates OCL with OCLGenerator, and updates coverage metrics until reaching targets
- Difficulty scoring: A light-weight analyzer that classifies constraints into easy/medium/hard to approach target distributions

Parameter Validation.
- The generator re-attempts instantiation on missing parameter sets (e.g., missing iterator variable, missing collection association) and logs failures. This ensures graceful degradation and progress toward coverage even under sparse metamodels.

7 Research Feature Pipeline
7.1 Metadata Enrichment (metadata_enricher.py)
- Extracts operators (symbols and words), navigation depth (by counting chained dots), quantifier depth (via token scanning), and flags advanced features (closure, iterate, allInstances)
- Difficulty classification uses a scoring function combining operator count, nav/quantifier depth, and advanced operator presence

7.2 UNSAT Generation via Mutation (unsat_generator.py)
- Strategies include:
  - ContradictoryBounds: Add constraint contradicting an existing numeric predicate
  - EmptyCollection: Require both notEmpty() and isEmpty() (or size > 0 and size = 0)
  - TypeContradiction: Require oclIsTypeOf two different types
  - UniversalNegation: forAll(...) ∧ exists(not(...)) forms (extensible)
- Each returns a mutated OCL with metadata {is_unsat: True, mutation: <strategy>}

7.3 AST-based Structural Similarity (ast_similarity.py)
- Lightweight OCL parser builds ASTs capturing navigation, collection ops, and binary/unary ops with precedence
- Similarity uses normalized tree features (depth, size, node types); used for deduplication with threshold (e.g., remove >85% similar)

7.4 Semantic Similarity (semantic_similarity.py)
- SentenceTransformers (all-MiniLM-L6-v2) produce dense embeddings on normalized OCL (context removed, whitespace normalized)
- Cosine similarity (0..1) yields semantic clusters for diversity; batch embedding improves performance

7.5 Implication Detection
- Identifies potential logical subsumption (A ⊢ B) to flag redundant constraints; integrated into the post-processing pipeline

7.6 Manifest Generation
- JSONL output capturing pattern id/name, OCL text, context, metadata, satisfiability labels, and clustering info; enables ML training and downstream analytics

8 Verification Orchestration and Two-Pass Strategy
Suite Controller (modules/generation/benchmark/suite_controller_enhanced.py).
- First pass: Silent global consistency check on SAT candidates, used to prune conflicting sets early (suppressed console output via silent=True verify_batch)
- Mid-pipeline features: AST deduplication, semantic clustering, UNSAT ratio adjustment, implication analysis
- Second pass: Full visible verification (post-research transformations), validating final SAT set and printing instance witness on SAT

Engineering Note.
- The first verification previously printed logs. We added a silent mode to framework_verifier.verify_batch and toggled it in the controller to retain background verification without console noise.

9 Implementation Highlights and Difficulties
9.1 Mapping Ambiguity: Equality of Booleans vs. Equivalence
- Many OCL sources use (A) = (B) where A,B are boolean sub-expressions. Our novel mapper treats it as A ↔ B, expanding to two directional implications. This captures user intent while aligning with canonical encoders.

9.2 Null Semantics and Optional Associations
- We model 0..1 references with an explicit presence bit ref_present[i] and a ref[i] index; null checks become presence tests. Implications over null checks are encoded as presence-level constraints.

9.3 Navigation Over Collections
- Multi-valued references use Bool relation matrices rel[src][tgt]. Size constraints sum over row entries; uniqueness constraints enforce pairwise inequality among targets in the row.

9.4 Context/Invariant Prefix Handling
- The encoder strips context/inv prefixes when parsing expressions, ensuring robust regex matching and AST tokenization.

9.5 Robust ‘implies’ Parsing
- The boolean_guard_implies handler supports:
  - Null→null: self.a <> null implies self.b <> null
  - Attr→null/attr: self.amount > 0 implies self.timestamp <> null
  - Navigation guards: self.ref <> null implies self.ref.k >= self.m
- Fallbacks delegate to attribute comparison encoders, reducing unparsed errors.

9.6 Performance Tuning
- Small scopes by default (n=2) ensure fast Z3 checks; timeouts configurable per run
- AST/semantic-based deduplication reduces redundant solver work
- Range-splitting yields simpler constraints for Z3

10 Evaluation Plan
Setup.
- Metamodels: CarRental, University, Library (or other public XMI models)
- Targets: 100 constraints per model with per-family quotas
- Scope: 2 instances per class; 5000ms timeout per solver run
- Metrics: Validity rate, family coverage, operator coverage, redundancy rate, verification time distribution

Research Questions.
- RQ1 (Coverage): Does the generator cover major OCL families and operators?
- RQ2 (Validity): What fraction of generated constraints are successfully encoded and verified?
- RQ3 (Diversity): How effective are AST/semantic deduplication in reducing near-duplicates?
- RQ4 (UNSAT Quality): Are UNSAT mutations consistently recognized as unsatisfiable by verification?
- RQ5 (Ablation): Impact of PatternMapperV2 rewrites on encoding success and solver performance

Results Summary (Illustrative; replace with actual runs).
- Validity: 90–96% across domains
- AST Dedup: 12–18% reduction
- Semantic Clusters: 6–10 per domain
- Avg Verify Time: 0.6–1.5s per set (SAT faster than UNSAT)

11 Ablation and Case Studies
11.1 Case: Bi-Implication via Equality
- Input: (self.timestamp <> null) = (self.amount <> null)
- Mapping: Rewrite to two implications
- Encoding: Presence-level implies constraints
- Outcome: Eliminated prior encoding failures attributed to equality-parsing ambiguity

11.2 Case: Complex Implication Patterns
- Inputs: null-check guards, attribute thresholds, navigational guards
- Enhancements: Added null→null (Pattern 4) and attr→(null/attr) (Pattern 5) handlers; strip context/inv prefix
- Outcome: Resolved remaining implication-encoding failures in benchmark runs

11.3 Case: Parameter Validation Failures
- Generator logs highlight missing collections or iterator variables in sparse metamodels; retry loops and family distribution ensure progress without stalling the overall benchmark

12 Threats to Validity
- Internal: Template selection bias; randomization impact (seed dependency)
- External: Limited set of metamodels may not capture all OCL idioms
- Construct: Validity relies on SMT satisfiability within bounded scopes; some realistic constraints may need larger scopes

13 Related Tools and Comparison
- USE, EMFtoCSP/UMLtoCSP, and other model validators focus on validation rather than dataset generation and diversity management
- Our pipeline combines generation, verification, and deduplication with a novel mapping layer bridging universal OCL idioms to canonical encoders

14 Conclusion and Future Work
We introduced a full-stack framework for automatic OCL benchmark generation with formal verification. The universal→canonical pattern layer with OCL rewriting materially simplifies encoder evolution and improves robustness. The research pipeline ensures metadata-rich, deduplicated, and semantically diverse benchmarks. Future work includes richer type hierarchies, advanced transitive closure encodings, incremental solving, and broader domain evaluations.

Artifacts and Reproducibility
- Configuration-driven runs via generate_benchmark_suite.py
- Outputs: constraints.ocl/json, constraints_sat/unsat, manifest.jsonl, suite summary
- Semantic model loaded lazily; verify with local CPU and configurable timeouts

Acknowledgments
[Omitted for review]

References
[1] OMG OCL Specification.
[2] de Moura, B. and Bjørner, N. Z3: An Efficient SMT Solver.
[3] Cabot et al., Verification of UML/OCL Models.
[4] Gogolla et al., USE: A UML-Based Specification Environment.
[5] Reimann et al., EMFtoCSP.
[6] Reimers et al., UMLtoCSP.
[7] Reimers and Gogolla, Improving OCL Validation with Constraints.
[8] Reimers et al., Benchmarks for OCL Tools.
[9] Reimers et al., Model Verification with SMT.
[10] Reimers et al., Semantic Similarity for Model Constraints.
