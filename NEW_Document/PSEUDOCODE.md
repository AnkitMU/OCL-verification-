# OCL Generation Framework - Pseudocode Documentation

## Table of Contents

1. [Core Module](#1-core-module)
2. [Metamodel Extraction Module](#2-metamodel-extraction-module)
3. [Pattern Registry Module](#3-pattern-registry-module)
4. [Generation Engine Module](#4-generation-engine-module)
5. [OCL Generator Module](#5-ocl-generator-module)
6. [Metadata Enrichment Module](#6-metadata-enrichment-module)
7. [UNSAT Generation Module](#7-unsat-generation-module)
8. [AST Similarity Module](#8-ast-similarity-module)
9. [Semantic Similarity Module](#9-semantic-similarity-module)
10. [Implication Checker Module](#10-implication-checker-module)
11. [Compatibility Resolution Module](#11-compatibility-resolution-module)
12. [Verification Module](#12-verification-module)
13. [SMT Encoding Module](#13-smt-encoding-module)
14. [Pattern Detection Module](#14-pattern-detection-module)
15. [Manifest Generation Module](#15-manifest-generation-module)
16. [Suite Controller Module](#16-suite-controller-module)

---

## 1. Core Module

### 1.1 Data Models

```
CLASS Pattern:
    ATTRIBUTES:
        id: String
        name: String
        category: String
        description: String
        template: String
        parameters: List<Parameter>
        examples: List<String>
        complexity: Integer (1-3)
        tags: List<String>
    
    METHODS:
        get_parameter(name: String) -> Parameter:
            FOR each param IN parameters:
                IF param.name == name:
                    RETURN param
            RETURN None
        
        is_applicable(context_class: Class) -> Boolean:
            // Check if pattern can be applied to this class
            FOR each param IN parameters:
                IF param.required:
                    options = param.get_options_for_context(context_class)
                    IF options.is_empty():
                        RETURN False
            RETURN True
```

```
CLASS Parameter:
    ATTRIBUTES:
        name: String
        label: String
        type: String (select, number, text)
        options: String OR List<String>
        required: Boolean
        default: Any
    
    METHODS:
        get_options_for_context(context: Class, existing_params: Dict) -> List:
            IF options IS List:
                RETURN options  // Static options
            ELSE IF options == "attributes":
                RETURN context.get_all_attributes()
            ELSE IF options == "numeric_attributes":
                RETURN context.get_attributes_of_type([Integer, Float])
            ELSE IF options == "string_attributes":
                RETURN context.get_attributes_of_type([String])
            ELSE IF options == "boolean_attributes":
                RETURN context.get_attributes_of_type([Boolean])
            ELSE IF options == "associations":
                RETURN context.get_all_associations()
            ELSE IF options == "collection_associations":
                RETURN context.get_associations_with_multiplicity("*")
            ELSE IF options == "classes":
                RETURN metamodel.get_all_classes()
            ELSE IF options == "target_attributes":
                // Attributes from associated class
                assoc = existing_params.get("association")
                IF assoc IS NOT None:
                    target_class = assoc.target_type
                    RETURN target_class.get_all_attributes()
            RETURN []
```

```
CLASS OCLConstraint:
    ATTRIBUTES:
        id: String (unique identifier)
        pattern_id: String
        pattern_name: String
        category: String
        context: String (class name)
        ocl: String (full OCL text)
        parameters: Dict<String, Any>
        metadata: Dict<String, Any>
            - difficulty: String (easy/medium/hard)
            - operators_used: List<String>
            - navigation_depth: Integer
            - quantifier_depth: Integer
            - is_unsat: Boolean
            - mutation_strategy: String (if UNSAT)
            - verification_result: String (sat/unsat/unknown)
            - execution_time: Float
            - semantic_cluster: Integer
            - implies: List<String> (IDs of implied constraints)
    
    METHODS:
        to_dict() -> Dict:
            RETURN {
                "id": id,
                "pattern": pattern_id,
                "context": context,
                "ocl": ocl,
                "metadata": metadata
            }
        
        from_dict(data: Dict) -> OCLConstraint:
            constraint = NEW OCLConstraint()
            constraint.id = data["id"]
            constraint.pattern_id = data["pattern"]
            constraint.context = data["context"]
            constraint.ocl = data["ocl"]
            constraint.metadata = data["metadata"]
            RETURN constraint
```

---

## 2. Metamodel Extraction Module

```
CLASS MetamodelExtractor:
    ATTRIBUTES:
        xmi_path: String
        metamodel: Metamodel
        classes: Dict<String, Class>
        datatypes: Dict<String, DataType>
    
    METHOD __init__(xmi_path: String):
        self.xmi_path = xmi_path
        self.classes = {}
        self.datatypes = {}
        self._parse_xmi()
    
    METHOD _parse_xmi() -> Void:
        // Load XMI file
        tree = XML.parse(xmi_path)
        root = tree.get_root()
        
        // Extract all datatypes
        FOR each node IN root.find_all("eClassifiers"):
            IF node.get_attribute("xsi:type") == "ecore:EDataType":
                datatype = NEW DataType()
                datatype.name = node.get_attribute("name")
                datatype.instance_class = node.get_attribute("instanceClassName")
                self.datatypes[datatype.name] = datatype
        
        // Extract all classes
        FOR each node IN root.find_all("eClassifiers"):
            IF node.get_attribute("xsi:type") == "ecore:EClass":
                class_obj = self._parse_class(node)
                self.classes[class_obj.name] = class_obj
        
        // Resolve references (inheritance, associations)
        self._resolve_references(root)
        
        // Create metamodel
        self.metamodel = NEW Metamodel(self.classes, self.datatypes)
    
    METHOD _parse_class(node: XMLNode) -> Class:
        class_obj = NEW Class()
        class_obj.name = node.get_attribute("name")
        class_obj.is_abstract = node.get_attribute("abstract", default=False)
        
        // Parse attributes
        FOR each attr_node IN node.find_all("eStructuralFeatures"):
            IF attr_node.get_attribute("xsi:type") == "ecore:EAttribute":
                attribute = self._parse_attribute(attr_node)
                class_obj.add_attribute(attribute)
        
        // Parse associations
        FOR each ref_node IN node.find_all("eStructuralFeatures"):
            IF ref_node.get_attribute("xsi:type") == "ecore:EReference":
                association = self._parse_association(ref_node)
                class_obj.add_association(association)
        
        RETURN class_obj
    
    METHOD _parse_attribute(node: XMLNode) -> Attribute:
        attribute = NEW Attribute()
        attribute.name = node.get_attribute("name")
        attribute.type = node.get_attribute("eType")
        attribute.lower_bound = node.get_attribute("lowerBound", default=0)
        attribute.upper_bound = node.get_attribute("upperBound", default=1)
        attribute.is_required = (attribute.lower_bound > 0)
        RETURN attribute
    
    METHOD _parse_association(node: XMLNode) -> Association:
        association = NEW Association()
        association.name = node.get_attribute("name")
        association.target_type = node.get_attribute("eType")
        association.lower_bound = node.get_attribute("lowerBound", default=0)
        association.upper_bound = node.get_attribute("upperBound", default=1)
        association.is_containment = node.get_attribute("containment", default=False)
        association.is_collection = (association.upper_bound == -1)
        RETURN association
    
    METHOD get_metamodel() -> Metamodel:
        RETURN self.metamodel
```

```
CLASS Metamodel:
    ATTRIBUTES:
        classes: Dict<String, Class>
        datatypes: Dict<String, DataType>
    
    METHODS:
        get_class(name: String) -> Class:
            RETURN classes.get(name)
        
        get_all_classes() -> List<Class>:
            RETURN classes.values()
        
        get_concrete_classes() -> List<Class>:
            result = []
            FOR each class IN classes.values():
                IF NOT class.is_abstract:
                    result.append(class)
            RETURN result
```

---

## 3. Pattern Registry Module

```
CLASS PatternRegistry:
    ATTRIBUTES:
        patterns: Dict<String, Pattern>
        patterns_by_category: Dict<String, List<Pattern>>
        pattern_file: String
    
    METHOD __init__(pattern_file: String = "templates/patterns_unified.json"):
        self.pattern_file = pattern_file
        self.patterns = {}
        self.patterns_by_category = {}
        self._load_patterns()
    
    METHOD _load_patterns() -> Void:
        // Load JSON file
        data = JSON.load(pattern_file)
        
        // Parse patterns
        FOR each pattern_data IN data["patterns"]:
            pattern = self._parse_pattern(pattern_data)
            self.patterns[pattern.id] = pattern
            
            // Group by category
            IF pattern.category NOT IN patterns_by_category:
                patterns_by_category[pattern.category] = []
            patterns_by_category[pattern.category].append(pattern)
    
    METHOD _parse_pattern(data: Dict) -> Pattern:
        pattern = NEW Pattern()
        pattern.id = data["id"]
        pattern.name = data["name"]
        pattern.category = data["category"]
        pattern.description = data["description"]
        pattern.template = data["template"]
        pattern.complexity = data.get("complexity", 1)
        pattern.tags = data.get("tags", [])
        pattern.examples = data.get("examples", [])
        
        // Parse parameters
        pattern.parameters = []
        FOR each param_data IN data.get("parameters", []):
            param = NEW Parameter()
            param.name = param_data["name"]
            param.label = param_data["label"]
            param.type = param_data["type"]
            param.options = param_data["options"]
            param.required = param_data.get("required", True)
            param.default = param_data.get("default", None)
            pattern.parameters.append(param)
        
        RETURN pattern
    
    METHOD get_pattern(pattern_id: String) -> Pattern:
        RETURN patterns.get(pattern_id)
    
    METHOD get_patterns_by_category(category: String) -> List<Pattern>:
        RETURN patterns_by_category.get(category, [])
    
    METHOD get_all_patterns() -> List<Pattern>:
        RETURN patterns.values()
    
    METHOD get_pattern_ids() -> List<String>:
        RETURN patterns.keys()
```

---

## 4. Generation Engine Module

```
CLASS BenchmarkEngineV2:
    ATTRIBUTES:
        metamodel: Metamodel
        pattern_registry: PatternRegistry
        ocl_generator: OCLGenerator
        random: Random
    
    METHOD __init__(metamodel: Metamodel, seed: Integer = None):
        self.metamodel = metamodel
        self.pattern_registry = NEW PatternRegistry()
        self.ocl_generator = NEW OCLGenerator(pattern_registry)
        self.random = NEW Random(seed)
    
    METHOD generate(profile: BenchmarkProfile) -> List<OCLConstraint>:
        """
        Main generation algorithm
        
        INPUT: BenchmarkProfile with:
            - num_constraints: Target number of constraints
            - families_pct: Dict mapping pattern_id -> probability
            - complexity_profile: String (easy/medium/hard)
            - seed: Random seed
        
        OUTPUT: List of OCLConstraint objects
        """
        
        constraints = []
        attempts = 0
        max_attempts = profile.num_constraints * 10
        
        // Get concrete classes (non-abstract)
        concrete_classes = metamodel.get_concrete_classes()
        
        // Create weighted pattern list
        pattern_weights = self._prepare_pattern_weights(profile.families_pct)
        
        WHILE constraints.length < profile.num_constraints AND attempts < max_attempts:
            attempts += 1
            
            // 1. Select pattern (weighted random)
            pattern = self._select_pattern(pattern_weights)
            
            // 2. Select context class (random)
            context_class = random.choice(concrete_classes)
            
            // 3. Check pattern applicability
            IF NOT self._is_pattern_applicable(pattern, context_class):
                CONTINUE
            
            // 4. Generate constraint
            constraint = self._generate_constraint(pattern, context_class, profile)
            
            IF constraint IS None:
                CONTINUE
            
            // 5. Check for duplicates (AST similarity)
            IF self._is_duplicate(constraint, constraints, threshold=0.85):
                CONTINUE
            
            // 6. Add to result
            constraints.append(constraint)
            
            // 7. Log progress
            IF constraints.length % 10 == 0:
                PRINT(f"Generated {constraints.length}/{profile.num_constraints} constraints")
        
        RETURN constraints
    
    METHOD _prepare_pattern_weights(families_pct: Dict) -> List<(Pattern, Float)>:
        """
        Convert families_pct to list of (pattern, weight) tuples
        """
        pattern_weights = []
        
        FOR pattern_id, weight IN families_pct.items():
            pattern = pattern_registry.get_pattern(pattern_id)
            IF pattern IS NOT None:
                pattern_weights.append((pattern, weight))
        
        // Normalize weights
        total_weight = SUM(weight for (_, weight) in pattern_weights)
        normalized = []
        FOR (pattern, weight) IN pattern_weights:
            normalized.append((pattern, weight / total_weight))
        
        RETURN normalized
    
    METHOD _select_pattern(pattern_weights: List) -> Pattern:
        """
        Weighted random selection
        """
        rand_val = random.uniform(0, 1)
        cumulative = 0.0
        
        FOR (pattern, weight) IN pattern_weights:
            cumulative += weight
            IF rand_val <= cumulative:
                RETURN pattern
        
        // Fallback: return last pattern
        RETURN pattern_weights[-1][0]
    
    METHOD _is_pattern_applicable(pattern: Pattern, context_class: Class) -> Boolean:
        """
        Check if pattern can be applied to this context class
        """
        FOR each param IN pattern.parameters:
            IF param.required:
                options = param.get_options_for_context(context_class, {})
                IF options.is_empty():
                    RETURN False
        RETURN True
    
    METHOD _generate_constraint(pattern: Pattern, context_class: Class, 
                                 profile: BenchmarkProfile) -> OCLConstraint:
        """
        Generate single constraint from pattern + context
        """
        // 1. Resolve all parameters
        params = {}
        FOR each param IN pattern.parameters:
            // Get available options
            options = param.get_options_for_context(context_class, params)
            
            IF options.is_empty():
                IF param.required:
                    RETURN None  // Cannot generate
                ELSE:
                    params[param.name] = param.default
            ELSE:
                // Select random option
                params[param.name] = random.choice(options)
        
        // 2. Fill template
        ocl_text = pattern.template
        FOR param_name, param_value IN params.items():
            placeholder = "{" + param_name + "}"
            ocl_text = ocl_text.replace(placeholder, str(param_value))
        
        // 3. Create constraint object
        constraint = NEW OCLConstraint()
        constraint.id = self._generate_constraint_id(pattern, context_class)
        constraint.pattern_id = pattern.id
        constraint.pattern_name = pattern.name
        constraint.category = pattern.category
        constraint.context = context_class.name
        constraint.ocl = f"context {context_class.name} inv: {ocl_text}"
        constraint.parameters = params
        constraint.metadata = {
            "difficulty": self._calculate_difficulty(ocl_text),
            "complexity": pattern.complexity
        }
        
        RETURN constraint
    
    METHOD _is_duplicate(new_constraint: OCLConstraint, 
                         existing: List<OCLConstraint>, 
                         threshold: Float) -> Boolean:
        """
        Check if constraint is too similar to existing ones
        """
        FOR each existing_constraint IN existing:
            similarity = self._compute_ast_similarity(new_constraint, existing_constraint)
            IF similarity >= threshold:
                RETURN True
        RETURN False
    
    METHOD _compute_ast_similarity(c1: OCLConstraint, c2: OCLConstraint) -> Float:
        """
        Quick AST similarity check (simplified version)
        """
        // Exact match
        IF c1.ocl == c2.ocl:
            RETURN 1.0
        
        // Same pattern + same context -> high similarity
        IF c1.pattern_id == c2.pattern_id AND c1.context == c2.context:
            RETURN 0.9
        
        // Different patterns -> low similarity
        RETURN 0.0
    
    METHOD _calculate_difficulty(ocl: String) -> String:
        """
        Estimate constraint difficulty
        """
        score = 0
        
        // Count complexity indicators
        IF "forAll" IN ocl OR "exists" IN ocl:
            score += 2
        IF "implies" IN ocl:
            score += 1
        IF ocl.count("->") >= 2:  // Multiple navigations
            score += 1
        IF "collect" IN ocl OR "select" IN ocl OR "reject" IN ocl:
            score += 2
        IF "closure" IN ocl:
            score += 3
        
        IF score <= 2:
            RETURN "easy"
        ELSE IF score <= 4:
            RETURN "medium"
        ELSE:
            RETURN "hard"
    
    METHOD _generate_constraint_id(pattern: Pattern, context: Class) -> String:
        """
        Generate unique constraint ID
        """
        timestamp = CURRENT_TIMESTAMP()
        random_suffix = random.randint(1000, 9999)
        RETURN f"{pattern.id}_{context.name}_{random_suffix}"
```

---

## 5. OCL Generator Module

```
CLASS OCLGenerator:
    ATTRIBUTES:
        pattern_registry: PatternRegistry
    
    METHOD __init__(pattern_registry: PatternRegistry):
        self.pattern_registry = pattern_registry
    
    METHOD generate_ocl(pattern_id: String, context: String, params: Dict) -> String:
        """
        Generate OCL constraint from pattern template
        
        INPUT:
            - pattern_id: ID of pattern to use
            - context: Class name for context
            - params: Dict of parameter values
        
        OUTPUT: OCL constraint text
        """
        
        // Get pattern
        pattern = pattern_registry.get_pattern(pattern_id)
        IF pattern IS None:
            RAISE Error(f"Pattern not found: {pattern_id}")
        
        // Fill template
        ocl_body = pattern.template
        FOR param_name, param_value IN params.items():
            placeholder = "{" + param_name + "}"
            ocl_body = ocl_body.replace(placeholder, str(param_value))
        
        // Check for unfilled placeholders
        IF "{" IN ocl_body:
            RAISE Error(f"Unfilled parameters in template: {ocl_body}")
        
        // Add context
        full_ocl = f"context {context} inv: {ocl_body}"
        
        RETURN full_ocl
    
    METHOD validate_parameters(pattern_id: String, params: Dict) -> Boolean:
        """
        Check if all required parameters are provided
        """
        pattern = pattern_registry.get_pattern(pattern_id)
        
        FOR each param IN pattern.parameters:
            IF param.required AND param.name NOT IN params:
                RETURN False
        
        RETURN True
```

---

## 6. Metadata Enrichment Module

```
CLASS MetadataEnricher:
    
    METHOD enrich_constraint_metadata(constraint: OCLConstraint) -> Void:
        """
        Add metadata to constraint
        
        Enriches constraint.metadata with:
            - operators_used: List of OCL operators
            - navigation_depth: Number of navigation hops
            - quantifier_depth: Nesting level of quantifiers
            - difficulty: easy/medium/hard
        """
        
        ocl = constraint.ocl
        
        // 1. Extract operators
        operators = self._extract_operators(ocl)
        constraint.metadata["operators_used"] = operators
        
        // 2. Calculate navigation depth
        nav_depth = self._calculate_navigation_depth(ocl)
        constraint.metadata["navigation_depth"] = nav_depth
        
        // 3. Calculate quantifier depth
        quant_depth = self._calculate_quantifier_depth(ocl)
        constraint.metadata["quantifier_depth"] = quant_depth
        
        // 4. Update difficulty if not set
        IF "difficulty" NOT IN constraint.metadata:
            difficulty = self._calculate_difficulty(ocl)
            constraint.metadata["difficulty"] = difficulty
    
    METHOD _extract_operators(ocl: String) -> List<String>:
        """
        Extract all OCL operators used
        """
        operators = []
        
        // Collection operations
        collection_ops = ["size", "isEmpty", "notEmpty", "includes", "excludes",
                         "includesAll", "excludesAll", "count", "sum", "product",
                         "select", "reject", "collect", "forAll", "exists", 
                         "one", "any", "isUnique", "sortedBy", "union", 
                         "intersection", "closure"]
        
        FOR each op IN collection_ops:
            IF op IN ocl:
                operators.append(op)
        
        // String operations
        string_ops = ["concat", "substring", "toUpper", "toLower", "size"]
        FOR each op IN string_ops:
            IF op + "(" IN ocl:
                operators.append(op)
        
        // Logical operators
        IF " implies " IN ocl:
            operators.append("implies")
        IF " and " IN ocl:
            operators.append("and")
        IF " or " IN ocl:
            operators.append("or")
        IF " not " IN ocl OR "not(" IN ocl:
            operators.append("not")
        IF " xor " IN ocl:
            operators.append("xor")
        
        // Comparison operators
        comparisons = [">", ">=", "<", "<=", "=", "<>"]
        FOR each comp IN comparisons:
            IF comp IN ocl:
                operators.append(comp)
        
        RETURN UNIQUE(operators)
    
    METHOD _calculate_navigation_depth(ocl: String) -> Integer:
        """
        Count maximum navigation depth (self.ref1.ref2.attr = depth 2)
        """
        max_depth = 0
        
        // Find all navigation chains
        navigation_patterns = REGEX_FIND_ALL(r'self(\.\w+)+', ocl)
        
        FOR each nav IN navigation_patterns:
            // Count dots (excluding "self.")
            depth = nav.count(".") - 1
            max_depth = MAX(max_depth, depth)
        
        RETURN max_depth
    
    METHOD _calculate_quantifier_depth(ocl: String) -> Integer:
        """
        Calculate nesting depth of quantifiers (forAll, exists)
        """
        depth = 0
        max_depth = 0
        
        FOR each char IN ocl:
            IF char == '(':
                depth += 1
                max_depth = MAX(max_depth, depth)
            ELSE IF char == ')':
                depth -= 1
        
        // Count quantifiers
        quantifiers = ["forAll", "exists", "one", "any"]
        quant_count = 0
        FOR each quant IN quantifiers:
            quant_count += ocl.count(quant)
        
        // Estimate nesting (simplified)
        IF quant_count == 0:
            RETURN 0
        ELSE IF quant_count == 1:
            RETURN 1
        ELSE:
            RETURN 2  // Assume nested if multiple quantifiers
    
    METHOD _calculate_difficulty(ocl: String) -> String:
        """
        Calculate overall difficulty score
        """
        score = 0
        
        // Quantifiers add complexity
        IF "forAll" IN ocl OR "exists" IN ocl:
            score += 2
        
        // Implications add complexity
        IF "implies" IN ocl:
            score += 1
        
        // Navigation adds complexity
        nav_depth = self._calculate_navigation_depth(ocl)
        score += nav_depth
        
        // Collection operations add complexity
        complex_ops = ["collect", "select", "reject", "closure", "sortedBy"]
        FOR each op IN complex_ops:
            IF op IN ocl:
                score += 2
        
        // Classify difficulty
        IF score <= 2:
            RETURN "easy"
        ELSE IF score <= 5:
            RETURN "medium"
        ELSE:
            RETURN "hard"
```

---

## 7. UNSAT Generation Module

```
CLASS UNSATGenerator:
    ATTRIBUTES:
        random: Random
        mutation_strategies: List<Function>
    
    METHOD __init__(seed: Integer = None):
        self.random = NEW Random(seed)
        self.mutation_strategies = [
            self.operator_flip,
            self.bound_tightening,
            self.negation,
            self.value_contradiction,
            self.quantifier_flip
        ]
    
    METHOD generate_mixed_sat_unsat_set(sat_constraints: List<OCLConstraint>,
                                         metamodel: Metamodel,
                                         unsat_ratio: Float = 0.4) -> Tuple:
        """
        Generate mixed SAT/UNSAT constraint set
        
        INPUT:
            - sat_constraints: List of SAT constraints
            - metamodel: Metamodel object
            - unsat_ratio: Target ratio of UNSAT (0.0-1.0)
        
        OUTPUT: (all_constraints, unsat_map)
            - all_constraints: Mixed list of SAT + UNSAT
            - unsat_map: Dict mapping UNSAT ID -> mutation strategy
        """
        
        // Calculate how many UNSAT to generate
        num_sat = sat_constraints.length
        num_unsat = ROUND(num_sat * unsat_ratio / (1 - unsat_ratio))
        
        // Select constraints for mutation
        to_mutate = random.sample(sat_constraints, MIN(num_unsat, num_sat))
        
        unsat_constraints = []
        unsat_map = {}
        
        FOR each sat_constraint IN to_mutate:
            // Choose random mutation strategy
            strategy = random.choice(mutation_strategies)
            
            // Apply mutation
            unsat_constraint = strategy(sat_constraint, metamodel)
            
            IF unsat_constraint IS NOT None:
                unsat_constraints.append(unsat_constraint)
                unsat_map[unsat_constraint.id] = strategy.name
        
        // Combine SAT + UNSAT
        all_constraints = sat_constraints + unsat_constraints
        random.shuffle(all_constraints)
        
        RETURN (all_constraints, unsat_map)
    
    METHOD operator_flip(constraint: OCLConstraint, metamodel: Metamodel) -> OCLConstraint:
        """
        Strategy 1: Flip comparison/logical operators
        
        Examples:
            - > becomes <=
            - = becomes <>
            - and becomes or
            - forAll becomes exists
        """
        
        ocl = constraint.ocl
        
        // Define operator flips
        OPERATOR_FLIPS = {
            ">": "<=",
            ">=": "<",
            "<": ">=",
            "<=": ">",
            "=": "<>",
            "<>": "=",
            " and ": " or ",
            " or ": " and ",
            " implies ": " and not ",
            "forAll": "exists",
            "exists": "forAll"
        }
        
        // Find and flip first operator
        FOR original, flipped IN OPERATOR_FLIPS.items():
            IF original IN ocl:
                ocl = ocl.replace(original, flipped, count=1)
                BREAK
        
        // Create UNSAT constraint
        unsat = constraint.copy()
        unsat.id = constraint.id + "_unsat_opflip"
        unsat.ocl = ocl
        unsat.metadata["is_unsat"] = True
        unsat.metadata["mutation_strategy"] = "operator_flip"
        unsat.metadata["original_id"] = constraint.id
        
        RETURN unsat
    
    METHOD bound_tightening(constraint: OCLConstraint, metamodel: Metamodel) -> OCLConstraint:
        """
        Strategy 2: Make numeric bounds impossible to satisfy
        
        Examples:
            - >= 5 becomes >= 5000
            - <= 100 becomes <= -1000
        """
        
        ocl = constraint.ocl
        
        // Find numeric comparisons
        match = REGEX_SEARCH(r'([><=]+)\s*(\d+)', ocl)
        IF match IS None:
            RETURN None  // No numeric comparison found
        
        operator = match.group(1)
        value = INTEGER(match.group(2))
        
        // Make bound extreme
        IF operator IN [">", ">="]:
            new_value = value * 1000  // Impossibly high
        ELSE IF operator IN ["<", "<="]:
            new_value = -1000  // Impossibly low
        ELSE:
            new_value = value + 9999  // For equality
        
        // Replace in OCL
        ocl = ocl.replace(str(value), str(new_value), count=1)
        
        // Create UNSAT constraint
        unsat = constraint.copy()
        unsat.id = constraint.id + "_unsat_bound"
        unsat.ocl = ocl
        unsat.metadata["is_unsat"] = True
        unsat.metadata["mutation_strategy"] = "bound_tightening"
        unsat.metadata["original_id"] = constraint.id
        
        RETURN unsat
    
    METHOD negation(constraint: OCLConstraint, metamodel: Metamodel) -> OCLConstraint:
        """
        Strategy 3: Wrap entire constraint in not(...)
        
        Examples:
            - self.age > 18 becomes not(self.age > 18)
            - self.rentals->notEmpty() becomes not(self.rentals->notEmpty())
        """
        
        ocl = constraint.ocl
        
        // Extract body (after "inv:")
        match = REGEX_SEARCH(r'inv:\s*(.+)', ocl)
        IF match IS None:
            RETURN None
        
        body = match.group(1).strip()
        negated_body = f"not({body})"
        
        // Replace in OCL
        ocl = ocl.replace(body, negated_body)
        
        // Create UNSAT constraint
        unsat = constraint.copy()
        unsat.id = constraint.id + "_unsat_neg"
        unsat.ocl = ocl
        unsat.metadata["is_unsat"] = True
        unsat.metadata["mutation_strategy"] = "negation"
        unsat.metadata["original_id"] = constraint.id
        
        RETURN unsat
    
    METHOD value_contradiction(constraint: OCLConstraint, metamodel: Metamodel) -> OCLConstraint:
        """
        Strategy 4: Add contradictory clause
        
        Examples:
            - self.age > 18 becomes self.age > 18 and self.age < 0
            - self.price > 0 becomes self.price > 0 and self.price < 0
        """
        
        ocl = constraint.ocl
        
        // Find attribute comparisons
        match = REGEX_SEARCH(r'self\.(\w+)\s*([><=]+)\s*(\d+)', ocl)
        IF match IS None:
            RETURN None
        
        attr = match.group(1)
        operator = match.group(2)
        value = INTEGER(match.group(3))
        
        // Add contradictory constraint
        IF operator IN [">", ">="]:
            contradiction = f" and self.{attr} < 0"
        ELSE IF operator IN ["<", "<="]:
            contradiction = f" and self.{attr} > 999999"
        ELSE:  // equality
            contradiction = f" and self.{attr} <> {value}"
        
        ocl = ocl + contradiction
        
        // Create UNSAT constraint
        unsat = constraint.copy()
        unsat.id = constraint.id + "_unsat_contra"
        unsat.ocl = ocl
        unsat.metadata["is_unsat"] = True
        unsat.metadata["mutation_strategy"] = "value_contradiction"
        unsat.metadata["original_id"] = constraint.id
        
        RETURN unsat
    
    METHOD quantifier_flip(constraint: OCLConstraint, metamodel: Metamodel) -> OCLConstraint:
        """
        Strategy 5: Flip quantifiers (forAll <-> exists)
        
        Examples:
            - self.items->forAll(i | i.price > 0) 
              becomes self.items->exists(i | i.price > 0)
              (UNSAT if items can be empty)
        """
        
        ocl = constraint.ocl
        
        // Check for quantifiers
        IF "forAll" IN ocl:
            ocl = ocl.replace("forAll", "exists", count=1)
        ELSE IF "exists" IN ocl:
            ocl = ocl.replace("exists", "forAll", count=1)
        ELSE:
            RETURN None  // No quantifiers
        
        // Create UNSAT constraint
        unsat = constraint.copy()
        unsat.id = constraint.id + "_unsat_quant"
        unsat.ocl = ocl
        unsat.metadata["is_unsat"] = True
        unsat.metadata["mutation_strategy"] = "quantifier_flip"
        unsat.metadata["original_id"] = constraint.id
        
        RETURN unsat
```

---

## 8. AST Similarity Module

```
CLASS ASTSimilarity:
    
    METHOD compute_similarity(c1: OCLConstraint, c2: OCLConstraint) -> Float:
        """
        Compute AST similarity between two constraints
        
        OUTPUT: Float between 0.0 (completely different) and 1.0 (identical)
        """
        
        // Quick checks
        IF c1.ocl == c2.ocl:
            RETURN 1.0
        
        IF c1.pattern_id != c2.pattern_id:
            RETURN 0.0  // Different patterns -> low similarity
        
        // Parse to AST
        ast1 = self._parse_to_ast(c1.ocl)
        ast2 = self._parse_to_ast(c2.ocl)
        
        // Compute tree edit distance
        distance = self._tree_edit_distance(ast1, ast2)
        max_size = MAX(ast1.size(), ast2.size())
        
        // Normalize to similarity score
        similarity = 1.0 - (distance / max_size)
        
        RETURN similarity
    
    METHOD _parse_to_ast(ocl: String) -> ASTNode:
        """
        Parse OCL to AST (simplified)
        """
        
        // Extract body
        match = REGEX_SEARCH(r'inv:\s*(.+)', ocl)
        body = match.group(1) IF match ELSE ocl
        
        // Tokenize
        tokens = self._tokenize(body)
        
        // Build AST
        ast = self._build_ast(tokens)
        
        RETURN ast
    
    METHOD _tokenize(text: String) -> List<String>:
        """
        Split OCL into tokens
        """
        // Define token patterns
        patterns = [
            r'\w+',           // Identifiers
            r'->',            // Arrow
            r'\d+',           // Numbers
            r'[><=]+',        // Comparisons
            r'[(){}]',        // Brackets
            r'\.',            // Dot
            r'\|',            // Pipe
        ]
        
        tokens = []
        pos = 0
        WHILE pos < text.length:
            matched = False
            FOR each pattern IN patterns:
                match = REGEX_MATCH(pattern, text[pos:])
                IF match:
                    tokens.append(match.group(0))
                    pos += match.end()
                    matched = True
                    BREAK
            IF NOT matched:
                pos += 1  // Skip unknown character
        
        RETURN tokens
    
    METHOD _build_ast(tokens: List<String>) -> ASTNode:
        """
        Build AST from tokens (simplified recursive descent)
        """
        root = NEW ASTNode("ROOT")
        stack = [root]
        
        FOR each token IN tokens:
            IF token == '(':
                // Start new subtree
                node = NEW ASTNode("EXPR")
                stack[-1].add_child(node)
                stack.append(node)
            ELSE IF token == ')':
                // End subtree
                IF stack.length > 1:
                    stack.pop()
            ELSE:
                // Add token as leaf
                node = NEW ASTNode(token)
                stack[-1].add_child(node)
        
        RETURN root
    
    METHOD _tree_edit_distance(tree1: ASTNode, tree2: ASTNode) -> Integer:
        """
        Compute tree edit distance using dynamic programming
        
        Operations: insert, delete, rename
        """
        
        // Base cases
        IF tree1 IS None AND tree2 IS None:
            RETURN 0
        IF tree1 IS None:
            RETURN tree2.size()
        IF tree2 IS None:
            RETURN tree1.size()
        
        // Node cost
        node_cost = 0 IF tree1.value == tree2.value ELSE 1
        
        // Recursive case: compute cost for children
        children1 = tree1.get_children()
        children2 = tree2.get_children()
        
        // Use dynamic programming to match children
        m = children1.length
        n = children2.length
        dp = MATRIX(m+1, n+1, infinity)
        dp[0][0] = 0
        
        FOR i FROM 0 TO m:
            FOR j FROM 0 TO n:
                IF i > 0:
                    // Delete from tree1
                    dp[i][j] = MIN(dp[i][j], 
                                   dp[i-1][j] + children1[i-1].size())
                IF j > 0:
                    // Insert to tree1
                    dp[i][j] = MIN(dp[i][j], 
                                   dp[i][j-1] + children2[j-1].size())
                IF i > 0 AND j > 0:
                    // Match or rename
                    child_cost = self._tree_edit_distance(children1[i-1], 
                                                           children2[j-1])
                    dp[i][j] = MIN(dp[i][j], dp[i-1][j-1] + child_cost)
        
        RETURN node_cost + dp[m][n]
    
    METHOD remove_duplicates(constraints: List<OCLConstraint>, 
                            threshold: Float = 0.85) -> List<OCLConstraint>:
        """
        Remove duplicate constraints based on AST similarity
        """
        
        unique = []
        
        FOR each constraint IN constraints:
            is_duplicate = False
            
            FOR each existing IN unique:
                similarity = self.compute_similarity(constraint, existing)
                IF similarity >= threshold:
                    is_duplicate = True
                    BREAK
            
            IF NOT is_duplicate:
                unique.append(constraint)
        
        RETURN unique
```

---

## 9. Semantic Similarity Module

```
CLASS SemanticSimilarity:
    ATTRIBUTES:
        model: SentenceTransformer
        embeddings_cache: Dict<String, Vector>
    
    METHOD __init__(model_name: String = "all-MiniLM-L6-v2"):
        self.model = SentenceTransformer(model_name)
        self.embeddings_cache = {}
    
    METHOD compute_embeddings_batch(constraints: List<OCLConstraint>) -> Dict:
        """
        Compute embeddings for all constraints
        
        OUTPUT: Dict mapping constraint_id -> embedding vector
        """
        
        embeddings = {}
        
        // Extract OCL texts
        ocl_texts = [c.ocl for c in constraints]
        constraint_ids = [c.id for c in constraints]
        
        // Compute embeddings in batch (efficient)
        vectors = self.model.encode(ocl_texts, 
                                     batch_size=32,
                                     show_progress_bar=True)
        
        // Store in dict
        FOR i FROM 0 TO constraints.length:
            embeddings[constraint_ids[i]] = vectors[i]
            self.embeddings_cache[constraint_ids[i]] = vectors[i]
        
        RETURN embeddings
    
    METHOD cosine_similarity(vec1: Vector, vec2: Vector) -> Float:
        """
        Compute cosine similarity between two vectors
        """
        dot_product = DOT(vec1, vec2)
        norm1 = NORM(vec1)
        norm2 = NORM(vec2)
        
        IF norm1 == 0 OR norm2 == 0:
            RETURN 0.0
        
        RETURN dot_product / (norm1 * norm2)
    
    METHOD cluster_constraints(constraints: List<OCLConstraint>, 
                              num_clusters: Integer = 5) -> Dict:
        """
        Cluster constraints by semantic similarity
        
        OUTPUT: Dict mapping cluster_id -> List<OCLConstraint>
        """
        
        // Compute embeddings
        embeddings = self.compute_embeddings_batch(constraints)
        
        // Convert to matrix
        embedding_matrix = []
        constraint_list = []
        FOR constraint IN constraints:
            embedding_matrix.append(embeddings[constraint.id])
            constraint_list.append(constraint)
        
        // Apply K-means clustering
        kmeans = KMeans(n_clusters=num_clusters, random_state=42)
        cluster_labels = kmeans.fit_predict(embedding_matrix)
        
        // Group by cluster
        clusters = {}
        FOR i FROM 0 TO constraint_list.length:
            cluster_id = cluster_labels[i]
            IF cluster_id NOT IN clusters:
                clusters[cluster_id] = []
            clusters[cluster_id].append(constraint_list[i])
        
        RETURN clusters
    
    METHOD find_similar_constraints(query: OCLConstraint,
                                    candidates: List<OCLConstraint>,
                                    top_k: Integer = 5) -> List<(OCLConstraint, Float)>:
        """
        Find most similar constraints to query
        
        OUTPUT: List of (constraint, similarity_score) tuples, sorted by score
        """
        
        // Get query embedding
        query_embedding = self.model.encode([query.ocl])[0]
        
        // Compute similarities
        similarities = []
        FOR each candidate IN candidates:
            IF candidate.id == query.id:
                CONTINUE  // Skip self
            
            // Get candidate embedding
            IF candidate.id IN embeddings_cache:
                candidate_embedding = embeddings_cache[candidate.id]
            ELSE:
                candidate_embedding = self.model.encode([candidate.ocl])[0]
            
            // Compute similarity
            similarity = self.cosine_similarity(query_embedding, candidate_embedding)
            similarities.append((candidate, similarity))
        
        // Sort by similarity (descending)
        similarities.sort(key=lambda x: x[1], reverse=True)
        
        // Return top k
        RETURN similarities[:top_k]
```

---

## 10. Implication Checker Module

```
CLASS ImplicationChecker:
    
    METHOD check_syntactic_implication(c1: OCLConstraint, 
                                       c2: OCLConstraint) -> Boolean:
        """
        Check if c1 syntactically implies c2
        
        Examples:
            - "self.age > 20" implies "self.age > 18"
            - "self.price >= 100" implies "self.price > 50"
        
        OUTPUT: True if c1 => c2, False otherwise
        """
        
        // Same constraint
        IF c1.ocl == c2.ocl:
            RETURN True
        
        // Check if same attribute comparison
        match1 = REGEX_SEARCH(r'self\.(\w+)\s*([><=]+)\s*(\d+)', c1.ocl)
        match2 = REGEX_SEARCH(r'self\.(\w+)\s*([><=]+)\s*(\d+)', c2.ocl)
        
        IF match1 AND match2:
            attr1 = match1.group(1)
            op1 = match1.group(2)
            val1 = INTEGER(match1.group(3))
            
            attr2 = match2.group(1)
            op2 = match2.group(2)
            val2 = INTEGER(match2.group(3))
            
            // Same attribute
            IF attr1 == attr2:
                RETURN self._check_numeric_implication(op1, val1, op2, val2)
        
        RETURN False
    
    METHOD _check_numeric_implication(op1: String, val1: Integer,
                                      op2: String, val2: Integer) -> Boolean:
        """
        Check if (x op1 val1) implies (x op2 val2)
        """
        
        // x > 20 implies x > 18
        IF op1 == ">" AND op2 == ">":
            RETURN val1 >= val2
        
        // x >= 20 implies x > 18
        IF op1 == ">=" AND op2 == ">":
            RETURN val1 > val2
        
        // x >= 20 implies x >= 18
        IF op1 == ">=" AND op2 == ">=":
            RETURN val1 >= val2
        
        // x < 10 implies x < 20
        IF op1 == "<" AND op2 == "<":
            RETURN val1 <= val2
        
        // x <= 10 implies x < 20
        IF op1 == "<=" AND op2 == "<":
            RETURN val1 < val2
        
        // x <= 10 implies x <= 20
        IF op1 == "<=" AND op2 == "<=":
            RETURN val1 <= val2
        
        // x = 10 implies x >= 10 and x <= 10
        IF op1 == "=":
            IF op2 == ">=":
                RETURN val1 >= val2
            IF op2 == "<=":
                RETURN val1 <= val2
        
        RETURN False
    
    METHOD find_implications(constraints: List<OCLConstraint>) -> Dict:
        """
        Find all implication relationships
        
        OUTPUT: Dict mapping constraint_id -> List<constraint_id>
                (constraints implied by this one)
        """
        
        implications = {}
        
        FOR each c1 IN constraints:
            implied_by_c1 = []
            
            FOR each c2 IN constraints:
                IF c1.id == c2.id:
                    CONTINUE
                
                IF self.check_syntactic_implication(c1, c2):
                    implied_by_c1.append(c2.id)
            
            IF implied_by_c1.length > 0:
                implications[c1.id] = implied_by_c1
        
        RETURN implications
    
    METHOD add_implication_metadata(constraints: List<OCLConstraint>) -> Void:
        """
        Add implication information to constraint metadata
        """
        
        implications = self.find_implications(constraints)
        
        FOR each constraint IN constraints:
            IF constraint.id IN implications:
                constraint.metadata["implies"] = implications[constraint.id]
            ELSE:
                constraint.metadata["implies"] = []
```

---

## 11. Compatibility Resolution Module

```
CLASS CompatibilityResolver:
    ATTRIBUTES:
        verifier: FrameworkConstraintVerifier
    
    METHOD __init__(verifier: FrameworkConstraintVerifier):
        self.verifier = verifier
    
    METHOD find_compatible_subset(constraints: List<OCLConstraint>, 
                                   metamodel: Metamodel) -> List<OCLConstraint>:
        """
        Greedy Maximal Compatible Subset (GMCS) Algorithm
        
        INPUT: List of constraints (may be mutually inconsistent)
        OUTPUT: Maximal subset that is globally consistent (SAT)
        
        COMPLEXITY: O(n²) - requires n verification calls in worst case
        """
        
        // STEP 1: Initial check - verify all constraints together
        result = verifier.verify_batch(constraints, silent=True)
        
        IF result.is_satisfiable:
            // All constraints are compatible!
            RETURN constraints
        
        // STEP 2: Greedy algorithm - build compatible subset
        compatible = []  // Start with empty set
        
        FOR each constraint IN constraints:
            // Test if adding this constraint keeps set SAT
            test_set = compatible + [constraint]
            
            // Verify with Z3 (silent mode)
            result = verifier.verify_batch(test_set, silent=True)
            
            IF result.is_satisfiable:
                // Adding constraint maintains consistency
                compatible.append(constraint)
                PRINT(f"✓ Added {constraint.id} (total: {compatible.length})")
            ELSE:
                // Adding constraint creates conflict - skip it
                PRINT(f"✗ Skipped {constraint.id} (conflicts)")
        
        RETURN compatible
    
    METHOD find_compatible_subset_with_prioritization(
            constraints: List<OCLConstraint>,
            metamodel: Metamodel,
            priority_fn: Function) -> List<OCLConstraint>:
        """
        Enhanced GMCS with priority-based ordering
        
        Process constraints in priority order to maximize retention
        of important constraints
        """
        
        // Sort by priority (descending)
        sorted_constraints = SORT(constraints, key=priority_fn, reverse=True)
        
        // Apply greedy algorithm on sorted list
        RETURN self.find_compatible_subset(sorted_constraints, metamodel)
    
    METHOD compute_priority_score(constraint: OCLConstraint) -> Float:
        """
        Compute priority score for constraint ordering
        
        Higher score = higher priority (process first)
        """
        score = 0.0
        
        // Prioritize rare patterns
        IF constraint.category IN ["advanced", "complex"]:
            score += 2.0
        
        // Prioritize harder constraints
        IF constraint.metadata.get("difficulty") == "hard":
            score += 1.5
        ELSE IF constraint.metadata.get("difficulty") == "medium":
            score += 1.0
        
        // Prioritize constraints with quantifiers
        IF "forAll" IN constraint.ocl OR "exists" IN constraint.ocl:
            score += 1.0
        
        // Prioritize constraints with implications
        IF "implies" IN constraint.ocl:
            score += 0.5
        
        RETURN score
    
    METHOD analyze_conflicts(constraints: List<OCLConstraint>,
                            metamodel: Metamodel) -> Dict:
        """
        Analyze which constraints conflict with each other
        
        OUTPUT: Dict mapping constraint_id -> List<conflicting_constraint_ids>
        
        WARNING: Expensive - requires O(n²) verification calls
        """
        
        conflicts = {}
        
        FOR i, c1 IN ENUMERATE(constraints):
            conflicts[c1.id] = []
            
            FOR j, c2 IN ENUMERATE(constraints):
                IF i >= j:
                    CONTINUE  // Skip self and duplicates
                
                // Test if c1 and c2 are compatible
                result = verifier.verify_batch([c1, c2], silent=True)
                
                IF NOT result.is_satisfiable:
                    // Conflict detected
                    conflicts[c1.id].append(c2.id)
                    IF c2.id NOT IN conflicts:
                        conflicts[c2.id] = []
                    conflicts[c2.id].append(c1.id)
        
        RETURN conflicts
```

---

## 12. Verification Module

```
CLASS FrameworkConstraintVerifier:
    ATTRIBUTES:
        metamodel: Metamodel
        checker: GenericGlobalConsistencyChecker  // Z3 encoder
        detector: ComprehensivePatternDetector
        scope: Dict<String, Integer>  // Instance limits
    
    METHOD __init__(metamodel: Metamodel, scope: Dict = None):
        self.metamodel = metamodel
        self.checker = NEW GenericGlobalConsistencyChecker(metamodel)
        self.detector = NEW ComprehensivePatternDetector()
        self.scope = scope OR self._default_scope()
    
    METHOD _default_scope() -> Dict:
        """
        Generate default scope (5 instances per class)
        """
        scope = {}
        FOR each class IN metamodel.get_all_classes():
            scope[f"n{class.name}"] = 5
        RETURN scope
    
    METHOD verify(constraint: OCLConstraint, silent: Boolean = False) -> VerificationResult:
        """
        Verify single constraint
        
        OUTPUT: VerificationResult with:
            - is_valid: True if encoding succeeded
            - is_satisfiable: True if Z3 found model (SAT)
            - solver_result: "sat", "unsat", or "unknown"
            - execution_time: Verification time in seconds
            - errors: List of errors (if any)
        """
        
        start_time = CURRENT_TIME()
        
        // Redirect stdout if silent
        IF silent:
            old_stdout = STDOUT
            STDOUT = NULL_STREAM
        
        TRY:
            // STEP 1: Pattern detection
            patterns = detector.detect_patterns(constraint.ocl)
            
            // STEP 2: OCL → Z3 encoding
            z3_formula = checker.encode_constraint(
                constraint.ocl,
                constraint.context,
                scope
            )
            
            // STEP 3: Z3 solving
            solver = NEW Z3Solver()
            solver.add(z3_formula)
            z3_result = solver.check()
            
            // STEP 4: Interpret result
            is_sat = (z3_result == Z3.SAT)
            solver_result = str(z3_result).lower()  // "sat", "unsat", "unknown"
            
            // Get model if SAT
            model = None
            IF is_sat:
                model = solver.model()
            
            execution_time = CURRENT_TIME() - start_time
            
            RETURN NEW VerificationResult(
                constraint_id=constraint.id,
                is_valid=True,
                is_satisfiable=is_sat,
                solver_result=solver_result,
                execution_time=execution_time,
                model=model,
                errors=[]
            )
        
        CATCH Exception AS e:
            // Encoding or solving failed
            execution_time = CURRENT_TIME() - start_time
            
            RETURN NEW VerificationResult(
                constraint_id=constraint.id,
                is_valid=False,
                is_satisfiable=False,
                solver_result="error",
                execution_time=execution_time,
                errors=[str(e)]
            )
        
        FINALLY:
            IF silent:
                STDOUT = old_stdout
    
    METHOD verify_batch(constraints: List<OCLConstraint>, 
                        silent: Boolean = False) -> BatchVerificationResult:
        """
        Verify multiple constraints together (global consistency)
        
        This encodes ALL constraints as a single Z3 formula and checks
        if there exists a model satisfying all of them simultaneously.
        """
        
        start_time = CURRENT_TIME()
        
        // Redirect stdout if silent
        IF silent:
            old_stdout = STDOUT
            STDOUT = NULL_STREAM
        
        TRY:
            // STEP 1: Create Z3 solver
            solver = NEW Z3Solver()
            
            // STEP 2: Encode all constraints
            FOR each constraint IN constraints:
                // Detect patterns
                patterns = detector.detect_patterns(constraint.ocl)
                
                // Encode to Z3
                z3_formula = checker.encode_constraint(
                    constraint.ocl,
                    constraint.context,
                    scope
                )
                
                // Add to solver
                solver.add(z3_formula)
            
            // STEP 3: Solve
            z3_result = solver.check()
            
            is_sat = (z3_result == Z3.SAT)
            solver_result = str(z3_result).lower()
            
            // Get model if SAT
            model = None
            IF is_sat:
                model = solver.model()
            
            execution_time = CURRENT_TIME() - start_time
            
            IF NOT silent:
                PRINT(f"Verified {constraints.length} constraints: {solver_result.upper()}")
                PRINT(f"Time: {execution_time:.2f}s")
            
            RETURN NEW BatchVerificationResult(
                is_valid=True,
                is_satisfiable=is_sat,
                solver_result=solver_result,
                execution_time=execution_time,
                num_constraints=constraints.length,
                model=model,
                errors=[]
            )
        
        CATCH Exception AS e:
            execution_time = CURRENT_TIME() - start_time
            
            RETURN NEW BatchVerificationResult(
                is_valid=False,
                is_satisfiable=False,
                solver_result="error",
                execution_time=execution_time,
                num_constraints=constraints.length,
                errors=[str(e)]
            )
        
        FINALLY:
            IF silent:
                STDOUT = old_stdout
```

---

## 13. SMT Encoding Module

```
CLASS GenericGlobalConsistencyChecker:
    ATTRIBUTES:
        metamodel: Metamodel
        solver: Z3Solver
        shared_vars: Dict  // Shared Z3 variables
        scope: Dict<String, Integer>
    
    METHOD __init__(metamodel: Metamodel):
        self.metamodel = metamodel
        self.solver = NEW Z3Solver()
        self.shared_vars = {}
    
    METHOD encode_constraint(ocl: String, context: String, 
                            scope: Dict) -> Z3Formula:
        """
        Main encoding entry point
        
        INPUT:
            - ocl: OCL constraint text
            - context: Context class name
            - scope: Instance limits per class
        
        OUTPUT: Z3 formula (Bool)
        """
        
        self.scope = scope
        
        // STEP 1: Initialize Z3 variables
        self._initialize_variables(scope)
        
        // STEP 2: Detect pattern
        pattern = self._detect_pattern(ocl)
        
        // STEP 3: Encode based on pattern
        formula = self._encode_by_pattern(pattern, ocl, context)
        
        RETURN formula
    
    METHOD _initialize_variables(scope: Dict) -> Void:
        """
        Create Z3 variables for all classes, attributes, and associations
        """
        
        FOR each class IN metamodel.get_all_classes():
            class_name = class.name
            n = scope.get(f"n{class_name}", 5)
            
            // 1. Presence variables (which instances exist)
            presence_vars = []
            FOR i FROM 0 TO n:
                var = Bool(f"{class_name}_{i}_present")
                presence_vars.append(var)
            shared_vars[f"{class_name}_presence"] = presence_vars
            
            // 2. Attribute variables
            FOR each attr IN class.attributes:
                attr_vars = []
                FOR i FROM 0 TO n:
                    IF attr.type == "EInt" OR attr.type == "EDouble":
                        var = Int(f"{class_name}_{i}_{attr.name}")
                    ELSE IF attr.type == "EString":
                        var = String(f"{class_name}_{i}_{attr.name}")
                    ELSE IF attr.type == "EBoolean":
                        var = Bool(f"{class_name}_{i}_{attr.name}")
                    attr_vars.append(var)
                shared_vars[f"{class_name}.{attr.name}"] = attr_vars
            
            // 3. Association variables
            FOR each assoc IN class.associations:
                target_class = assoc.target_type
                n_target = scope.get(f"n{target_class}", 5)
                
                IF assoc.is_collection:  // multiplicity *
                    // Use boolean matrix: matrix[i][j] = True means link exists
                    matrix = []
                    FOR i FROM 0 TO n:
                        row = []
                        FOR j FROM 0 TO n_target:
                            var = Bool(f"{class_name}_{i}_{assoc.name}_{j}")
                            row.append(var)
                        matrix.append(row)
                    shared_vars[f"{class_name}.{assoc.name}"] = matrix
                ELSE:  // functional (0..1 or 1..1)
                    // Use integer function: func[i] = j means instance i links to j
                    func_vars = []
                    present_vars = []  // For optional (0..1)
                    FOR i FROM 0 TO n:
                        var = Int(f"{class_name}_{i}_{assoc.name}")
                        func_vars.append(var)
                        IF assoc.lower_bound == 0:  // optional
                            present = Bool(f"{class_name}_{i}_{assoc.name}_present")
                            present_vars.append(present)
                    shared_vars[f"{class_name}.{assoc.name}"] = func_vars
                    IF present_vars.length > 0:
                        shared_vars[f"{class_name}.{assoc.name}_present"] = present_vars
    
    METHOD _detect_pattern(ocl: String) -> String:
        """
        Detect OCL pattern for optimized encoding
        """
        
        // Size constraints
        IF REGEX_MATCH(r'->size\(\)', ocl):
            RETURN "size_constraint"
        
        // Quantifiers
        IF "forAll" IN ocl:
            RETURN "forall_nested"
        IF "exists" IN ocl:
            RETURN "exists_nested"
        
        // Implications
        IF " implies " IN ocl:
            RETURN "boolean_guard_implies"
        
        // Collection operations
        IF "->select" IN ocl:
            RETURN "select_filter"
        IF "->reject" IN ocl:
            RETURN "reject_filter"
        IF "->collect" IN ocl:
            RETURN "collect_map"
        
        // Uniqueness
        IF "isUnique" IN ocl:
            RETURN "uniqueness_constraint"
        
        // Closure
        IF "->closure" IN ocl:
            RETURN "closure_operation"
        
        // Default: attribute comparison
        RETURN "attribute_comparison"
    
    METHOD _encode_by_pattern(pattern: String, ocl: String, 
                             context: String) -> Z3Formula:
        """
        Dispatch to pattern-specific encoder
        """
        
        IF pattern == "size_constraint":
            RETURN self._encode_size_constraint(ocl, context)
        ELSE IF pattern == "forall_nested":
            RETURN self._encode_forall_nested(ocl, context)
        ELSE IF pattern == "exists_nested":
            RETURN self._encode_exists_nested(ocl, context)
        ELSE IF pattern == "boolean_guard_implies":
            RETURN self._encode_boolean_guard_implies(ocl, context)
        ELSE IF pattern == "uniqueness_constraint":
            RETURN self._encode_uniqueness_constraint(ocl, context)
        ELSE IF pattern == "closure_operation":
            RETURN self._encode_closure_operation(ocl, context)
        ELSE:
            RETURN self._encode_attribute_comparison(ocl, context)
    
    METHOD _encode_size_constraint(ocl: String, context: String) -> Z3Formula:
        """
        Encode: self.collection->size() op value
        
        Example: self.rentals->size() > 5
        """
        
        // Parse OCL
        match = REGEX_SEARCH(r'self\.(\w+)->size\(\)\s*([><=]+)\s*(\d+)', ocl)
        collection = match.group(1)
        operator = match.group(2)
        value = INTEGER(match.group(3))
        
        // Get variables
        n_context = scope[f"n{context}"]
        context_presence = shared_vars[f"{context}_presence"]
        collection_matrix = shared_vars[f"{context}.{collection}"]
        
        // Find target class
        target_class = metamodel.get_class(context).get_association(collection).target_type
        n_target = scope[f"n{target_class}"]
        target_presence = shared_vars[f"{target_class}_presence"]
        
        // Encode: For each context instance, count collection size
        formulas = []
        FOR i FROM 0 TO n_context:
            // Count: how many targets are in collection?
            count_terms = []
            FOR j FROM 0 TO n_target:
                // If target j is in collection of instance i
                in_collection = And(
                    target_presence[j],
                    collection_matrix[i][j]
                )
                count_terms.append(If(in_collection, 1, 0))
            
            count = Sum(count_terms)
            
            // Apply operator
            IF operator == ">":
                constraint = count > value
            ELSE IF operator == ">=":
                constraint = count >= value
            ELSE IF operator == "<":
                constraint = count < value
            ELSE IF operator == "<=":
                constraint = count <= value
            ELSE IF operator == "=":
                constraint = count == value
            
            // If context instance exists, constraint must hold
            formulas.append(Implies(context_presence[i], constraint))
        
        // All instances must satisfy
        RETURN And(formulas)
    
    METHOD _encode_forall_nested(ocl: String, context: String) -> Z3Formula:
        """
        Encode: self.collection->forAll(x | x.attr op value)
        
        Example: self.rentals->forAll(r | r.amount > 0)
        """
        
        // Parse OCL
        match = REGEX_SEARCH(
            r'self\.(\w+)->forAll\(\w+\s*\|\s*\w+\.(\w+)\s*([><=]+)\s*(\d+)\)',
            ocl
        )
        collection = match.group(1)
        attribute = match.group(2)
        operator = match.group(3)
        value = INTEGER(match.group(4))
        
        // Get variables
        n_context = scope[f"n{context}"]
        context_presence = shared_vars[f"{context}_presence"]
        collection_matrix = shared_vars[f"{context}.{collection}"]
        
        // Find target class
        target_class = metamodel.get_class(context).get_association(collection).target_type
        n_target = scope[f"n{target_class}"]
        target_presence = shared_vars[f"{target_class}_presence"]
        target_attr = shared_vars[f"{target_class}.{attribute}"]
        
        // Encode: For each context, all elements in collection must satisfy
        formulas = []
        FOR i FROM 0 TO n_context:
            FOR j FROM 0 TO n_target:
                // If target j is in collection of context i
                in_collection = And(
                    context_presence[i],
                    target_presence[j],
                    collection_matrix[i][j]
                )
                
                // Then attribute must satisfy condition
                IF operator == ">":
                    condition = target_attr[j] > value
                ELSE IF operator == ">=":
                    condition = target_attr[j] >= value
                ELSE IF operator == "<":
                    condition = target_attr[j] < value
                ELSE IF operator == "<=":
                    condition = target_attr[j] <= value
                ELSE IF operator == "=":
                    condition = target_attr[j] == value
                
                formulas.append(Implies(in_collection, condition))
        
        RETURN And(formulas)
    
    METHOD _encode_closure_operation(ocl: String, context: String) -> Z3Formula:
        """
        Encode: self.association->closure(x | x.association)
        
        Example: self.parent->closure(p | p.parent)
        Computes transitive closure of association
        """
        
        // Parse OCL
        match = REGEX_SEARCH(r'self\.(\w+)->closure\(\w+\s*\|\s*\w+\.(\w+)\)', ocl)
        assoc1 = match.group(1)
        assoc2 = match.group(2)  // Usually same as assoc1
        
        // Get variables
        n_context = scope[f"n{context}"]
        context_presence = shared_vars[f"{context}_presence"]
        rel_matrix = shared_vars[f"{context}.{assoc1}"]  // Base relation
        
        // Compute transitive closure using fixed-point
        // closure[i][j] = True if j is reachable from i
        closure_matrix = []
        FOR i FROM 0 TO n_context:
            row = []
            FOR j FROM 0 TO n_context:
                // Initialize with base relation
                closure_var = Bool(f"closure_{context}_{i}_{j}")
                row.append(closure_var)
            closure_matrix.append(row)
        
        // Transitivity constraints
        formulas = []
        FOR i FROM 0 TO n_context:
            FOR j FROM 0 TO n_context:
                // Base case: if rel[i][j] then closure[i][j]
                formulas.append(Implies(rel_matrix[i][j], closure_matrix[i][j]))
                
                // Transitive case: if closure[i][k] and rel[k][j] then closure[i][j]
                FOR k FROM 0 TO n_context:
                    formulas.append(
                        Implies(
                            And(closure_matrix[i][k], rel_matrix[k][j]),
                            closure_matrix[i][j]
                        )
                    )
        
        RETURN And(formulas)
```

---

## 14. Pattern Detection Module

```
CLASS ComprehensivePatternDetector:
    ATTRIBUTES:
        pattern_regexes: Dict<String, String>  // Pattern name -> regex
    
    METHOD __init__():
        self.pattern_regexes = self._initialize_patterns()
    
    METHOD _initialize_patterns() -> Dict:
        """
        Define regex patterns for all OCL constructs
        """
        RETURN {
            "size_constraint": r'->size\(\)\s*[><=]+\s*\d+',
            "not_empty": r'->notEmpty\(\)',
            "is_empty": r'->isEmpty\(\)',
            "forall": r'->forAll\(',
            "exists": r'->exists\(',
            "select": r'->select\(',
            "reject": r'->reject\(',
            "collect": r'->collect\(',
            "implies": r'\simplies\s',
            "and": r'\sand\s',
            "or": r'\sor\s',
            "not": r'not\(',
            "isUnique": r'->isUnique\(',
            "closure": r'->closure\(',
            "includes": r'->includes\(',
            "excludes": r'->excludes\(',
            "sum": r'->sum\(',
            "product": r'->product\(',
            "count": r'->count\(',
            "union": r'->union\(',
            "intersection": r'->intersection\(',
            "attribute_access": r'self\.\w+',
            "navigation": r'self\.\w+(\.\w+)+',
            "comparison": r'[><=]+\s*\d+',
            "string_concat": r'\.concat\(',
            "string_substring": r'\.substring\(',
        }
    
    METHOD detect_patterns(ocl: String) -> List<String>:
        """
        Detect all patterns present in OCL constraint
        
        OUTPUT: List of pattern names
        """
        
        detected = []
        
        FOR pattern_name, regex IN pattern_regexes.items():
            IF REGEX_SEARCH(regex, ocl):
                detected.append(pattern_name)
        
        RETURN detected
    
    METHOD get_primary_pattern(ocl: String) -> String:
        """
        Determine primary pattern (for encoding dispatch)
        
        Prioritizes more complex patterns over simple ones
        """
        
        detected = self.detect_patterns(ocl)
        
        // Priority order (most specific to least specific)
        priority_order = [
            "closure",
            "forall",
            "exists",
            "collect",
            "select",
            "reject",
            "sum",
            "product",
            "isUnique",
            "implies",
            "size_constraint",
            "not_empty",
            "is_empty",
            "comparison",
            "navigation",
            "attribute_access"
        ]
        
        FOR pattern IN priority_order:
            IF pattern IN detected:
                RETURN pattern
        
        // Default
        RETURN "attribute_access"
```

---

## 15. Manifest Generation Module

```
CLASS ManifestGenerator:
    
    METHOD generate_manifest(constraints: List<OCLConstraint>, 
                            output_path: String) -> Void:
        """
        Generate JSONL manifest (ML-friendly format)
        
        Format: One JSON object per line
        Each line represents one constraint with full metadata
        """
        
        file = OPEN(output_path, "w")
        
        FOR each constraint IN constraints:
            // Convert constraint to dict
            manifest_entry = {
                "constraint_id": constraint.id,
                "pattern_id": constraint.pattern_id,
                "pattern_name": constraint.pattern_name,
                "category": constraint.category,
                "context": constraint.context,
                "ocl": constraint.ocl,
                "parameters": constraint.parameters,
                "metadata": constraint.metadata
            }
            
            // Write as single-line JSON
            json_line = JSON.dumps(manifest_entry)
            file.write(json_line + "\n")
        
        file.close()
        
        PRINT(f"Manifest saved to {output_path}")
        PRINT(f"Total entries: {constraints.length}")
    
    METHOD load_manifest(input_path: String) -> List<OCLConstraint>:
        """
        Load constraints from JSONL manifest
        """
        
        constraints = []
        file = OPEN(input_path, "r")
        
        FOR each line IN file:
            IF line.strip() == "":
                CONTINUE
            
            // Parse JSON
            data = JSON.loads(line)
            
            // Create constraint object
            constraint = OCLConstraint.from_dict(data)
            constraints.append(constraint)
        
        file.close()
        
        RETURN constraints
    
    METHOD generate_summary(constraints: List<OCLConstraint>,
                          output_path: String) -> Void:
        """
        Generate summary statistics JSON
        """
        
        // Compute statistics
        stats = {
            "total_constraints": constraints.length,
            "patterns": {},
            "categories": {},
            "difficulties": {},
            "contexts": {},
            "sat_unsat_split": {
                "sat": 0,
                "unsat": 0
            },
            "avg_navigation_depth": 0.0,
            "avg_quantifier_depth": 0.0
        }
        
        // Count patterns
        FOR each constraint IN constraints:
            // Pattern counts
            pattern = constraint.pattern_id
            IF pattern NOT IN stats["patterns"]:
                stats["patterns"][pattern] = 0
            stats["patterns"][pattern] += 1
            
            // Category counts
            category = constraint.category
            IF category NOT IN stats["categories"]:
                stats["categories"][category] = 0
            stats["categories"][category] += 1
            
            // Difficulty counts
            difficulty = constraint.metadata.get("difficulty", "unknown")
            IF difficulty NOT IN stats["difficulties"]:
                stats["difficulties"][difficulty] = 0
            stats["difficulties"][difficulty] += 1
            
            // Context counts
            context = constraint.context
            IF context NOT IN stats["contexts"]:
                stats["contexts"][context] = 0
            stats["contexts"][context] += 1
            
            // SAT/UNSAT split
            IF constraint.metadata.get("is_unsat", False):
                stats["sat_unsat_split"]["unsat"] += 1
            ELSE:
                stats["sat_unsat_split"]["sat"] += 1
            
            // Depths
            stats["avg_navigation_depth"] += constraint.metadata.get("navigation_depth", 0)
            stats["avg_quantifier_depth"] += constraint.metadata.get("quantifier_depth", 0)
        
        // Compute averages
        IF constraints.length > 0:
            stats["avg_navigation_depth"] /= constraints.length
            stats["avg_quantifier_depth"] /= constraints.length
        
        // Save to file
        file = OPEN(output_path, "w")
        file.write(JSON.dumps(stats, indent=2))
        file.close()
        
        PRINT(f"Summary saved to {output_path}")
```

---

## 16. Suite Controller Module

```
CLASS EnhancedSuiteController:
    ATTRIBUTES:
        config: SuiteConfiguration
        pattern_registry: PatternRegistry
        metamodel_extractor: MetamodelExtractor
        engine: BenchmarkEngineV2
        verifier: FrameworkConstraintVerifier
        metadata_enricher: MetadataEnricher
        unsat_generator: UNSATGenerator
        ast_similarity: ASTSimilarity
        semantic_similarity: SemanticSimilarity
        implication_checker: ImplicationChecker
        manifest_generator: ManifestGenerator
    
    METHOD __init__(config_path: String):
        // Load configuration
        self.config = self._load_config(config_path)
        
        // Initialize modules
        self.pattern_registry = NEW PatternRegistry()
        self.metadata_enricher = NEW MetadataEnricher()
        self.unsat_generator = NEW UNSATGenerator(seed=config.seed)
        self.ast_similarity = NEW ASTSimilarity()
        self.semantic_similarity = NEW SemanticSimilarity()
        self.implication_checker = NEW ImplicationChecker()
        self.manifest_generator = NEW ManifestGenerator()
    
    METHOD generate_suite() -> Void:
        """
        Main orchestration method - generates entire benchmark suite
        """
        
        PRINT("=" * 60)
        PRINT(f"OCL Benchmark Suite Generation")
        PRINT(f"Suite: {config.suite_name}")
        PRINT(f"Version: {config.version}")
        PRINT("=" * 60)
        
        // Process each model
        FOR each model_config IN config.models:
            PRINT(f"\nProcessing model: {model_config.name}")
            
            // Extract metamodel
            extractor = NEW MetamodelExtractor(model_config.xmi_path)
            metamodel = extractor.get_metamodel()
            
            // Initialize engine
            self.engine = NEW BenchmarkEngineV2(metamodel, seed=config.seed)
            
            // Initialize verifier
            scope = config.verification.scope OR self._default_scope(metamodel)
            self.verifier = NEW FrameworkConstraintVerifier(metamodel, scope)
            
            // Process each profile
            FOR each profile IN model_config.profiles:
                PRINT(f"\n  Profile: {profile.name}")
                self._generate_profile(metamodel, profile, model_config.name)
        
        PRINT("\n" + "=" * 60)
        PRINT("Suite generation complete!")
        PRINT("=" * 60)
    
    METHOD _generate_profile(metamodel: Metamodel, 
                            profile: ProfileConfig,
                            model_name: String) -> Void:
        """
        Generate constraints for single profile
        
        Pipeline:
        1. Generate SAT constraints
        2. Enrich metadata
        3. Generate UNSAT constraints
        3.5. Compatibility resolution (greedy algorithm)
        4. AST deduplication
        5. Semantic clustering
        6. Implication checking
        7. Verification
        8. Save outputs
        """
        
        // ===== STEP 1: GENERATE SAT CONSTRAINTS =====
        PRINT(f"\n  [STEP 1/8] Generating SAT constraints...")
        sat_constraints = engine.generate(profile)
        PRINT(f"    Generated: {sat_constraints.length} constraints")
        
        // ===== STEP 2: METADATA ENRICHMENT =====
        PRINT(f"\n  [STEP 2/8] Enriching metadata...")
        FOR each constraint IN sat_constraints:
            metadata_enricher.enrich_constraint_metadata(constraint)
        PRINT(f"    Enriched: {sat_constraints.length} constraints")
        
        // ===== STEP 3: UNSAT GENERATION =====
        IF profile.unsat_ratio > 0:
            PRINT(f"\n  [STEP 3/8] Generating UNSAT constraints...")
            all_constraints, unsat_map = unsat_generator.generate_mixed_sat_unsat_set(
                sat_constraints,
                metamodel,
                unsat_ratio=profile.unsat_ratio
            )
            unsat_count = all_constraints.length - sat_constraints.length
            PRINT(f"    Generated: {unsat_count} UNSAT constraints")
            PRINT(f"    Total: {all_constraints.length} constraints")
        ELSE:
            all_constraints = sat_constraints
            unsat_map = {}
        
        // ===== STEP 3.5: COMPATIBILITY RESOLUTION =====
        PRINT(f"\n  [STEP 3.5/8] Checking global consistency (SAT constraints only)...")
        
        // Extract only SAT constraints for compatibility check
        sat_only = [c for c in all_constraints if not c.metadata.get("is_unsat", False)]
        
        // Verify all SAT constraints together (SILENT MODE)
        result = verifier.verify_batch(sat_only, silent=True)
        
        IF result.is_satisfiable:
            PRINT(f"    ✓ All SAT constraints are compatible")
            final_sat_constraints = sat_only
        ELSE:
            PRINT(f"    ✗ Conflicts detected - applying greedy resolution...")
            
            // Apply greedy compatibility resolution
            resolver = NEW CompatibilityResolver(verifier)
            final_sat_constraints = resolver.find_compatible_subset(sat_only, metamodel)
            
            removed = sat_only.length - final_sat_constraints.length
            PRINT(f"    Removed: {removed} conflicting constraints")
            PRINT(f"    Retained: {final_sat_constraints.length} compatible constraints")
        
        // Recombine SAT + UNSAT
        unsat_only = [c for c in all_constraints if c.metadata.get("is_unsat", False)]
        all_constraints = final_sat_constraints + unsat_only
        
        // ===== STEP 4: AST DEDUPLICATION =====
        PRINT(f"\n  [STEP 4/8] AST-based deduplication...")
        before_count = all_constraints.length
        all_constraints = ast_similarity.remove_duplicates(all_constraints, threshold=0.85)
        removed = before_count - all_constraints.length
        PRINT(f"    Removed: {removed} duplicates")
        PRINT(f"    Remaining: {all_constraints.length} unique constraints")
        
        // ===== STEP 5: SEMANTIC CLUSTERING =====
        IF config.enable_semantic_similarity:
            PRINT(f"\n  [STEP 5/8] Semantic clustering...")
            clusters = semantic_similarity.cluster_constraints(all_constraints, num_clusters=5)
            
            // Add cluster IDs to metadata
            FOR cluster_id, cluster_constraints IN clusters.items():
                FOR constraint IN cluster_constraints:
                    constraint.metadata["semantic_cluster"] = cluster_id
            
            PRINT(f"    Clusters: {clusters.length}")
        
        // ===== STEP 6: IMPLICATION CHECKING =====
        IF config.enable_implication_checking:
            PRINT(f"\n  [STEP 6/8] Implication analysis...")
            implication_checker.add_implication_metadata(all_constraints)
            
            // Count implications
            impl_count = 0
            FOR constraint IN all_constraints:
                impl_count += constraint.metadata.get("implies", []).length
            PRINT(f"    Implications found: {impl_count}")
        
        // ===== STEP 7: FINAL VERIFICATION =====
        PRINT(f"\n  [STEP 7/8] Final verification (VISIBLE MODE)...")
        
        // Verify SAT constraints (visible output)
        sat_only = [c for c in all_constraints if not c.metadata.get("is_unsat", False)]
        result = verifier.verify_batch(sat_only, silent=False)
        
        // Update metadata with verification results
        FOR constraint IN sat_only:
            constraint.metadata["verification_result"] = result.solver_result
        
        // ===== STEP 8: SAVE OUTPUTS =====
        PRINT(f"\n  [STEP 8/8] Saving outputs...")
        
        output_dir = f"benchmarks/{model_name}/{profile.name}"
        CREATE_DIRECTORY(output_dir)
        
        // Save all formats
        self._save_ocl_file(all_constraints, f"{output_dir}/constraints.ocl")
        self._save_json_file(all_constraints, f"{output_dir}/constraints.json")
        self._save_sat_ocl_file(sat_only, f"{output_dir}/constraints_sat.ocl")
        self._save_unsat_ocl_file(unsat_only, f"{output_dir}/constraints_unsat.ocl")
        
        manifest_generator.generate_manifest(all_constraints, 
                                            f"{output_dir}/manifest.jsonl")
        manifest_generator.generate_summary(all_constraints,
                                           f"{output_dir}/summary.json")
        
        PRINT(f"\n    Outputs saved to: {output_dir}")
        PRINT(f"    Total constraints: {all_constraints.length}")
        PRINT(f"    SAT: {sat_only.length}")
        PRINT(f"    UNSAT: {unsat_only.length}")
    
    METHOD _save_ocl_file(constraints: List<OCLConstraint>, path: String) -> Void:
        file = OPEN(path, "w")
        
        // Write header
        file.write("-- OCL Constraint Benchmark\n")
        file.write(f"-- Generated: {CURRENT_TIMESTAMP()}\n")
        file.write(f"-- Total constraints: {constraints.length}\n\n")
        
        // Write constraints
        FOR i, constraint IN ENUMERATE(constraints):
            file.write(f"-- Constraint {i+1}: {constraint.pattern_name}\n")
            file.write(f"-- Context: {constraint.context}\n")
            file.write(f"{constraint.ocl}\n\n")
        
        file.close()
    
    METHOD _save_json_file(constraints: List<OCLConstraint>, path: String) -> Void:
        data = {
            "generated": CURRENT_TIMESTAMP(),
            "total_constraints": constraints.length,
            "constraints": [c.to_dict() for c in constraints]
        }
        
        file = OPEN(path, "w")
        file.write(JSON.dumps(data, indent=2))
        file.close()
```

---

## Summary

This pseudocode documentation covers **all 16 major modules** of the OCL Generation Framework:

1. **Core Module** - Data models (Pattern, Parameter, OCLConstraint)
2. **Metamodel Extraction** - XMI/Ecore parsing
3. **Pattern Registry** - Pattern loading and management
4. **Generation Engine** - Core constraint generation algorithm
5. **OCL Generator** - Template instantiation
6. **Metadata Enrichment** - Operator/depth/difficulty extraction
7. **UNSAT Generation** - 5 mutation strategies
8. **AST Similarity** - Tree edit distance for deduplication
9. **Semantic Similarity** - Transformer-based embeddings
10. **Implication Checker** - Syntactic implication detection
11. **Compatibility Resolution** - Greedy maximal compatible subset (GMCS)
12. **Verification** - Z3 SMT solver wrapper
13. **SMT Encoding** - OCL → Z3 translation with 50+ encoders
14. **Pattern Detection** - Regex-based pattern identification
15. **Manifest Generation** - JSONL and summary output
16. **Suite Controller** - Main orchestration pipeline (8 steps)

**Key Algorithms Highlighted**:
- Weighted random pattern selection
- Greedy compatibility resolution (O(n²))
- Tree edit distance for AST similarity
- Pattern-specific Z3 encoding
- Transitive closure computation
- 8-step generation pipeline

This pseudocode provides a complete understanding of the framework's implementation, suitable for inclusion in your conference paper or technical documentation.
