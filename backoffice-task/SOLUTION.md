# Solution

## Part 1: Bug Investigation and Fixes

### Issues Identified

#### 1. Input Validation Crash (Test 3) (debuging: 15min; documentation: 15min)

- Bug -> The Regulatory ID was not sanitized before validation, which caused the validateRegulatoryId function to fail for certain inputs (incorrect formating)
- This bug caused problems with validation and caused crashes in batch processing

---

### Solutions Implemented

1. Input Validation Crash

- I introduced a new constant cleanedId using the formatRegulatoryId function, which sanitizes and formats the regulatoryId based on the country (e.g., DE-12345-ABCD).
- ValidateRegulatoryId now uses cleanedId for regex validation eliminating crashes.
- Also I commented out problematic const cleanIid and i replaced it insted with sanitazed const cleanedId in the validation result.

---
