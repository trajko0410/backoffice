# Solution

### Time Allocation:

- Part 1: Bug Investigation & Fixes -> 1 hour 30 minutes (debuging: 45 min, documentation 45 min)
- Part 2: Support Ticket Analysis -> 4 hours 30 min (writing an report)

## Part 1: Bug Investigation and Fixes

### Issues Identified

#### 1. Input Validation Crash Bug (debugging: 15min; documentation: 15min)

- Bug -> The Regulatory ID was not sanitized before validation, which caused the validateRegulatoryId function to fail for certain inputs (incorrect formatting)
- This bug caused problems with validation and caused crashes in batch processing

---

#### 2. Cache Contamination Bug (debugging: 10min, documentation: 15min)

- Bug -> CacheKey constant used just regulatoryId as a key witch created conflicts
- This bug caused a problem where regulatory ID valid for one country (e.g. Germany) was passed to other countries (e.g. France), leading to incorrect successful validations.

---

#### 3. Performance Bug - Sequential Processing and duplicate Check (debugging: 20min, documentation: 15min)

- Bug -> processBatch was processing products one by one with an extra setTimeout delay which slowed down batch processing significantly. Also there was no check if the product
  already exists in the database.
- This bug was slowing batch processing down and could potentialy duplicate data in database

---

### Solutions Implemented

#### 1. Input Validation Crash Bug

- I introduced a new constant cleanedId using the formatRegulatoryId function, which sanitizes and formats the regulatoryId based on the country (e.g., DE-12345-ABCD).
- ValidateRegulatoryId now uses cleanedId for regex validation eliminating crashes.
- Also I commented out problematic const cleanId and I replaced it instead with sanitazed const cleanedId in the validation result.

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

---

## Part 2: Support Ticket Analysis

### Ticket 2847 – Data Export Generating Incomplete CSV Files

#### Ticket Prioritization

- Priority: **Critical** do it as fast as you can and as soon as you can
- Thee reason this issue is critical is that it directly affects regulatory reporting, which users (in this case, data administrators) must complete and submit. If the issue is not resolved promptly, the company could face regulatory penalties, lose investor trust, and decrease trust from end-users of its services and products. Additionally, the resolution deadline is very short (2 days), so addressing this bug must be prioritized immediately.

#### Root Cause Analysis

- The root cause of the problem appears to be related to date filtering or time zone mismatches. To analyze the issue, the export should be run with the same filters as the user who reported the bug. The CSV output should then be compared with what is displayed on the frontend.
- The database should also be checked with an SQL query to verify that the expected records exist. If the database contains correct data, the issue likely lies in the export function. In that case, the filtering logic and any pagination or limit mechanisms should be reviewed to ensure no records are skipped.
- If the database query itself has an issue, it should be corrected and tested. However, since the user reports that the frontend displays the correct data, the database query is likely not the root cause.

#### Proposed Solution

- If the filtering logic is incorrect, it should be updated to use inclusive ranges (BETWEEN or >=/<=) and dates should be normalized to the start (00:00:00) and end (23:59:59) of the day.
- Any time zone mismatches should be resolved by standardizing dates to UTC before filtering, and if user-local time is used for filtering, convert it accordingly.
- The CSV export function should also be reviewed to ensure that pagination or limits do not skip any records, iterating through all pages if necessary.
- During debugging process, additional tests should be performed using different filters, with console logging as needed, to replicate and verify the issue fully.
- If there is problem with wrong query the query should be changed and tested to see what data we get but since user said that frontend shows correct data this is not a problem

#### Implementation Estimates

The total estimated time to resolve this issue is approximately ** 11-16 hours (1.5–2.5 working days)**, sequenced to prioritize urgent reporting needs first.

**Day 1 – Urgent / Critical Tasks**  
The first day focuses on reproducing the issue and understanding the scope of missing data. This includes going through the export steps in the web interface, confirming which entries are missing, reviewing CSV generation logic, and checking relevant logs. This task is estimated to take **2 hours**. In parallel, an immediate workaround should be applied if needed, such as manually generating the missing entries for regulatory reporting, which is estimated to take **2–4 hours**.

**Optional for Day 1:**\
If time allows, initial work on the permanent fix can begin. This could include reviewing the export code or configuration, identifying the potential source of the issue, or even starting minor code adjustments. These tasks are not mandatory on Day 1 but can help accelerate the overall timeline.

---

**Day 2 – Permanent Fix Implementation**  
The second day is focused on implementing a permanent fix. This includes correcting any code or configuration issues causing the CSV export to miss recent entries, ensuring filters, date ranges, and export logic behave as intended. This is estimated at **4–6 hours**.

**Optional for Day 2:**\
 Early testing of other trial statuses or date ranges can start during this phase. If some testing was already begun on Day 1, it can be continued or partially completed. This optional overlap helps reduce tasks on Day 3.

---

**Day 3 – Testing & Deployment**  
The third day is dedicated to completing testing and deploying the fix. Unit and integration tests should confirm that the export now includes all expected entries, and regression testing ensures that other export types (e.g., “Inactive” trials) are not affected. Testing is estimated at **2 hours**, and deployment to production, including final verification with the user, is estimated at **1–2 hours**.

**Optional for Day 3:**\
 Any remaining permanent fix adjustments or additional test scenarios that were started on Day 2 can be finalized. This ensures a smooth transition to production and mitigates the risk of follow-up issues.

| Task / Phase                         | Estimate (Hours) | Day           |
| ------------------------------------ | ---------------- | ------------- |
| Investigation & Issue Reproduction   | 2                | Day 1         |
| Immediate Workaround (if urgent)     | 2–4              | Day 1         |
| Permanent Fix (Code / Config Update) | 4–6              | Day 2         |
| Unit & Integration Testing           | 2                | Day 3         |
| Deployment & Verification            | 1–2              | Day 3         |
| **Total**                            | 11–16            | ~1.5–2.5 days |

Since this ticket is highly critical and needs to be addressed as quickly as possible, the goal is to complete as much work as early as possible. Ideally, by the end of Day 1, the client should have a usable workaround in place to ensure continuity of operations.
Note: The estimates above assume that the work is perform by a single person. If multiple team members are involved, the total calendar time may be shorter.

### Ticket 2789 – Feature Request: Bulk Operations for Regulatory Status Updates

#### Ticket Prioritization

- Priority: Low creating new feature for better user experience
- Reasoning: This is not urgent issue, as it does not block any regulatory reporting or deadlines. However it significantly affects efficiency for regular staff. Currenttly when updating hundreds of drug entries manually takes 4-5h per country update.
- Implementing a bulk update feture would save time for staff, increase efficiency and it would reduce risk of human error.
- The goal would be to try to implement this feature before the end of the month so staff could use this feature before the next mass update from regulatory authorities

#### Root Cause Analysis

- The current system only allows single-entry updates for regulatory status, which is inefficient for large-scale updates and it can be prone to human errors.
- To address this, the current update process in the database should be reviewed, including how queries work (do they allow single update or bulk most likely just single) and we should analyse any constraints such as data validation, change logging/audit and relevant functions that could impact bulk updates.
- It is also important to ensure that required checks and business rules (such as user permissions) are not bypassed during bulk operations.

#### Proposed Solution

- The proposed solution for this ticket would be implementation of bulk update API that allows multiple drug entries to be updated at once. This API should validate each and every entry before updating to ensure compliance with existing rules, include logging/audit so we could trace bulk changes and this API should accept list of drug id-s along with new regulatory status (approve, rejected, under review).
- This approach ensures both efficiency and data integrity while providing traceability for all changes.

Simple example (in real we would pass $1,$2)

Single drug update query

```
UPDATE drugs
SET regulatory_status = 'Approved'
WHERE drug_id = 101;
```

bulk update

```
UPDATE drugs
SET regulatory_status = 'Approved'
WHERE drug_id IN (101, 102, 103, 104, 105);
```

Note: In production we will not use hardcoded values — all parameters (drug IDs and status) will be passed safely using placeholders (e.g., $1, $2) to prevent SQL injection and ensure maintainability.

#### Implementation Estimates

The total estimated time to implement this feature is approximately **12–19 hours (~2–2.5 working days)**. The work is split into an immediate fix that provides partial relief for staff and a long-term solution that fully integrates bulk updates into the system.

**Day 1**\
**Immediate Fix (Short-Term Workaround):**
On Day 1, the focus would be on providing staff with a temporary method to reduce workload while the permanent solution is under development. This could include preparing SQL scripts for bulk updates that can be executed by database administrators. These scripts must be carefully reviewed and logged to ensure compliance and traceability. The effort for this temporary solution is estimated at **2–3 hours**, and it would immediately reduce the need for manual record-by-record updates. However, it is not sustainable as a long-term approach, since it bypasses user permissions and lacks a user-friendly interface.

**Long-Term Fix (Full Feature Implementation):**
The permanent solution involves implementing a dedicated bulk update API with proper validation, logging, and frontend support.

On Day 1 (afternoon), after the temporary fix, work begins on designing the API contract and database query logic. This design step is expected to take 2–4 hours. Backend development follows, implementing validation for each entry and ensuring audit logs are generated for traceability. This will take approximately **4–6 hours**.

---

**Day 2**\
On Day 2, the backend implementation continues if not completed on the first day. Once the backend is stable, attention moves to the frontend. A new interface will be introduced, either through a multi-select option or a CSV upload feature, enabling staff to update multiple records at once. This frontend development is estimated at **3–4 hours**.

---

**Day 3**\
On Day 3, the work will focus on testing, quality assurance, and bug fixes. Ensuring the feature is reliable and compliant is crucial, so this phase is allocated **2–4 hours**. Finally, 1 hour will be used to create user documentation and training materials, so that staff can quickly adopt and use the new feature with confidence.

| Task / Phase                               | Estimate (Hours) | Day                |
| ------------------------------------------ | ---------------- | ------------------ |
| Immediate Fix (SQL Script Workaround)      | 2–3              | Day 1 (short-term) |
| Design API & Database Query Logic          | 2–4              | Day 1              |
| Backend Implementation (Validation + Log)  | 4–6              | Day 1–2            |
| Frontend Implementation (Multi-select/CSV) | 3–4              | Day 2              |
| Testing, QA & Bug Fixes                    | 2–4              | Day 3              |
| Documentation & User Guidance              | 1                | Day 3              |
| **Total**                                  | 12–19            | ~2–2.5 days        |

Note: The estimates above assume that the work is perform by a single person. If multiple team members are involved, the total calendar time may be shorter.

### Ticket 2848 Invalid currency transition

#### Ticket Prioritization

- Priority: High
- Reasoning: The issue currently prevents the Pricing Analyst from updating drug prices in the Swiss market, directly affecting pricing operations and potentially impacting revenue for that region. Attempts to apply a standard 10% price increase (CHF 150 → CHF 165.50) consistently trigger the "Invalid currency transition" error, indicating a likely regression in the validation logic.
- While the problem is currently isolated to the CHF/Swiss market and does not represent a system-wide outage, it affects business-critical operations and requires timely resolution. Pricing updates are time-sensitive, as delays may impact regulatory compliance and reporting deadlines.
- Although temporary workarounds exist—such as a manual override or direct database updates—these approaches carry inherent risks and are not suitable for sustained operations. Considering the reproducibility, business impact, and time sensitivity, this ticket is prioritized as High to ensure prompt investigation and resolution.

#### Root Cause Analysis

- The first step is to reproduce the issue by attempting the CHF price update (150.00 → 165.50) in a staging or test environment. This allows confirmation of the exact error message and conditions, ensuring the problem is consistent and not a one-off glitch.
- Next, application error logs should be examined, including server logs, stack traces, and error codes, to identify whether the source of the error is in the validation logic, database constraints, or any service calls. Simultaneously, the input data must be validated, confirming the currency code, numeric format, and decimal precision are correct, and checking for locale parsing issues such as comma versus period as decimal separator.
- The database state and constraints also need to be reviewed. This involves verifying that the existing CHF price is correctly stored (150.00), inspecting any triggers, constraints, or stored procedures related to pricing updates, and ensuring there is no corruption or mismatch that could prevent updates.
- Following that, the validation logic itself should be inspected. The rules governing price transitions, including the 10% increase threshold for CHF, must be confirmed to be implemented correctly. Additionally, recent changes should be reviewed, including code merges, deployments, or configuration updates since last month, with a focus on CHF handling, thresholds, rounding, and market-specific rules. Since this update worked last month, there is a clear regression risk indicating that a recent change likely introduced the bug.
- It is also important to check for environment or configuration differences across DEV, UAT, and PRD environments, ensuring that CHF thresholds and configurations are consistent. A cross-market comparison should be performed by testing updates in other markets such as EUR and USD to determine if the issue is specific to CHF or if it may have a broader impact.
- Finally, findings should be validated against official business rules to confirm that a 10% increase is indeed allowed. This step ensures that the problem is correctly classified as a bug or a misconfiguration, guiding the appropriate resolution approach.

#### Proposed Solution

- The first step is to implement a short-term fix to allow pricing updates in Switzerland (CHF) according to business rules, such as the 10% increase. If the issue stems from a misconfiguration, the CHF thresholds or currency settings in the configuration database should be updated.
- In urgent cases, a temporary workaround may be applied, including a manual override for Swiss prices with a proper audit trail, or a direct database update with logging and a rollback plan to ensure traceability and minimize risk.
- For a long-term resolution, the validation logic must be corrected to properly handle CHF price transitions. This includes ensuring that rounding, decimal precision, and currency rules are consistently applied across the system. - All changes should undergo a formal code review before deployment. Additionally, configuration alignment is essential: allowed thresholds per market should be standardized in configuration files rather than hardcoded in the codebase, and these thresholds should be documented for maintainability and clarity.
- Automated unit and integration tests should be added for CHF price updates, covering normal increases as well as edge cases. Regression testing should also be performed for other currencies, such as EUR and USD, to ensure no new issues are introduced. Before production deployment, validation should be conducted in a staging environment to confirm the fix behaves as expected.
- To prevent similar issues in the future, a release audit checklist should be implemented to ensure that market rules are not inadvertently broken during deployments. Monitoring and alerting should be added to flag any failed pricing updates in production promptly. Finally, all business rules, including CHF-specific rules, should be documented and versioned so that analysts and developers have a clear, up-to-date reference.

#### Implementation Estimates

The total estimated time to resolve this issue is approximately **24-28 (3–4 working days)**, broken down into phases and sequenced to prioritize urgent tasks first.

**Day 1**\
On day 1, the focus will be on urgent and critical tasks. This includes reproducing the issue in a test or staging environment, reviewing logs, checking the database state, examining validation logic, and analyzing recent changes. This step ensures the problem is consistently reproducible and that all relevant information is captured. Simultaneously, the immediate fix or temporary workaround should be applied, such as a configuration update or a manual database adjustment with an audit trail. These tasks are estimated to take **2–4 hours each**, and if time permits, initial work on the permanent fix can begin.

---

**Day 2**\
Day 2 is dedicated to implementing the permanent fix and starting early testing. The permanent fix involves correcting the validation logic or adjusting CHF configuration thresholds and undergoing peer review before merging. This step is expected to take **1 full day**. Optionally, unit and integration tests for CHF rules can be initiated during this phase to accelerate validation.

---

**Day 3**\
On day 3, the focus shifts to completing testing and deploying the fix. This includes finishing unit and integration tests for CHF price updates, performing regression tests for other currencies (EUR, USD), and resolving any new bugs discovered during testing. Once tests are successfully completed, deployment to UAT and production should be conducted, with validation by the Pricing Analyst. Each of these tasks is estimated to take approximately **4 hours**.

By sequencing tasks in this way—urgent issue reproduction and workaround on Day 1, permanent fix on Day 2, and testing/deployment on Day 3—the resolution process is both efficient and minimizes risk to ongoing operations.

#### Hour Estimates

| Task / Phase                          | Estimate (Hours) | Day       |
| ------------------------------------- | ---------------- | --------- |
| Investigation & Root Cause Analysis   | 2–4              | Day 1     |
| Immediate Fix / Workaround            | 2–4              | Day 1     |
| Permanent Fix (Code or Config Update) | 8                | Day 2     |
| Unit & Integration Testing (CHF)      | 4                | Day 3     |
| Regression Testing (Other Currencies) | 4                | Day 3     |
| Deployment & Verification             | 4                | Day 3     |
| **Total**                             | 24–28            | ~3–4 days |

Note: The estimates above assume that the work is perform by a single person. If multiple team members are involved, the total calendar time may be shorter.

## Overall Assessment

- The highest priority should be given to urgent bug fixes, such as Ticket 2847 (Incomplete CSV Export) and Ticket 2848 (Invalid Currency Transition). These issues directly affect critical business operations, including regulatory reporting and pricing updates, and can have financial or compliance consequences if not addressed promptly. While permanent fixes are being developed, temporary workarounds, such as manual updates or SQL scripts with proper audit logging, can be applied to ensure continuity without violating compliance or introducing additional risks.
- Medium- and low-priority tasks, including feature enhancements like Ticket 2789 (Bulk Operations for Regulatory Status Updates). These improvements are not time-critical but have a significant impact on operational efficiency. Delivering such features before scheduled mass updates or reporting deadlines will reduce manual effort, minimize human error, and increase overall staff productivity.
- Finally, to prevent similar issues in the future, a strong release checks and monitoring strategy should be in place. Pre-release checklists can confirm compliance with business rules, while real-time monitoring and alerts for critical processes—like regulatory exports or pricing updates—can quickly highlight failures so they can be addressed before they become serious problems. By combining these priorities and strategies, the system can minimize operational risk, help staff work more efficiently, and ensure that all critical processes are reliable and easy to audit.
