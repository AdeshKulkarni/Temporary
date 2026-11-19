# Final Security Remediation Guide

This guide resolves:
1. **The last remaining `Access Control: Database` (High)** issue in `LfarAudiServiceImpl.java`.
2. **All 5 `System Information Leak: External` (Medium)** issues across your controllers/services.

**Business Logic Confirmation:** Zero business logic is altered. Detailed exceptions are still logged internally on the server (`log.info` / `log.error`), while safe generic messages are returned to the client.

---

## Part 1: Resolve Remaining `Access Control: Database` (High)

### 1. `LfarAudiServiceImpl.java` -> `saveQuestion()` (Line 159)
**Why it is still flagging:** Look at the `CrsLfarDataId` object on line 156:
`new CrsLfarDataId(branch_code, formatter.parse(quarterEndDate), reportId, cleanSeq)`
While `reportId` and `cleanSeq` were sanitized, `branch_code` was pulled directly from the `user` map without sanitization! Fortify sees `branch_code` as tainted and flags the `findById(dataId)` call.

*Find this (around line 130 in `saveQuestion()`):*
```java
String branch_code = user.get("branch_code");
String quarterEndDate = user.get("quarterEndDate");
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
String reportId = sanitizeSegment(String.valueOf(data.get("reportId")), "reportId");
```

*Replace with (Sanitize `branch_code` as well):*
```java
String branch_code = sanitizeSegment(String.valueOf(user.get("branch_code")), "branch_code");
String quarterEndDate = user.get("quarterEndDate");
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
String reportId = sanitizeSegment(String.valueOf(data.get("reportId")), "reportId");
```
*(Because `branch_code` is now rebuilt via `sanitizeSegment`, all four parameters inside `CrsLfarDataId` are completely untainted, clearing line 159).*

---

## Part 2: Resolve `System Information Leak: External` (5 Medium Issues)

**What this issue is:** In your `catch` blocks, you are putting raw Java exceptions (`e.getMessage()` or the `e` object itself) directly into `responseVo.setMessage(...)` or `result.put("msg", ...)`. When the Controller returns `ResponseEntity<>(responseVo, HttpStatus.OK)`, internal stack traces, SQL errors, or class names are leaked to the external caller.
**How to resolve it:** Keep logging the actual exception on the server so your team can debug issues, but replace `e.getMessage()` and `e` in the response payload with a generic, hardcoded string.

### Issue 1: `Lfar2ServiceImpl.java` -> `getData()` (Line 316)
*Find this:*
```java
} catch (Exception e) {
    log.info("NullPointer Exception Occurred: " + e.getCause());
    e.printStackTrace();
    responseVo.setStatusCode(HttpStatus.INTERNAL_SERVER_ERROR.value());
    responseVo.setMessage(e.getMessage());
}
```
*Replace with:*
```java
} catch (Exception e) {
    log.info("NullPointer Exception Occurred: " + e.getCause());
    e.printStackTrace();
    responseVo.setStatusCode(HttpStatus.INTERNAL_SERVER_ERROR.value());
    responseVo.setMessage("Error fetching data");
}
```

---

### Issues 2 & 3: `Lfar3ComplianceServiceImpl.java` -> `saveComplianceData()` (Lines 162 & 167)
*Find this:*
```java
} catch (NullPointerException e) {
    log.error("Null Pointer Exception Occurred : {}", e.getMessage());
    result.put(STATUS, false);
    result.put("msg", e.getMessage());
} catch (Exception e) {
    log.error("Exception Occurred : {}",e.getMessage());
    // e.printStackTrace();
    result.put(STATUS, false);
    result.put("msg", e.getMessage());
}
```
*Replace with:*
```java
} catch (NullPointerException e) {
    log.error("Null Pointer Exception Occurred : {}", e.getMessage());
    result.put(STATUS, false);
    result.put("msg", "Invalid or missing required data");
} catch (Exception e) {
    log.error("Exception Occurred : {}", e.getMessage());
    // e.printStackTrace();
    result.put(STATUS, false);
    result.put("msg", "Failed to save compliance data");
}
```

---

### Issues 4 & 5: `Lfar3ServiceImpl.java` -> `saveData()` (Lines 159 & 163)
*Find this:*
```java
} catch (NullPointerException e) {
    log.info("Null Pointer Exception Occured" + e);
    result.put("status", false);
    result.put("msg", e);
} catch (Exception e) {
    log.info("Exception Occured" + e);
    result.put("status", false);
    result.put("msg", e);
}
```
*Replace with:*
```java
} catch (NullPointerException e) {
    log.info("Null Pointer Exception Occured" + e);
    result.put("status", false);
    result.put("msg", "Invalid or missing required data");
} catch (Exception e) {
    log.info("Exception Occured" + e);
    result.put("status", false);
    result.put("msg", "Failed to save data");
}
```

---

### Proactive Check (Preventing Future Leaks in Other Files)
While checking your original code, **`Lfar2_1ServiceImpl.java`** (around line 104 in `getData()`) also has a `responseVo.setMessage(e.getMessage());` line. Make sure you replace that with `responseVo.setMessage("Error fetching data");` as well so Fortify doesn't flag it on the next scan!