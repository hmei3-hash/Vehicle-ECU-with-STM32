# Test Plan

This document contains test case templates for the ECU project. Fill in each test case as you implement and verify each feature.

**Instructions:**
1. Copy a template for each new test
2. Fill in Purpose, Setup, Input, and Expected Result before testing
3. Fill in Actual Result and Status after testing
4. Commit your test results

---

## Test Case Template

```
Test ID:        [TEST_ID]
Phase:          [PHASE_NUMBER]
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Phase 0 — Hardware Bring-up Tests

### TEST-000: Development Environment Setup

```
Test ID:        TEST-000
Phase:          0
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-001: LED Blink Verification

```
Test ID:        TEST-001
Phase:          0
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-002: Serial Output Verification

```
Test ID:        TEST-002
Phase:          0
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Phase 1 — GPIO Tests

### TEST-010: GPIO Output Control

```
Test ID:        TEST-010
Phase:          1
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-011: GPIO Input Read

```
Test ID:        TEST-011
Phase:          1
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Phase 2 — Timer / Interrupt Tests

### TEST-020: Timer Interrupt Frequency

```
Test ID:        TEST-020
Phase:          2
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-021: External Interrupt

```
Test ID:        TEST-021
Phase:          2
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Phase 3 — Sensor Tests

### TEST-030: TOF Sensor Reading

```
Test ID:        TEST-030
Phase:          3
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-031: Temperature Sensor Reading

```
Test ID:        TEST-031
Phase:          3
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Phase 4 — CAN Basic Tests

### TEST-040: CAN Loopback TX/RX

```
Test ID:        TEST-040
Phase:          4
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-041: CAN Two-Node Communication

```
Test ID:        TEST-041
Phase:          4
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Phase 5 — Multi-node CAN Tests

### TEST-050: Three-Node Communication

```
Test ID:        TEST-050
Phase:          5
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-051: CAN Filter Verification

```
Test ID:        TEST-051
Phase:          5
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Phase 6 — CAN Protocol Tests

### TEST-060: Message Encode/Decode

```
Test ID:        TEST-060
Phase:          6
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Phase 7 — FreeRTOS Tests

### TEST-070: Multi-task Execution

```
Test ID:        TEST-070
Phase:          7
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-071: Inter-task Queue Communication

```
Test ID:        TEST-071
Phase:          7
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Phase 8 — System Integration Tests

### TEST-080: Full System End-to-End

```
Test ID:        TEST-080
Phase:          8
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-081: Long-Duration Soak Test

```
Test ID:        TEST-081
Phase:          8
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Phase 9 — Fault Detection Tests

### TEST-090: Sensor Fault Detection

```
Test ID:        TEST-090
Phase:          9
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-091: CAN Timeout Detection

```
Test ID:        TEST-091
Phase:          9
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

### TEST-092: Watchdog Timer Test

```
Test ID:        TEST-092
Phase:          9
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```

---

## Adding New Tests

Copy the template below and append to the appropriate phase section:

```
Test ID:        TEST-[PHASE][NUMBER]
Phase:          [PHASE_NUMBER]
Purpose:        [TEST_PURPOSE]
Setup:          [TEST_SETUP]
Input:          [TEST_INPUT]
Expected Result:[EXPECTED_RESULT]
Actual Result:  [ACTUAL_RESULT]
Status:         [PASS/FAIL]
Date:           [DATE]
Notes:          [NOTES]
```
