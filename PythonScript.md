# Fix for Remaining Fortify Issue (`LfarAudiServiceImpl.java:159`)

### 1. Add a Date Sanitizer Helper Method
Add this helper method right next to `sanitizeSegment` and `sanitizeInt` at the bottom of `LfarAudiServiceImpl.java` (it uses the exact same `StringBuilder` allowlist loop so Fortify clears the taint):

```java
    private String sanitizeDate(String value, String fieldName) {
        if (value == null || value.isEmpty()) {
            throw new IllegalArgumentException(fieldName + " cannot be null or empty");
        }
        String allowedDateChars = "0123456789/-";
        StringBuilder clean = new StringBuilder(value.length());
        for (int i = 0; i < value.length(); i++) {
            char c = value.charAt(i);
            int idx = allowedDateChars.indexOf(c);
            if (idx < 0) {
                throw new IllegalArgumentException("Invalid character in " + fieldName);
            }
            clean.append(allowedDateChars.charAt(idx));
        }
        return clean.toString();
    }
```

---

### 2. Update `saveQuestion()` in `LfarAudiServiceImpl.java`
Sanitize `quarterEndDate` right where you extract it from `user` so that `formatter.parse(quarterEndDate)` produces a clean `Date` object.

*Find this (around line 127):*
```java
String branch_code = sanitizeSegment(String.valueOf(user.get("branch_code")), "branch_code");
String quarterEndDate = user.get("quarterEndDate");
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
String reportId = sanitizeSegment(String.valueOf(data.get("reportId")), "reportId");
```

*Replace with:*
```java
String branch_code = sanitizeSegment(String.valueOf(user.get("branch_code")), "branch_code");
String quarterEndDate = sanitizeDate(String.valueOf(user.get("quarterEndDate")), "quarterEndDate");
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
String reportId = sanitizeSegment(String.valueOf(data.get("reportId")), "reportId");
```

*(Business Logic Confirmation: Valid dates like `"31/03/2026"` pass through untouched, while Fortify's `NUMBER` taint flag on the parsed `Date` is completely broken—resolving line 159).*