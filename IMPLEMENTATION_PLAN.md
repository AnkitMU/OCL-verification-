# Implementation Plan: Concrete Fixes for Framework Correctness

## Overview
This document outlines 10 actionable fixes to materially improve the correctness and debuggability of the hybrid OCL-to-SMT framework.

---

## ✅ Fix 1: Date Comparison Integration

### Status: date_adapter.py EXISTS but NOT INTEGRATED

### Current Issue:
- Date fields (startDate, endDate, dateFrom, dateTo) modeled as EString in XMI
- Currently compared lexicographically as Int without proper mapping
- Lines 755-768 in `enhanced_smt_encoder.py` create Int vars but don't detect/handle dates

### Fix Implementation:
**Option A (Chosen)**: Symbolic Int mapping with ordinal indices
- Date fields → Int ordinals for Z3 comparison
- DateAdapter already implements symbolic strategy

### Files to Modify:
1. `enhanced_smt_encoder.py`:
   - Import DateAdapter at top
   - Add `self.date_adapter = DateAdapter(strategy='symbolic')` to `__init__`
   - Modify `encode_numeric_comparison()` to detect dates and use ordinals
   - Modify `encode_contractual_temporal()` to handle date comparisons

2. Integration points:
   - Lines 462-487: `encode_numeric_comparison()` - add date detection
   - Lines 750-768: `encode_contractual_temporal()` - add date detection for comparisons

---

## 🔧 Fix 2: isEmpty on Optional References

### Status: PARTIAL (multiplicity helpers exist, needs isEmpty/notEmpty handling)

### Current Issue:
- `self.vehicle->isEmpty() or self.vehicle.branch = self.branch` requires optional presence modeling
- Lines 166-183 have multiplicity helpers BUT not used in boolean_guard_implies or contractual_temporal

### Fix Implementation:
For optional [0..1] references:
1. Create presence bit `vehicle_present`
2. If present, enforce constraints
3. Encode properly in violation form

### Files to Modify:
1. `enhanced_smt_encoder.py`:
   - `encode_boolean_guard_implies()` (lines 581-650)
   - `encode_contractual_temporal()` (lines 712-783)
   
Add logic to:
- Detect `->notEmpty()` / `->isEmpty()` guards
- Create presence bits for optional references
- Tie presence to association multiplicity

---

## ✅ Fix 3: size() Must Equal Sum of Membership Bits

### Status: HELPER EXISTS (_tie_size_to_presence at line 181), needs audit

### Current Issue:
Size variables may float independently without constraint

### Fix Implementation:
Audit all size() encodings:
- `encode_size_constraint()` - lines 284-336
- `encode_uniqueness_constraint()` - Check if it uses size
- `_encode_collection_with_multiplicity()` - lines 154-156 (already ties size!)

### Action:
Run audit to ensure `_tie_size_to_presence()` is called wherever size vars created

---

## 🔧 Fix 4: Distinctness for isUnique

### Status: NEEDS FIX

### Current Issue:
- Lines 94-147 in `pattern_encoders.py`: UniquenessConstraintEncoder
- Lines 133-140 check distinctness over ALL targets, not just present members

### Fix Implementation:
```python
# Current (WRONG):
violations = []
for j1 in range(target_scope):
    for j2 in range(j1 + 1, target_scope):
        violation = And(target_bits[j1], target_bits[j2], attrs[j1] == attrs[j2])
        violations.append(violation)

# Should be (CORRECT):
# Already correct! It checks target_bits[j1] AND target_bits[j2]
# No fix needed - code is already correct
```

### Action:
Verify current implementation is correct (it appears to be)

---

## ⚠️ Fix 5: Quantifier Scoping with Finite Domains

### Status: NEEDS VERIFICATION

### Current Issue:
forAll/exists may bind over all instances instead of collection members

### Fix Implementation:
Check `encode_forall_nested()` (lines 824-876) and `encode_exists_nested()` (lines 884-924)

Current code at lines 872-874:
```python
violations = [And(target_bits[i], Not(conds[i])) for i in range(scope)]
```
✅ This DOES tie to presence bits correctly!

### Action:
Verify exists_nested also ties to collection membership

---

## 🔧 Fix 6: Enhance Normalization for Boolean Operations

### Status: PARTIAL (isEmpty→notEmpty exists, needs distributive law)

### Current Issue:
`A and (B or C)` not normalized to `(A and B) or (A and C)`
Prevents proper pattern detection for complex guards

### Fix Implementation:
Add to `ocl_normalizer.py`:
1. Distributive law: `A and (B or C)` → `(A and B) or (A and C)`
2. Implication expansion: `P implies Q` → `not P or Q` (may not be needed)
3. Double negation: `not not P` → `P` (check if exists)

### Files to Modify:
`src/ssr_ocl/super_encoder/ocl_normalizer.py`

---

## ✅ Fix 7: Attribute Resolution False Positives

### Status: ALREADY FIXED (from summary)

Phase 4 in test_enhanced_framework.py already checks attributes before reporting "missing association"

### Verification Needed:
Run test to confirm no false positives for credits, maxSeats, etc.

---

## 🔧 Fix 8: SMT Sort Hygiene

### Status: NEEDS AUDIT AND FIX

### Current Issue:
- Numeric literals may not use IntVal()/RealVal()
- Real attributes (gpa, dailyRate, tankLevel) may get Int vars

### Fix Implementation:
1. Audit all literal creation in enhanced_smt_encoder.py
2. Lines 467, 494: Already using IntVal/RealVal ✅
3. Check attribute type detection: line 115-120 has `_get_attribute_type()`
4. Ensure Real vars created for EDouble/EFloat attributes

### Files to Modify:
- `enhanced_smt_encoder.py`: Audit all Int() and Real() creations
- Add attribute type inspection to use correct Z3 types

---

## 🔧 Fix 9: Policy Function for Representation Selection

### Status: NEW FEATURE NEEDED

### Current Issue:
Both functional and matrix representations exist, but no policy to choose

### Fix Implementation:
Add helper method:
```python
def _choose_representation(self, assoc: AssociationMetadata) -> str:
    """Choose functional vs matrix based on multiplicity"""
    if assoc.is_unique and assoc.upper_bound == 1:
        return 'functional'  # *..1 or 1..1
    else:
        return 'matrix'  # *..* or complex
```

### Files to Modify:
- `enhanced_smt_encoder.py`: Add policy method
- Call from encoding methods to optimize representation

---

## 🔧 Fix 10: Counterexample Pretty-Printer

### Status: NEW FEATURE NEEDED

### Current Issue:
Z3 models returned as raw vars, hard to interpret

### Fix Implementation:
Create new file: `counterexample_printer.py`

Features:
- Map indices to class names (Branch#1, Vehicle#3)
- Print association edges: "Vehicle#3.branch → Branch#1"
- Identify which clause failed
- Integrate into Phase 6 of test_enhanced_framework.py

---

## Priority Order

### HIGH PRIORITY (Correctness):
1. ✅ Fix 1: Date comparison integration (CRITICAL)
2. 🔧 Fix 2: isEmpty on optional references (CRITICAL)
3. ✅ Fix 3: size() membership bits audit (verify existing)
4. 🔧 Fix 6: Normalization enhancements (affects pattern detection)

### MEDIUM PRIORITY (Robustness):
5. ✅ Fix 4: isUnique distinctness (verify existing)
6. ⚠️ Fix 5: Quantifier scoping (verify existing)
7. 🔧 Fix 8: SMT sort hygiene (audit and fix)

### LOW PRIORITY (Optimization/UX):
8. ✅ Fix 7: Attribute resolution (already fixed)
9. 🔧 Fix 9: Representation policy (performance)
10. 🔧 Fix 10: Pretty-printer (debuggability)

---

## Testing Strategy

After each fix:
1. Run `python3 src/ssr_ocl/super_encoder/test_enhanced_framework.py`
2. Verify CarRental domain constraints pass
3. Check specific constraints:
   - DatesOrder (Fix 1)
   - ValidWindowAndBranch (Fix 2)
   - VINUniqueAcrossCompany (Fix 4)
   - LegalAgeForAnyRental (Fix 5)

---

## Legend
- ✅ Already implemented or verified
- 🔧 Needs implementation
- ⚠️ Needs verification
- ❌ Broken, needs fix
