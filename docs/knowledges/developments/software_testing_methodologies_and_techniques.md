# Software Testing Methodologies and Techniques

---

## 1. Overview of Software Testing

Software testing is a critical process in the software development lifecycle (SDLC) aimed at evaluating the functionality, performance, and security of an application. It ensures that the software meets specified requirements and is free of defects before deployment.

---

## 2. Regression Testing & Verification

### 2.1 Regression Testing (Degradation Testing)
Regression testing verifies that recent code updates or modifications have not adversely affected existing functionalities.

* **Execution Scope:** 
  * *Ideal Scenario:* Re-run the full suite of test cases to guarantee zero side effects.
  * *Practical Scenario:* When full execution is restricted by time or budget, select and prioritize test cases based on impacted components and risk analysis.
* **Mandatory Policy:** Must be conducted regardless of the size of the code change—even minor bug fixes can introduce unexpected side effects.
* **Evaluation Method:** Compare the current test outcomes against historical baselines (previous run results) to identify unintended differences or regressions.

### 2.2 Source Comparison (Code Diffing)
* **Definition:** A static analysis process of reviewing code diffs (e.g., using `git diff` or comparison tools) to confirm that no unintended files or lines of code were modified.
* **Note:** Source comparison is a verification activity rather than a dynamic software test, serving as a safeguard prior to test execution.

---

## 3. White-Box Testing (Structural Testing)

White-box testing focuses on the internal structure, logic, and code path of the software. Testers require knowledge of the underlying source code to design test scenarios.

### 3.1 Test Coverage Criteria

Coverage measures the extent to which the application's source code is executed during testing.

| Level | Name | Description | Example / Formula |
| :--- | :--- | :--- | :--- |
| **C0** | **Statement Coverage** (Instruction Coverage) | Ensures every executable statement in the code is run at least once. | $\text{Coverage} = \frac{\text{Executed Statements}}{\text{Total Statements}} \times 100\%$ |
| **C1** | **Branch Coverage** (Decision Coverage) | Ensures every control structure outcome (each `true` and `false` branch of conditional statements like `if/else`) is evaluated at least once. | Evaluates both outcomes of an `if (A)` branch. |
| **C2** | **Condition Coverage** | Ensures every individual sub-condition within a complex boolean expression evaluates to both `true` and `false` at least once across test runs. | For `if (A AND B)`, both `A` and `B` must be tested independently as `true` and `false`. |

---

## 4. Black-Box Testing (Specification-Based Testing)

Black-box testing evaluates the functionality of an application without examining its internal code structure. Inputs are provided, and outputs are verified against business requirements.

### 4.1 Equivalence Partitioning
* **Concept:** Divides the input domain into non-overlapping groups (partitions) where the system is expected to exhibit identical behavior for any value within the group.
* **Objective:** Selects one representative sample from each valid and invalid partition, maximizing test coverage while eliminating redundant tests.

### 4.2 Boundary Value Analysis (BVA)
* **Concept:** Focuses on testing at the boundaries between equivalence partitions, as errors frequently occur around edge cases.
* **Usage:** For a valid input range $[A, B]$, tests typically evaluate boundary values: $A-1$, $A$, $A+1$, $B-1$, $B$, and $B+1$.

### 4.3 Decision Table Testing
* **Concept:** A tabular method used to map combinations of complex business rules, conditions, and their resulting actions.
* **Application:** Ideal for business logic involving multiple interdependent inputs (e.g., discounts based on user tier, cart total, and promotional codes).

```
+--------------------------+---+---+---+---+
| Conditions               | 1 | 2 | 3 | 4 |
+--------------------------+---+---+---+---+
| Is Premium Member?       | Y | Y | N | N |
| Order Value > $100?      | Y | N | Y | N |
+--------------------------+---+---+---+---+
| Actions                  |   |   |   |   |
+--------------------------+---+---+---+---+
| Apply 20% Discount       | X |   |   |   |
| Apply 10% Discount       |   | X | X |   |
| Apply Standard Shipping |   |   |   | X |
+--------------------------+---+---+---+---+
```

---

## 5. Root Cause Analysis (RCA)

### 5.1 The 5 Whys Technique
* **Concept:** An iterative interrogative technique used to explore the cause-and-effect relationships underlying a particular defect or failure.
* **Process:** Asking "Why?" repeatedly (typically 5 times) moves beyond surface symptoms to identify the root cause of an issue.

#### Example Scenario:
1. **Why did the application crash?**  
   *Because an unhandled null pointer exception occurred.*
2. **Why was the exception thrown?**  
   *Because the API response returned an unexpected `null` field.*
3. **Why did the API return a `null` field?**  
   *Because the database record was missing required attributes.*
4. **Why were attributes missing in the database?**  
   *Because data validation was skipped during the legacy user migration.*
5. **Why was validation skipped during migration?**  
   *Because the migration script lacked automated integration tests to verify data integrity.*  
   **(Root Cause: Inadequate testing in data migration pipeline)**