# Solution

### Time Allocation:

- Part 1: Bug Investigation & Fixes -> 1 hour 30 minutes (debuging: 45 min, documentation 45 min)
- Part 2: Support Ticket Analysis

## Part 1: Bug Investigation and Fixes

### Issues Identified

#### 1. Input Validation Crash Bug (debuging: 15min; documentation: 15min)

- Bug -> The Regulatory ID was not sanitized before validation, which caused the validateRegulatoryId function to fail for certain inputs (incorrect formating)
- This bug caused problems with validation and caused crashes in batch processing

---

#### 2. Cashe Contamination Bug (debuging: 10min, documentation: 15min)

- Bug -> CasheKey constant used just regulatoryId as a key witch created conflicts
- This bug caused a problem where regulatory ID valid for one country (e.g. Germany) was passed to other countries (e.g. France), leading to incorrect successful validations.

---

#### 3. Performance Bug - Sequential Processing and duplicate Check (debuging: 20min, documentation: 15min)

- Bug -> processBatch was processing products one by one with an extra setTimeout delay which slowed down batch processing significantly. Also there was no check if the product
  already exists in the database.
- This bug was slowing batch processing down and could potentialy dublicate data in database

### Solutions Implemented

#### 1. Input Validation Crash Bug

- I introduced a new constant cleanedId using the formatRegulatoryId function, which sanitizes and formats the regulatoryId based on the country (e.g., DE-12345-ABCD).
- ValidateRegulatoryId now uses cleanedId for regex validation eliminating crashes.
- Also I commented out problematic const cleanIid and i replaced it insted with sanitazed const cleanedId in the validation result.

---

#### 2. Cache Contamination Bug

- I updated the cache key to include country, and this ensures that the same ID in different countries maintains separate cache entries. That way the same ID in different countries has a separate cache which solves the problem of wrong successful validations.

```
const cacheKey = `${country}:${cleanedId}`;
```

---

#### 3. Performance Bug - Sequential Processing and duplicate Check

- Replaced sequential processBatch with chunked parallel processing, significantly improving speed.
- Added a check in processSubmission to see if the product already exists in the database (findByRegulatoryId) before recording, preventing duplicates.

## Part 2: Support Ticket Analysis
