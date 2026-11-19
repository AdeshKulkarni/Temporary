# Remaining Files Remediation Guide (Colleague's StringBuilder Pattern)

To clear all remaining **Access Control: Database** issues from your scan, add the local `sanitizeSegment` and `sanitizeInt` helper methods to each of the files below and apply the exact variable replacements.

---

### Local Helper Methods (Copy into each class below)
Ensure these two methods are present inside **`Lfar2_1ServiceImpl`**, **`Lfar3ServiceImpl`**, **`Lfar3ComplianceServiceImpl`**, and **`Lfar2ComplianceServiceImpl`**:

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

---

### 1. `Lfar2_1ServiceImpl.java` (Clears 7 Issues: Lines 63, 67, 71, 78, 92, 98, 220, 259)

**A. In `getData` method:**
*Find:*
```java
String submissionId = String.valueOf(data.get("submissionId"));
String reportId = String.valueOf(data.get("reportId"));
```
*Replace with:*
```java
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
String reportId = sanitizeSegment(String.valueOf(data.get("reportId")), "reportId");
```
*(Note: Because `submissionId` and `reportId` are sanitized right at the top, this automatically resolves lines 63, 67, 71, 78, 92, 98, and 259).*

**B. In `deleteData` method (around line 217):**
*Find:*
```java
int deleteId = Integer.parseInt(String.valueOf(data.get("value")));
```
*(or if you still have `centralizedSecurityValidator.validateDeleteAccess(...)` there, replace it)*
*Replace with:*
```java
int deleteId = sanitizeInt(data.get("value"), "deleteId");
```

---

### 2. `Lfar3ServiceImpl.java` (Clears 4 Issues: Lines 62, 76, 82, 188, 234)

**A. In `getData` method:**
*Find:*
```java
String submissionId = String.valueOf(data.get("submissionId"));
String reportId = String.valueOf(data.get("reportId"));
```
*Replace with:*
```java
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
String reportId = sanitizeSegment(String.valueOf(data.get("reportId")), "reportId");
```
*(This clears lines 62, 76, 82, and 234).*

**B. In `deleteData` method (around line 182):**
*Find:*
```java
String deleteId =(data.get("value")).toString();
boolean exists = crsLfarRepository.existsById(Integer.parseInt(deleteId));
log.info("exists : " + exists);
if(exists){
    crsLfarRepository.deleteById(Integer.parseInt(deleteId));
```
*Replace with:*
```java
int cleanDeleteId = sanitizeInt(data.get("value"), "deleteId");
boolean exists = crsLfarRepository.existsById(cleanDeleteId);
log.info("exists : " + exists);
if(exists){
    crsLfarRepository.deleteById(cleanDeleteId);
```

---

### 3. `Lfar3ComplianceServiceImpl.java` (Clears 3 Issues: Lines 49, 57, 62, 132)

**A. In `getComplianceData` method:**
*Find:*
```java
String submissionId = String.valueOf(data.get("submissionId"));
String reportId = String.valueOf(data.get("reportId"));
```
*Replace with:*
```java
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
String reportId = sanitizeSegment(String.valueOf(data.get("reportId")), "reportId");
```
*(This clears lines 49, 57, and 62).*

**B. In `saveComplianceData` method (around line 125):**
*Find:*
```java
String rowId = rowData.get(rowData.size() - 2);
log.info("rowId : " + rowId);
if (rowId == null || rowId.equals("") || rowId.equals(" ")) {
    throw new IllegalArgumentException("Row Id cannot be null or empty");
}
CrsLfar crsLfar = crsLfarRepository.findById(Integer.parseInt(rowId)).get();
```
*Replace with:*
```java
int cleanRowId = sanitizeInt(rowData.get(rowData.size() - 2), "rowId");
log.info("rowId : " + cleanRowId);
CrsLfar crsLfar = crsLfarRepository.findById(cleanRowId).get();
```

---

### 4. `Lfar2ComplianceServiceImpl.java` (Clears 2 Issues: Lines 45, 53)

**A. In `getComplianceData` method:**
*Find:*
```java
String reportId = String.valueOf(data.get("reportId"));
```
*Replace with:*
```java
String reportId = sanitizeSegment(String.valueOf(data.get("reportId")), "reportId");
```
*(This clears both lines 45 and 53).*

---

### 5. `LfarAudiServiceImpl.java` (Clears Remaining Issues: Lines 159, 293)

*(Assuming you already added `sanitizeSegment` and `sanitizeInt` to this file)*

**A. In `saveQuestion` method (around line 157):**
*Find:*
```java
if (lfarDataRepository.existsById(new CrsLfarDataId(branch_code, formatter.parse(quarterEndDate), reportId, questionData.get("LFAR_SEQ").toString()))) {
    CrsLfarData crs = lfarDataRepository.findById(new CrsLfarDataId(branch_code, formatter.parse(quarterEndDate), reportId, questionData.get("LFAR_SEQ").toString())).get();
```
*Replace with:*
```java
String cleanSeq = sanitizeSegment(String.valueOf(questionData.get("LFAR_SEQ")), "LFAR_SEQ");
CrsLfarDataId dataId = new CrsLfarDataId(branch_code, formatter.parse(quarterEndDate), reportId, cleanSeq);

if (lfarDataRepository.existsById(dataId)) {
    CrsLfarData crs = lfarDataRepository.findById(dataId).get();
```

**B. In `deleteTabData` method (around line 274 - 293):**
*Find:*
```java
Integer deleteId = Integer.valueOf(String.valueOf(data.get("value")));
String branch_code = user.get("branch_code");
String quarterEndDate = user.get("quarterEndDate");
String reportId = data.get("reportId").toString();
String tableId = data.get("tableId").toString();
String submissionId = data.get("submissionId").toString();
```
*Replace with (Notice how `submissionId` at line 290 was still tainted! All fields inside `CrsLfarTableDataId` must be sanitized):*
```java
int deleteId = sanitizeInt(data.get("value"), "deleteId");
String branch_code = sanitizeSegment(user.get("branch_code"), "branch_code");
String quarterEndDate = user.get("quarterEndDate");
String reportId = sanitizeSegment(String.valueOf(data.get("reportId")), "reportId");
String tableId = sanitizeSegment(String.valueOf(data.get("tableId")), "tableId");
String submissionId = sanitizeSegment(String.valueOf(data.get("submissionId")), "submissionId");
```