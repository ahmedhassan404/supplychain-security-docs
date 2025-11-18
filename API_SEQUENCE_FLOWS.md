# ChainGuard API Sequence Flows

This document describes the typical API interaction sequences for common user workflows in ChainGuard, a Dependency Analysis and Supply Chain Security Platform for Web Applications.

---

## 1. User Authentication Flow

```
1. User enters credentials
   ↓
2. POST /api/auth/login
   Body: { email, password }
   ↓
3. Backend validates credentials
   ↓
4. Returns: { token, user }
   ↓
5. Frontend stores token
   ↓
6. GET /api/auth/me (verify token)
   Headers: { Authorization: Bearer <token> }
   ↓
7. User is authenticated
```

---

## 2. Complete Project Scan Flow

```
1. User uploads project files
   ↓
2. POST /api/scan/upload
   Body: FormData { files, projectName }
   ↓
3. Returns: { scanId: "scan_123" }
   ↓
4. POST /api/scan/start
   Body: { scanId: "scan_123", options: {...} }
   ↓
5. Returns: { status: "scanning" }
   ↓
6. [Polling Loop] GET /api/scan/scan_123/status
   Every 2-3 seconds
   Returns: { progress: 65, status: "scanning" }
   ↓
7. [While scanning] GET /api/scan/scan_123/packages
   Returns: List of packages with current status
   ↓
8. When status = "complete"
   ↓
9. GET /api/scan/scan_123/results
   Returns: Complete scan results with vulnerabilities
   ↓
10. Navigate to Results page
    ↓
11. GET /api/vulnerabilities?scanId=scan_123
    Returns: All vulnerabilities from scan
    ↓
12. User views detailed vulnerability
    GET /api/vulnerabilities/1
    Returns: Detailed CVE information
    ↓
13. User applies fix
    POST /api/vulnerabilities/1/fix
    Returns: Fix confirmation
```

---

## 3. Dashboard Load Flow

```
1. User navigates to Dashboard
   ↓
2. [Parallel Requests]
   ├─ GET /api/dashboard/stats
   │  Returns: Overview statistics
   │
   ├─ GET /api/dashboard/activity?limit=20
   │  Returns: Recent activity feed
   │
   └─ GET /api/dashboard/dependency-health
      Returns: Health scores for dependencies
   ↓
3. All data loaded, dashboard rendered
   ↓
4. [Auto-refresh every 30 seconds]
   GET /api/dashboard/stats
   GET /api/dashboard/activity?limit=20
```

---

## 4. View Vulnerability Details Flow

```
1. User clicks on vulnerability in Results page
   ↓
2. GET /api/vulnerabilities/1
   Returns: Basic vulnerability info
   ↓
3. [Optional] GET /api/vulnerabilities/1/cve-details
   Returns: Detailed CVE information from external sources
   ↓
4. Display vulnerability details
   ↓
5. User clicks "Apply Fix"
   ↓
6. POST /api/vulnerabilities/1/fix
   Body: { action: "update", targetVersion: "4.17.21" }
   ↓
7. Returns: { status: "fixing", updatedPackage: {...} }
   ↓
8. Refresh vulnerability list
   GET /api/vulnerabilities?status=open
```

---

## 5. Generate Report Flow

```
1. User navigates to Reports page
   ↓
2. User selects report type and date range
   ↓
3. POST /api/reports/generate
   Body: {
     reportType: "executive",
     dateRange: "last30",
     includeCharts: true
   }
   ↓
4. Returns: { reportId: "report_123", status: "generating" }
   ↓
5. [Polling] GET /api/reports/report_123
   Every 5 seconds
   Returns: { status: "generating" } or { status: "complete" }
   ↓
6. When status = "complete"
   ↓
7. GET /api/reports/report_123?format=pdf
   Returns: PDF file download
   OR
   GET /api/reports/report_123?format=json
   Returns: JSON data
   ↓
8. User downloads report
```

---

## 6. AI Insights Flow

```
1. User navigates to Insights page
   ↓
2. [Parallel Requests]
   ├─ GET /api/insights/ml-metrics
   │  Returns: Model performance metrics
   │
   ├─ GET /api/insights/anomalies?limit=20
   │  Returns: Detected anomalies
   │
   ├─ GET /api/insights/threat-predictions
   │  Returns: Threat predictions
   │
   ├─ GET /api/insights/feature-importance
   │  Returns: Feature importance analysis
   │
   └─ GET /api/insights/summary
      Returns: Summary statistics
   ↓
3. All data loaded, insights page rendered
   ↓
4. User clicks on anomaly
   ↓
5. [Optional] GET /api/insights/anomalies/1
   Returns: Detailed anomaly information
```

---

## 7. Export Scan Results Flow

```
1. User completes scan and views results
   ↓
2. User clicks "Export Report" button
   ↓
3. User selects format (PDF, JSON, CSV)
   ↓
4. GET /api/export/scan/scan_123?format=pdf
   OR
   GET /api/export/scan/scan_123?format=json
   OR
   GET /api/export/scan/scan_123?format=csv
   ↓
5. Returns: File download
   ↓
6. File downloaded to user's device
```

---

## 8. Ignore Vulnerability Flow

```
1. User views vulnerability details
   ↓
2. User decides to ignore (false positive, etc.)
   ↓
3. POST /api/vulnerabilities/1/ignore
   Body: {
     reason: "False positive",
     expiresAt: "2024-02-20T00:00:00Z"
   }
   ↓
4. Returns: { status: "ignored" }
   ↓
5. Vulnerability no longer appears in active lists
   ↓
6. GET /api/vulnerabilities?status=open
   (Excludes ignored vulnerabilities)
```

---

## 9. Real-time Scan Progress (WebSocket Alternative)

```
[If WebSockets are implemented]

1. POST /api/scan/start
   Returns: { scanId: "scan_123" }
   ↓
2. WebSocket connection established
   ws://api/scan/scan_123/stream
   ↓
3. Backend sends progress updates:
   {
     "type": "progress",
     "progress": 65,
     "currentPackage": "express@4.18.0"
   }
   ↓
4. Backend sends package updates:
   {
     "type": "package",
     "package": {
       "name": "express",
       "status": "vulnerable"
     }
   }
   ↓
5. When complete:
   {
     "type": "complete",
     "scanId": "scan_123"
   }
   ↓
6. Close WebSocket connection
```

---

## 10. Multi-Project Dashboard Flow

```
1. User has multiple projects
   ↓
2. GET /api/projects
   Returns: List of user's projects
   ↓
3. User selects project
   ↓
4. GET /api/dashboard/stats?projectId=1
   Returns: Stats for specific project
   ↓
5. GET /api/dashboard/activity?projectId=1
   Returns: Activity for specific project
   ↓
6. All project-specific data loaded
```

---

## Error Handling Flow

```
1. API request fails
   ↓
2. Check error status code:
   
   - 401 Unauthorized
     → Token expired or invalid
     → POST /api/auth/login (refresh token)
     → Retry original request
   
   - 403 Forbidden
     → User doesn't have permission
     → Show error message
     → Redirect to appropriate page
   
   - 404 Not Found
     → Resource doesn't exist
     → Show "Not found" message
     → Navigate back or to home
   
   - 429 Too Many Requests
     → Rate limit exceeded
     → Wait and retry with exponential backoff
   
   - 500 Internal Server Error
     → Server error
     → Show generic error message
     → Log error for debugging
     → Allow user to retry
   ↓
3. Display appropriate error message to user
```

---

## Data Refresh Patterns

### Dashboard Auto-Refresh
```
Every 30 seconds:
  - GET /api/dashboard/stats
  - GET /api/dashboard/activity?limit=20
```

### Scan Progress Polling
```
While status = "scanning":
  Every 2-3 seconds:
    - GET /api/scan/{scanId}/status
    - GET /api/scan/{scanId}/packages
```

### Report Generation Polling
```
While status = "generating":
  Every 5 seconds:
    - GET /api/reports/{reportId}
```

### Insights Refresh
```
Every 60 seconds:
  - GET /api/insights/summary
  - GET /api/insights/anomalies?limit=20
```

---

## Optimistic Updates

### Apply Fix (Optimistic)
```
1. User clicks "Apply Fix"
   ↓
2. Immediately update UI (optimistic)
   - Mark vulnerability as "fixing"
   ↓
3. POST /api/vulnerabilities/1/fix
   ↓
4. On success:
   - Update UI to show "fixed"
   ↓
5. On error:
   - Revert optimistic update
   - Show error message
   - Allow retry
```

---

## Caching Strategy

### Cacheable Endpoints
- `GET /api/dashboard/stats` (cache 30 seconds)
- `GET /api/dashboard/dependency-health` (cache 5 minutes)
- `GET /api/insights/ml-metrics` (cache 5 minutes)
- `GET /api/insights/feature-importance` (cache 10 minutes)

### Non-Cacheable Endpoints
- `GET /api/dashboard/activity` (always fresh)
- `GET /api/scan/{scanId}/status` (always fresh)
- `GET /api/vulnerabilities` (with filters, always fresh)

---

These sequence flows provide a clear understanding of how the frontend should interact with the backend APIs to provide a smooth user experience.

