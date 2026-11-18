# File-by-File Fortify Remediation Guide (Using Colleague's Sanitization Pattern)

To satisfy Fortify SCA, we will implement local helper methods in each service class that rebuild strings character-by-character using `StringBuilder` and lookups against explicit allowlist constants. This breaks the "taint flow" without relying on built-in methods that Fortify ignores.

---

### Step 0: Add Allowlist Constants to `CommonConstants.java`
Make sure these constants are available globally in your project (e.g., in `com.lfar.lfarreportsservice.constants.CommonConstants` or directly in your service classes):

```java
package com.lfar.lfarreportsservice.constants;

public class CommonConstants {
    public static final String ALLOWED_CHARS = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_-(). ";
    public static final String ALLOWED_DIGITS = "0123456789";
}
```

---

### Step 1: Start with the Highest Vulnerability File — `Lfar2ServiceImpl.java`

Open `Lfar2ServiceImpl.java`. First, add these two helper methods inside the `Lfar2ServiceImpl` class:

```java
    private String sanitizeSegment(String value, String fieldName) {
        if (value == null || value.isEmpty() || value.contains("..")) {
            throw new IllegalArgumentException("Invalid characters in " + fieldName);
        }
        StringBuilder clean = new StringBuilder(value.length());
        for (int i = 0; i < value.length(); i++) {
            char c = value.charAt(i);
            int idx = CommonConstants.ALLOWED_CHARS.indexOf(c);
            if (idx < 0) {
                throw new IllegalArgumentException("Invalid character in " + fieldName);
            }
            clean.append(CommonConstants.ALLOWED_CHARS.charAt(idx));
        }
        return clean.toString();
    }

    private int sanitizeInt(Object rawValue, String fieldName) {
        if (rawValue == null) {
            throw new IllegalArgumentException(fieldName + " cannot be null");
        }
        String str = rawValue.toString().trim();
        if (str.isEmpty()) {
            throw new IllegalArgumentException(fieldName + " cannot be empty");
        }
        StringBuilder clean = new StringBuilder(str.length());
        for (int i = 0; i < str.length(); i++) {
            char c = str.charAt(i);
            int idx = CommonConstants.ALLOWED_DIGITS.indexOf(c);
            if (idx < 0) {
                throw new IllegalArgumentException("Invalid digit in " + fieldName);
            }
            clean.append(CommonConstants.ALLOWED_DIGITS.charAt(idx));
        }
        return Integer.parseInt(clean.toString());
    }
```

#### Code Replacements in `Lfar2ServiceImpl.java`:

**1. In `getData` method (around line 51):**
*Before:*
```java
Map<String, Object> data = (Map<String, Object>) map.get("data");
String submissionId = String.valueOf(data.get("submissionId"));
```
*After:*
```java
Map<String, Object> data = (Map<String, Object>) map.get("data");
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
```

**2. In `saveData` method (around line 1025):**
*Before:*
```java
String submissionId = String.valueOf(data.get("submissionId"));
```
*After:*
```java
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
```

---

### Step 2: Next File — `LfarAudiServiceImpl.java`

Add the same `sanitizeSegment` and `sanitizeInt` helper methods inside `LfarAudiServiceImpl.java`.

#### Code Replacements in `LfarAudiServiceImpl.java`:

**1. In `getData` method:**
*Before:*
```java
String reportId = (String) data.get("reportId");
String submissionId = String.valueOf(data.get("submissionId"));
```
*After:*
```java
String reportId = sanitizeSegment(String.valueOf(data.get("reportId")), "reportId");
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
```

**2. Inside the loop for table extraction:**
*Before:*
```java
String tableId = String.valueOf(questionsData.get(i).get("LFAR_SEQ"));
```
*After:*
```java
String tableId = sanitizeSegment(String.valueOf(questionsData.get(i).get("LFAR_SEQ")), "tableId");
```

**3. In `deleteTabData` method:**
*Before:*
```java
Integer deleteId = Integer.valueOf(String.valueOf(data.get("value")));
```
*After:*
```java
int deleteId = sanitizeInt(data.get("value"), "deleteId");
```