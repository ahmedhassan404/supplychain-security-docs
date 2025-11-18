# ChainGuard Backend API Documentation

This document describes all the backend APIs required to make the ChainGuard frontend fully functional. ChainGuard is a Dependency Analysis and Supply Chain Security Platform for Web Applications. The APIs are organized by feature/module.

---

## Table of Contents

1. [Authentication](#authentication)
2. [Dashboard](#dashboard)
3. [Project Management](#project-management)
4. [Project Scanning](#project-scanning)
5. [Dependency Analysis](#dependency-analysis)
6. [Package Management](#package-management)
7. [Vulnerability Results](#vulnerability-results)
8. [CVE Database & Threat Intelligence](#cve-database--threat-intelligence)
9. [AI Insights](#ai-insights)
10. [Suspicious Package Classification](#suspicious-package-classification)
11. [SBOM Generation](#sbom-generation)
12. [Reports](#reports)
13. [Export & Download](#export--download)
14. [Real-time Updates](#real-time-updates)

---

## Authentication

### POST /api/auth/login
**Description:** Authenticates a user and returns a JWT token.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response 200:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "name": "John Doe",
    "email": "user@example.com",
    "role": "admin"
  }
}
```

**Response 401:** Invalid email or password
```json
{
  "error": "Invalid credentials"
}
```

**Test Data Example:** 
- email: `test@test.com`
- password: `123456`

---

### POST /api/auth/register
**Description:** Registers a new user account.

**Request Body:**
```json
{
  "name": "John Doe",
  "email": "user@example.com",
  "password": "password123"
}
```

**Response 201:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "name": "John Doe",
    "email": "user@example.com",
    "role": "user"
  }
}
```

**Response 400:** Email already exists
```json
{
  "error": "Email already registered"
}
```

---

### POST /api/auth/logout
**Description:** Logs out the current user (invalidates token).

**Headers:**
```
Authorization: Bearer <token>
```

**Response 200:**
```json
{
  "message": "Logged out successfully"
}
```

---

### GET /api/auth/me
**Description:** Gets the current authenticated user's profile.

**Headers:**
```
Authorization: Bearer <token>
```

**Response 200:**
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "user@example.com",
  "role": "admin",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

**Response 401:** Unauthorized
```json
{
  "error": "Unauthorized"
}
```

---

## Dashboard

### GET /api/dashboard/stats
**Description:** Retrieves security statistics for the dashboard overview.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `timeRange` (optional): `"24h" | "7d" | "30d" | "all"` (default: "30d")

**Response 200:**
```json
{
  "critical": 24,
  "high": 156,
  "medium": 432,
  "totalVulnerabilities": 2847,
  "totalPackages": 2847,
  "trends": {
    "critical": "+3 this week",
    "high": "+12 this week",
    "medium": "-8 this week"
  },
  "scanCompletion": 94.2,
  "riskScore": 7.2,
  "issuesFixed": 87
}
```

**Test Data Example:** 
- timeRange: `30d`

---

### GET /api/dashboard/activity
**Description:** Retrieves recent activity feed for the dashboard.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `limit` (optional): number (default: 20)
- `offset` (optional): number (default: 0)

**Response 200:**
```json
{
  "activities": [
    {
      "id": 1,
      "type": "vulnerability",
      "title": "Critical CVE-2024-1234 detected",
      "package": "express@4.18.0",
      "severity": "critical",
      "time": "2 minutes ago",
      "timestamp": "2024-01-20T14:30:00Z"
    },
    {
      "id": 2,
      "type": "scan",
      "title": "Dependency scan completed",
      "package": "react-router-dom",
      "severity": "info",
      "time": "15 minutes ago",
      "timestamp": "2024-01-20T14:15:00Z"
    },
    {
      "id": 3,
      "type": "fixed",
      "title": "Vulnerability patched",
      "package": "lodash@4.17.21",
      "severity": "success",
      "time": "1 hour ago",
      "timestamp": "2024-01-20T13:30:00Z"
    }
  ],
  "total": 50,
  "limit": 20,
  "offset": 0
}
```

---

### GET /api/dashboard/dependency-health
**Description:** Retrieves health scores for major dependencies.

**Headers:**
```
Authorization: Bearer <token>
```

**Response 200:**
```json
{
  "dependencies": [
    {
      "name": "React",
      "health": 92,
      "critical": 0,
      "high": 2,
      "medium": 5,
      "low": 1
    },
    {
      "name": "Next.js",
      "health": 78,
      "critical": 1,
      "high": 5,
      "medium": 8,
      "low": 2
    },
    {
      "name": "Express",
      "health": 85,
      "critical": 0,
      "high": 3,
      "medium": 6,
      "low": 1
    },
    {
      "name": "TypeScript",
      "health": 96,
      "critical": 0,
      "high": 1,
      "medium": 2,
      "low": 0
    },
    {
      "name": "Vite",
      "health": 88,
      "critical": 0,
      "high": 2,
      "medium": 4,
      "low": 1
    }
  ]
}
```

---

## Project Management

### GET /api/projects
**Description:** Retrieves all projects for the authenticated user.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `limit` (optional): number (default: 20)
- `offset` (optional): number (default: 0)
- `sortBy` (optional): `"name" | "createdAt" | "lastScanned"` (default: "lastScanned")
- `order` (optional): `"asc" | "desc"` (default: "desc")

**Response 200:**
```json
{
  "projects": [
    {
      "id": 1,
      "name": "my-project",
      "description": "Main application project",
      "packageManager": "npm",
      "lastScanned": "2024-01-20T15:05:00Z",
      "lastScanId": "scan_1234567890",
      "totalPackages": 156,
      "vulnerablePackages": 12,
      "riskScore": 7.2,
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2024-01-20T15:05:00Z"
    }
  ],
  "total": 5,
  "limit": 20,
  "offset": 0
}
```

**Flow:**
1. Validate authentication token
2. Query database for user's projects
3. Join with latest scan data
4. Calculate risk scores and package counts
5. Return paginated results

**Test Data Example:**
- limit: `20`
- offset: `0`

---

### GET /api/projects/:projectId
**Description:** Gets detailed information about a specific project.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `projectId`: number

**Response 200:**
```json
{
  "id": 1,
  "name": "my-project",
  "description": "Main application project",
  "packageManager": "npm",
  "repository": {
    "url": "https://github.com/user/my-project",
    "type": "git"
  },
  "lastScanned": "2024-01-20T15:05:00Z",
  "lastScanId": "scan_1234567890",
  "totalPackages": 156,
  "directDependencies": 45,
  "transitiveDependencies": 111,
  "vulnerablePackages": 12,
  "riskScore": 7.2,
  "securityScore": 78.5,
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-20T15:05:00Z",
  "scanHistory": [
    {
      "scanId": "scan_1234567890",
      "completedAt": "2024-01-20T15:05:00Z",
      "vulnerabilitiesFound": 12
    }
  ]
}
```

**Response 404:** Project not found
```json
{
  "error": "Project not found"
}
```

**Flow:**
1. Validate authentication and project ownership
2. Query project details from database
3. Fetch latest scan information
4. Calculate aggregated statistics
5. Return project details

---

### POST /api/projects
**Description:** Creates a new project.

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "name": "my-new-project",
  "description": "Project description",
  "packageManager": "npm",
  "repository": {
    "url": "https://github.com/user/my-project",
    "type": "git"
  }
}
```

**Response 201:**
```json
{
  "id": 2,
  "name": "my-new-project",
  "description": "Project description",
  "packageManager": "npm",
  "createdAt": "2024-01-20T16:00:00Z",
  "message": "Project created successfully"
}
```

**Response 400:** Invalid input
```json
{
  "error": "Project name is required"
}
```

**Flow:**
1. Validate request body (name, packageManager required)
2. Check for duplicate project names for user
3. Create project record in database
4. Return created project

**Validation:**
- `name`: Required, 1-100 characters, alphanumeric and hyphens only
- `packageManager`: Required, must be one of: "npm", "yarn", "pnpm"
- `repository.url`: Optional, must be valid URL if provided

---

### PUT /api/projects/:projectId
**Description:** Updates an existing project.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `projectId`: number

**Request Body:**
```json
{
  "name": "updated-project-name",
  "description": "Updated description",
  "repository": {
    "url": "https://github.com/user/updated-project",
    "type": "git"
  }
}
```

**Response 200:**
```json
{
  "id": 1,
  "name": "updated-project-name",
  "description": "Updated description",
  "updatedAt": "2024-01-20T16:30:00Z",
  "message": "Project updated successfully"
}
```

**Response 404:** Project not found
```json
{
  "error": "Project not found"
}
```

---

### DELETE /api/projects/:projectId
**Description:** Deletes a project and all associated scans.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `projectId`: number

**Response 200:**
```json
{
  "message": "Project deleted successfully"
}
```

**Response 404:** Project not found
```json
{
  "error": "Project not found"
}
```

**Flow:**
1. Validate authentication and project ownership
2. Delete all associated scans and scan data
3. Delete project record
4. Return success message

**Security:** Only project owner can delete project

---

## Project Scanning

### POST /api/scan/upload
**Description:** Uploads project files (package.json, lock files, etc.) for scanning.

**Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Request Body (Form Data):**
- `files`: File[] (package.json, yarn.lock, pnpm-lock.yaml, package-lock.json, etc.)
- `projectName` (optional): string
- `projectId` (optional): number (for re-scanning existing project)

**Response 200:**
```json
{
  "scanId": "scan_1234567890",
  "status": "uploaded",
  "uploadedFiles": [
    {
      "filename": "package.json",
      "size": 2048,
      "type": "application/json"
    }
  ],
  "message": "Files uploaded successfully"
}
```

**Response 400:** Invalid file format
```json
{
  "error": "Invalid file format. Supported: package.json, lock files"
}
```

**Test Data Example:**
- files: `package.json`, `package-lock.json`
- projectName: `"my-project"`

---

### POST /api/scan/start
**Description:** Starts scanning uploaded project files.

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "scanId": "scan_1234567890",
  "options": {
    "includeDevDependencies": true,
    "scanTransitive": true,
    "checkCVEs": true,
    "checkLicenses": false
  }
}
```

**Response 200:**
```json
{
  "scanId": "scan_1234567890",
  "status": "scanning",
  "startedAt": "2024-01-20T15:00:00Z",
  "estimatedCompletion": "2024-01-20T15:05:00Z"
}
```

**Response 404:** Scan not found
```json
{
  "error": "Scan not found"
}
```

---

### GET /api/scan/:scanId/status
**Description:** Gets the current status and progress of a scan.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `scanId`: string

**Response 200:**
```json
{
  "scanId": "scan_1234567890",
  "status": "scanning",
  "progress": 65,
  "packagesScanned": 130,
  "totalPackages": 200,
  "currentPackage": "express@4.18.0",
  "startedAt": "2024-01-20T15:00:00Z",
  "estimatedCompletion": "2024-01-20T15:05:00Z"
}
```

**Status Values:** `"uploading" | "scanning" | "complete" | "error"`

---

### GET /api/scan/:scanId/packages
**Description:** Gets the list of packages being scanned with their current status.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `scanId`: string

**Query Parameters:**
- `status` (optional): `"scanning" | "safe" | "vulnerable"` (filter by status)

**Response 200:**
```json
{
  "packages": [
    {
      "id": 1,
      "name": "react",
      "version": "18.3.1",
      "status": "safe",
      "scannedAt": "2024-01-20T15:01:00Z"
    },
    {
      "id": 2,
      "name": "express",
      "version": "5.1.0",
      "status": "vulnerable",
      "scannedAt": "2024-01-20T15:01:30Z"
    },
    {
      "id": 3,
      "name": "typescript",
      "version": "5.9.2",
      "status": "safe",
      "scannedAt": "2024-01-20T15:02:00Z"
    }
  ],
  "total": 6,
  "safe": 4,
  "vulnerable": 2
}
```

---

### GET /api/scan/:scanId/results
**Description:** Gets complete scan results after scanning is complete.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `scanId`: string

**Response 200:**
```json
{
  "scanId": "scan_1234567890",
  "status": "complete",
  "completedAt": "2024-01-20T15:05:00Z",
  "summary": {
    "totalPackages": 6,
    "safePackages": 4,
    "vulnerablePackages": 2,
    "riskScore": 33.3
  },
  "packages": [
    {
      "id": 1,
      "name": "react",
      "version": "18.2.0",
      "status": "safe",
      "vulnerabilities": []
    },
    {
      "id": 2,
      "name": "express",
      "version": "4.18.2",
      "status": "safe",
      "vulnerabilities": []
    },
    {
      "id": 3,
      "name": "lodash",
      "version": "4.17.20",
      "status": "vulnerable",
      "vulnerabilities": [
        {
          "cve": "CVE-2024-1234",
          "severity": "critical",
          "title": "Remote Code Execution",
          "description": "Lodash versions before 4.17.21 are vulnerable to RCE",
          "fixedIn": "4.17.21",
          "publishedAt": "2024-01-10T00:00:00Z"
        }
      ]
    }
  ]
}
```

---

## Dependency Analysis

### GET /api/scan/:scanId/dependency-tree
**Description:** Retrieves the complete dependency tree structure for a scan, including all direct and transitive dependencies with their relationships.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `scanId`: string

**Query Parameters:**
- `includeTransitive` (optional): boolean (default: true)
- `depth` (optional): number (default: null, returns all levels)

**Response 200:**
```json
{
  "scanId": "scan_1234567890",
  "root": {
    "id": 1,
    "name": "my-project",
    "version": "1.0.0",
    "type": "root"
  },
  "tree": {
    "id": 1,
    "name": "my-project",
    "version": "1.0.0",
    "dependencies": [
      {
        "id": 2,
        "name": "express",
        "version": "4.18.0",
        "type": "direct",
        "status": "vulnerable",
        "vulnerabilities": [
          {
            "cve": "CVE-2024-1234",
            "severity": "critical"
          }
        ],
        "dependencies": [
          {
            "id": 3,
            "name": "accepts",
            "version": "1.3.8",
            "type": "transitive",
            "status": "safe",
            "dependencies": []
          }
        ]
      },
      {
        "id": 4,
        "name": "react",
        "version": "18.3.1",
        "type": "direct",
        "status": "safe",
        "dependencies": [
          {
            "id": 5,
            "name": "loose-envify",
            "version": "1.4.0",
            "type": "transitive",
            "status": "safe",
            "dependencies": []
          }
        ]
      }
    ]
  },
  "statistics": {
    "totalPackages": 156,
    "directDependencies": 45,
    "transitiveDependencies": 111,
    "vulnerablePackages": 12,
    "maxDepth": 5
  }
}
```

**Flow:**
1. Validate authentication and scan ownership
2. Query dependency tree from database
3. Build hierarchical tree structure
4. Enrich with vulnerability data
5. Calculate statistics
6. Return tree structure

**Test Data Example:**
- scanId: `scan_1234567890`
- includeTransitive: `true`

---

### GET /api/scan/:scanId/dependency-graph
**Description:** Retrieves dependency graph data optimized for visualization, including nodes and edges for graph rendering.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `scanId`: string

**Query Parameters:**
- `layout` (optional): `"circular" | "hierarchical" | "force-directed"` (default: "circular")
- `filter` (optional): `"all" | "vulnerable" | "direct"` (default: "all")

**Response 200:**
```json
{
  "scanId": "scan_1234567890",
  "nodes": [
    {
      "id": 1,
      "name": "my-project",
      "version": "1.0.0",
      "type": "root",
      "status": "safe",
      "x": 400,
      "y": 300,
      "radius": 30
    },
    {
      "id": 2,
      "name": "express",
      "version": "4.18.0",
      "type": "direct",
      "status": "vulnerable",
      "x": 300,
      "y": 200,
      "radius": 24,
      "vulnerabilityCount": 2
    },
    {
      "id": 3,
      "name": "react",
      "version": "18.3.1",
      "type": "direct",
      "status": "safe",
      "x": 500,
      "y": 200,
      "radius": 24
    }
  ],
  "edges": [
    {
      "id": "e1",
      "source": 1,
      "target": 2,
      "type": "direct",
      "weight": 1
    },
    {
      "id": "e2",
      "source": 1,
      "target": 3,
      "type": "direct",
      "weight": 1
    },
    {
      "id": "e3",
      "source": 2,
      "target": 4,
      "type": "transitive",
      "weight": 0.5
    }
  ],
  "metadata": {
    "totalNodes": 156,
    "totalEdges": 234,
    "vulnerableNodes": 12,
    "layout": "circular"
  }
}
```

**Flow:**
1. Validate authentication and scan ownership
2. Query dependency relationships from database
3. Calculate node positions based on layout algorithm
4. Build edge list with relationships
5. Add metadata for visualization
6. Return graph data

**Notes:**
- Node positions (x, y) are calculated server-side for consistent rendering
- Edge weights represent dependency strength (direct = 1.0, transitive = 0.5)
- Vulnerable nodes are highlighted with different styling

---

### GET /api/scan/:scanId/transitive-dependencies
**Description:** Analyzes and returns all transitive dependencies with risk assessment.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `scanId`: string

**Query Parameters:**
- `packageName` (optional): string (filter by specific package)
- `severity` (optional): `"critical" | "high" | "medium" | "low"` (filter by severity)

**Response 200:**
```json
{
  "scanId": "scan_1234567890",
  "transitiveDependencies": [
    {
      "id": 10,
      "name": "lodash",
      "version": "4.17.20",
      "introducedBy": [
        {
          "package": "express",
          "version": "4.18.0",
          "path": ["express", "lodash"]
        }
      ],
      "status": "vulnerable",
      "vulnerabilities": [
        {
          "cve": "CVE-2024-1234",
          "severity": "critical",
          "cvssScore": 9.8
        }
      ],
      "depth": 2,
      "usageCount": 3
    }
  ],
  "statistics": {
    "totalTransitive": 111,
    "vulnerableTransitive": 8,
    "maxDepth": 5,
    "averageDepth": 2.3
  }
}
```

**Flow:**
1. Validate authentication and scan ownership
2. Query transitive dependencies from database
3. Build dependency paths (which packages introduced each transitive dep)
4. Enrich with vulnerability data
5. Calculate statistics
6. Return transitive dependencies

---

### GET /api/scan/:scanId/dependency-paths
**Description:** Gets all dependency paths from root to a specific package, showing how a package was introduced.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `scanId`: string

**Query Parameters:**
- `packageName`: string (required)
- `packageVersion` (optional): string

**Response 200:**
```json
{
  "scanId": "scan_1234567890",
  "package": {
    "name": "lodash",
    "version": "4.17.20"
  },
  "paths": [
    {
      "path": [
        {
          "name": "my-project",
          "version": "1.0.0",
          "type": "root"
        },
        {
          "name": "express",
          "version": "4.18.0",
          "type": "direct"
        },
        {
          "name": "lodash",
          "version": "4.17.20",
          "type": "transitive"
        }
      ],
      "depth": 2,
      "isVulnerable": true
    },
    {
      "path": [
        {
          "name": "my-project",
          "version": "1.0.0",
          "type": "root"
        },
        {
          "name": "webpack",
          "version": "5.88.0",
          "type": "direct"
        },
        {
          "name": "lodash",
          "version": "4.17.20",
          "type": "transitive"
        }
      ],
      "depth": 2,
      "isVulnerable": true
    }
  ],
  "totalPaths": 2
}
```

**Flow:**
1. Validate authentication and scan ownership
2. Find all paths from root to target package using graph traversal
3. Build path structures with package details
4. Check vulnerability status for each path
5. Return all paths

---

## Package Management

### GET /api/packages/:packageName
**Description:** Retrieves detailed information about a specific npm package, including metadata, versions, and security information.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `packageName`: string

**Query Parameters:**
- `version` (optional): string (specific version to query)

**Response 200:**
```json
{
  "name": "express",
  "description": "Fast, unopinionated, minimalist web framework",
  "version": "4.18.0",
  "latestVersion": "4.18.2",
  "repository": {
    "url": "https://github.com/expressjs/express",
    "type": "git"
  },
  "homepage": "https://expressjs.com",
  "license": "MIT",
  "maintainers": [
    {
      "name": "dougwilson",
      "email": "doug@somethingdoug.com"
    }
  ],
  "keywords": ["web", "framework", "http", "api"],
  "downloads": {
    "lastMonth": 45000000,
    "total": 1200000000
  },
  "security": {
    "status": "vulnerable",
    "vulnerabilities": [
      {
        "cve": "CVE-2024-1234",
        "severity": "critical",
        "affectedVersions": ["<4.18.2"],
        "fixedIn": "4.18.2"
      }
    ],
    "lastAudited": "2024-01-20T10:00:00Z"
  },
  "metadata": {
    "createdAt": "2010-12-29T19:38:25.450Z",
    "updatedAt": "2024-01-15T12:30:00.000Z",
    "publisher": {
      "username": "dougwilson",
      "email": "doug@somethingdoug.com"
    }
  },
  "dependencies": {
    "count": 30,
    "list": [
      {
        "name": "accepts",
        "version": "~1.3.8"
      }
    ]
  }
}
```

**Flow:**
1. Validate authentication
2. Query npm registry API for package metadata
3. Check internal CVE database for vulnerabilities
4. Fetch download statistics
5. Return comprehensive package information

**Test Data Example:**
- packageName: `express`
- version: `4.18.0`

---

### GET /api/packages/search
**Description:** Searches for packages in the npm registry.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `q`: string (required, search query)
- `limit` (optional): number (default: 20)
- `offset` (optional): number (default: 0)

**Response 200:**
```json
{
  "packages": [
    {
      "name": "express",
      "description": "Fast, unopinionated, minimalist web framework",
      "version": "4.18.2",
      "downloads": 45000000,
      "score": {
        "quality": 0.98,
        "popularity": 0.99,
        "maintenance": 0.95
      }
    },
    {
      "name": "express-validator",
      "description": "Express middleware for validator.js",
      "version": "7.0.1",
      "downloads": 5000000,
      "score": {
        "quality": 0.92,
        "popularity": 0.85,
        "maintenance": 0.88
      }
    }
  ],
  "total": 150,
  "limit": 20,
  "offset": 0
}
```

**Flow:**
1. Validate authentication
2. Query npm registry search API
3. Enrich with security scores from internal database
4. Return paginated results

---

### GET /api/packages/:packageName/versions
**Description:** Gets all available versions of a package with security status.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `packageName`: string

**Response 200:**
```json
{
  "packageName": "express",
  "versions": [
    {
      "version": "4.18.2",
      "publishedAt": "2024-01-15T12:30:00.000Z",
      "security": {
        "status": "safe",
        "vulnerabilities": []
      },
      "isLatest": true
    },
    {
      "version": "4.18.1",
      "publishedAt": "2024-01-10T10:20:00.000Z",
      "security": {
        "status": "vulnerable",
        "vulnerabilities": [
          {
            "cve": "CVE-2024-1234",
            "severity": "critical"
          }
        ]
      },
      "isLatest": false
    },
    {
      "version": "4.18.0",
      "publishedAt": "2024-01-05T08:15:00.000Z",
      "security": {
        "status": "vulnerable",
        "vulnerabilities": [
          {
            "cve": "CVE-2024-1234",
            "severity": "critical"
          }
        ]
      },
      "isLatest": false
    }
  ],
  "total": 150
}
```

---

### GET /api/packages/:packageName/:version/dependencies
**Description:** Gets all dependencies (direct and transitive) for a specific package version.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `packageName`: string
- `version`: string

**Query Parameters:**
- `includeTransitive` (optional): boolean (default: true)
- `depth` (optional): number (max depth to traverse)

**Response 200:**
```json
{
  "package": {
    "name": "express",
    "version": "4.18.0"
  },
  "directDependencies": [
    {
      "name": "accepts",
      "version": "~1.3.8",
      "resolvedVersion": "1.3.8",
      "status": "safe"
    }
  ],
  "transitiveDependencies": [
    {
      "name": "mime-types",
      "version": "2.1.35",
      "introducedBy": "accepts@1.3.8",
      "status": "safe"
    }
  ],
  "statistics": {
    "totalDependencies": 30,
    "directCount": 5,
    "transitiveCount": 25,
    "vulnerableCount": 2
  }
}
```

---

## Vulnerability Results

### GET /api/vulnerabilities
**Description:** Gets all vulnerabilities with filtering and pagination.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `severity` (optional): `"critical" | "high" | "medium" | "low"`
- `status` (optional): `"open" | "fixed" | "ignored"`
- `packageName` (optional): string
- `limit` (optional): number (default: 50)
- `offset` (optional): number (default: 0)
- `sortBy` (optional): `"severity" | "date" | "package"` (default: "severity")
- `order` (optional): `"asc" | "desc"` (default: "desc")

**Response 200:**
```json
{
  "vulnerabilities": [
    {
      "id": 1,
      "package": {
        "id": 3,
        "name": "lodash",
        "version": "4.17.20"
      },
      "cve": "CVE-2024-1234",
      "severity": "critical",
      "title": "Remote Code Execution",
      "description": "Lodash versions before 4.17.21 are vulnerable to RCE",
      "cvssScore": 9.8,
      "fixedIn": "4.17.21",
      "status": "open",
      "detectedAt": "2024-01-20T15:05:00Z",
      "publishedAt": "2024-01-10T00:00:00Z"
    },
    {
      "id": 2,
      "package": {
        "id": 5,
        "name": "moment",
        "version": "2.29.1"
      },
      "cve": "CVE-2024-5678",
      "severity": "high",
      "title": "Privilege Escalation",
      "description": "Moment.js vulnerable to privilege escalation",
      "cvssScore": 7.5,
      "fixedIn": "2.30.0",
      "status": "open",
      "detectedAt": "2024-01-20T15:05:00Z",
      "publishedAt": "2024-01-12T00:00:00Z"
    }
  ],
  "total": 25,
  "limit": 50,
  "offset": 0
}
```

---

### GET /api/vulnerabilities/:vulnerabilityId
**Description:** Gets detailed information about a specific vulnerability.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `vulnerabilityId`: number

**Response 200:**
```json
{
  "id": 1,
  "package": {
    "id": 3,
    "name": "lodash",
    "version": "4.17.20",
    "latestVersion": "4.17.21"
  },
  "cve": "CVE-2024-1234",
  "severity": "critical",
  "title": "Remote Code Execution",
  "description": "Lodash versions before 4.17.21 are vulnerable to RCE through prototype pollution.",
  "cvssScore": 9.8,
  "cvssVector": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H",
  "fixedIn": "4.17.21",
  "status": "open",
  "detectedAt": "2024-01-20T15:05:00Z",
  "publishedAt": "2024-01-10T00:00:00Z",
  "references": [
    "https://nvd.nist.gov/vuln/detail/CVE-2024-1234",
    "https://github.com/lodash/lodash/issues/1234"
  ],
  "remediation": {
    "action": "update",
    "targetVersion": "4.17.21",
    "command": "npm install lodash@4.17.21"
  },
  "affectedProjects": [
    {
      "projectId": 1,
      "projectName": "my-project"
    }
  ]
}
```

---

### POST /api/vulnerabilities/:vulnerabilityId/fix
**Description:** Applies a fix for a vulnerability (updates package version).

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `vulnerabilityId`: number

**Request Body:**
```json
{
  "action": "update",
  "targetVersion": "4.17.21",
  "projectId": 1
}
```

**Response 200:**
```json
{
  "vulnerabilityId": 1,
  "status": "fixing",
  "message": "Fix applied successfully",
  "updatedPackage": {
    "name": "lodash",
    "oldVersion": "4.17.20",
    "newVersion": "4.17.21"
  }
}
```

**Response 400:** Invalid fix action
```json
{
  "error": "Invalid fix action or target version"
}
```

---

### POST /api/vulnerabilities/:vulnerabilityId/ignore
**Description:** Marks a vulnerability as ignored (won't show in reports).

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `vulnerabilityId`: number

**Request Body:**
```json
{
  "reason": "False positive",
  "expiresAt": "2024-02-20T00:00:00Z"
}
```

**Response 200:**
```json
{
  "vulnerabilityId": 1,
  "status": "ignored",
  "message": "Vulnerability ignored successfully"
}
```

---

### GET /api/vulnerabilities/:vulnerabilityId/cve-details
**Description:** Gets detailed CVE information from external sources.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `vulnerabilityId`: number

**Response 200:**
```json
{
  "cve": "CVE-2024-1234",
  "title": "Remote Code Execution in Lodash",
  "description": "Detailed CVE description...",
  "cvssScore": 9.8,
  "cvssVector": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H",
  "publishedAt": "2024-01-10T00:00:00Z",
  "modifiedAt": "2024-01-15T00:00:00Z",
  "references": [
    {
      "url": "https://nvd.nist.gov/vuln/detail/CVE-2024-1234",
      "source": "NVD"
    },
    {
      "url": "https://github.com/lodash/lodash/issues/1234",
      "source": "GitHub"
    }
  ],
  "affectedVersions": [
    "<4.17.21"
  ],
  "fixedVersions": [
    ">=4.17.21"
  ]
}
```

---

## AI Insights

### GET /api/insights/ml-metrics
**Description:** Retrieves machine learning model performance metrics.

**Headers:**
```
Authorization: Bearer <token>
```

**Response 200:**
```json
{
  "metrics": [
    {
      "label": "Model Accuracy",
      "value": 94.2,
      "trend": 2.3,
      "unit": "percentage"
    },
    {
      "label": "Anomaly Detection Rate",
      "value": 87.5,
      "trend": 5.1,
      "unit": "percentage"
    },
    {
      "label": "Threat Prediction Precision",
      "value": 89.3,
      "trend": 1.8,
      "unit": "percentage"
    },
    {
      "label": "False Positive Reduction",
      "value": 92.1,
      "trend": -0.5,
      "unit": "percentage"
    }
  ],
  "lastUpdated": "2024-01-20T15:00:00Z"
}
```

---

### GET /api/insights/anomalies
**Description:** Retrieves detected anomalies in dependency patterns.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `severity` (optional): `"critical" | "high" | "medium" | "low"`
- `limit` (optional): number (default: 20)
- `offset` (optional): number (default: 0)

**Response 200:**
```json
{
  "anomalies": [
    {
      "id": 1,
      "type": "Behavioral Pattern",
      "severity": "critical",
      "confidence": 94.2,
      "description": "Unusual dependency update frequency detected in express package",
      "detectedAt": "2024-01-20T13:00:00Z",
      "package": {
        "name": "express",
        "version": "4.18.0"
      },
      "details": {
        "expectedFrequency": "monthly",
        "actualFrequency": "daily",
        "deviation": "high"
      }
    },
    {
      "id": 2,
      "type": "Supply Chain Anomaly",
      "severity": "high",
      "confidence": 87.5,
      "description": "Package axios shows atypical version release pattern",
      "detectedAt": "2024-01-20T11:00:00Z",
      "package": {
        "name": "axios",
        "version": "1.4.0"
      }
    }
  ],
  "total": 28,
  "limit": 20,
  "offset": 0
}
```

---

### GET /api/insights/threat-predictions
**Description:** Retrieves AI-generated threat predictions and probabilities.

**Headers:**
```
Authorization: Bearer <token>
```

**Response 200:**
```json
{
  "threats": [
    {
      "id": 1,
      "vector": "Supply Chain Injection",
      "probability": 34,
      "impact": "critical",
      "mitigationSteps": [
        "Implement SBOM verification",
        "Enable signature verification",
        "Monitor package changes in real-time"
      ],
      "affectedPackages": [
        {
          "name": "express",
          "riskLevel": "high"
        }
      ],
      "predictedDate": "2024-02-15T00:00:00Z"
    },
    {
      "id": 2,
      "vector": "Dependency Confusion",
      "probability": 22,
      "impact": "high",
      "mitigationSteps": [
        "Use exact version pinning",
        "Audit private package names",
        "Configure package repository whitelist"
      ]
    }
  ],
  "generatedAt": "2024-01-20T15:00:00Z"
}
```

---

### GET /api/insights/feature-importance
**Description:** Retrieves feature importance analysis from ML models.

**Headers:**
```
Authorization: Bearer <token>
```

**Response 200:**
```json
{
  "features": [
    {
      "name": "Version Age",
      "importance": 92,
      "description": "How old the package version is"
    },
    {
      "name": "Maintenance Frequency",
      "importance": 87,
      "description": "How often the package is updated"
    },
    {
      "name": "Contributor Activity",
      "importance": 81,
      "description": "Number of active contributors"
    },
    {
      "name": "Issue Resolution Time",
      "importance": 76,
      "description": "Average time to resolve issues"
    },
    {
      "name": "Security Update Response",
      "importance": 74,
      "description": "Time to patch security vulnerabilities"
    },
    {
      "name": "Community Size",
      "importance": 68,
      "description": "Number of users and downloads"
    }
  ],
  "modelVersion": "v2.1.0",
  "lastTrained": "2024-01-15T00:00:00Z"
}
```

---

### GET /api/insights/model-training-status
**Description:** Gets the status of ML model training progress.

**Headers:**
```
Authorization: Bearer <token>
```

**Response 200:**
```json
{
  "phases": [
    {
      "phase": "Data Collection & Preprocessing",
      "progress": 100,
      "status": "Complete",
      "completedAt": "2024-01-10T00:00:00Z"
    },
    {
      "phase": "Feature Engineering",
      "progress": 100,
      "status": "Complete",
      "completedAt": "2024-01-12T00:00:00Z"
    },
    {
      "phase": "Model Training",
      "progress": 100,
      "status": "Complete",
      "completedAt": "2024-01-14T00:00:00Z"
    },
    {
      "phase": "Validation & Testing",
      "progress": 85,
      "status": "In Progress",
      "startedAt": "2024-01-15T00:00:00Z"
    },
    {
      "phase": "Hyperparameter Tuning",
      "progress": 60,
      "status": "In Progress",
      "startedAt": "2024-01-16T00:00:00Z"
    },
    {
      "phase": "Production Deployment",
      "progress": 0,
      "status": "Pending"
    }
  ],
  "overallProgress": 74
}
```

---

### GET /api/insights/summary
**Description:** Gets summary statistics for AI insights dashboard.

**Headers:**
```
Authorization: Bearer <token>
```

**Response 200:**
```json
{
  "activeLearningModels": 12,
  "anomaliesDetected24h": 28,
  "threatIntelligenceScore": 7.8,
  "modelAccuracy": 94.2,
  "lastUpdated": "2024-01-20T15:00:00Z"
}
```

---

## Suspicious Package Classification

### POST /api/packages/classify
**Description:** Classifies a package as suspicious or safe using machine learning models. Analyzes package metadata, behavior patterns, and threat indicators.

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "packageName": "suspicious-package",
  "version": "1.0.0",
  "features": {
    "analyzeMetadata": true,
    "analyzeBehavior": true,
    "checkReputation": true
  }
}
```

**Response 200:**
```json
{
  "package": {
    "name": "suspicious-package",
    "version": "1.0.0"
  },
  "classification": {
    "status": "suspicious",
    "confidence": 87.5,
    "riskScore": 8.2,
    "reasons": [
      {
        "type": "behavioral",
        "severity": "high",
        "description": "Unusual update frequency detected",
        "confidence": 92.3
      },
      {
        "type": "reputation",
        "severity": "medium",
        "description": "New maintainer with limited history",
        "confidence": 78.5
      },
      {
        "type": "metadata",
        "severity": "low",
        "description": "Missing repository information",
        "confidence": 65.2
      }
    ]
  },
  "mlModel": {
    "version": "v2.1.0",
    "features": [
      {
        "name": "Version Age",
        "importance": 0.92,
        "value": 0.15
      },
      {
        "name": "Maintenance Frequency",
        "importance": 0.87,
        "value": 0.08
      },
      {
        "name": "Contributor Activity",
        "importance": 0.81,
        "value": 0.12
      }
    ]
  },
  "recommendations": [
    "Review package source code before use",
    "Check package maintainer reputation",
    "Consider alternative packages with better track record"
  ],
  "classifiedAt": "2024-01-20T16:00:00Z"
}
```

**Flow:**
1. Validate authentication
2. Fetch package metadata from npm registry
3. Extract features (version age, maintenance frequency, etc.)
4. Run ML classification model
5. Check reputation databases
6. Analyze behavioral patterns
7. Aggregate results and calculate risk score
8. Return classification with confidence scores

**Classification Statuses:** `"safe" | "suspicious" | "malicious" | "unknown"`

**Test Data Example:**
- packageName: `suspicious-package`
- version: `1.0.0`

---

### GET /api/packages/:packageName/classification
**Description:** Retrieves the classification status and history for a package.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `packageName`: string

**Query Parameters:**
- `version` (optional): string

**Response 200:**
```json
{
  "package": {
    "name": "suspicious-package",
    "version": "1.0.0"
  },
  "currentClassification": {
    "status": "suspicious",
    "confidence": 87.5,
    "riskScore": 8.2,
    "classifiedAt": "2024-01-20T16:00:00Z",
    "modelVersion": "v2.1.0"
  },
  "classificationHistory": [
    {
      "status": "suspicious",
      "confidence": 87.5,
      "classifiedAt": "2024-01-20T16:00:00Z",
      "modelVersion": "v2.1.0"
    },
    {
      "status": "safe",
      "confidence": 65.2,
      "classifiedAt": "2024-01-15T10:00:00Z",
      "modelVersion": "v2.0.5"
    }
  ],
  "flags": [
    {
      "type": "typosquatting",
      "severity": "high",
      "description": "Package name similar to popular package 'express'"
    }
  ]
}
```

---

### GET /api/scan/:scanId/suspicious-packages
**Description:** Gets all packages from a scan that have been classified as suspicious.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `scanId`: string

**Query Parameters:**
- `minConfidence` (optional): number (default: 70, 0-100)
- `limit` (optional): number (default: 50)

**Response 200:**
```json
{
  "scanId": "scan_1234567890",
  "suspiciousPackages": [
    {
      "id": 15,
      "name": "suspicious-package",
      "version": "1.0.0",
      "classification": {
        "status": "suspicious",
        "confidence": 87.5,
        "riskScore": 8.2
      },
      "reasons": [
        "Unusual update frequency",
        "New maintainer with limited history"
      ],
      "detectedAt": "2024-01-20T15:05:00Z"
    }
  ],
  "total": 3,
  "statistics": {
    "suspiciousCount": 3,
    "maliciousCount": 0,
    "averageRiskScore": 7.8
  }
}
```

---

### POST /api/packages/:packageName/report-suspicious
**Description:** Allows users to report a package as suspicious for manual review.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `packageName`: string

**Request Body:**
```json
{
  "version": "1.0.0",
  "reason": "Package contains unexpected behavior",
  "description": "Detailed description of suspicious activity",
  "evidence": [
    "Package modifies system files",
    "Unusual network activity detected"
  ]
}
```

**Response 200:**
```json
{
  "reportId": "report_7890123456",
  "package": {
    "name": "suspicious-package",
    "version": "1.0.0"
  },
  "status": "submitted",
  "message": "Report submitted successfully. Our security team will review it.",
  "submittedAt": "2024-01-20T16:30:00Z"
}
```

**Flow:**
1. Validate authentication
2. Validate request body
3. Create suspicious package report
4. Queue for security team review
5. Return report ID

---

## SBOM Generation

### POST /api/sbom/generate
**Description:** Generates a Software Bill of Materials (SBOM) for a scan in various formats (SPDX, CycloneDX, etc.).

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "scanId": "scan_1234567890",
  "format": "spdx",
  "includeTransitive": true,
  "includeVulnerabilities": true,
  "includeLicenses": true
}
```

**Formats:** `"spdx" | "cyclonedx" | "swid" | "json"`

**Response 200:**
```json
{
  "sbomId": "sbom_1234567890",
  "scanId": "scan_1234567890",
  "format": "spdx",
  "status": "generating",
  "estimatedCompletion": "2024-01-20T17:00:00Z",
  "message": "SBOM generation started"
}
```

**Flow:**
1. Validate authentication and scan ownership
2. Validate format and options
3. Queue SBOM generation job
4. Return SBOM ID for polling

---

### GET /api/sbom/:sbomId
**Description:** Retrieves a generated SBOM file.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `sbomId`: string

**Query Parameters:**
- `format` (optional): `"spdx" | "cyclonedx" | "json"` (default: original format)

**Response 200 (SPDX format):**
Returns SPDX document with `Content-Type: text/spdx`

**Response 200 (CycloneDX format):**
Returns CycloneDX JSON with `Content-Type: application/json`

**Response 200 (JSON format):**
```json
{
  "sbomId": "sbom_1234567890",
  "scanId": "scan_1234567890",
  "format": "spdx",
  "version": "SPDX-2.3",
  "createdAt": "2024-01-20T17:00:00Z",
  "packages": [
    {
      "name": "express",
      "version": "4.18.0",
      "type": "direct",
      "license": "MIT",
      "publisher": "dougwilson",
      "vulnerabilities": [
        {
          "cve": "CVE-2024-1234",
          "severity": "critical"
        }
      ]
    }
  ],
  "statistics": {
    "totalPackages": 156,
    "directPackages": 45,
    "transitivePackages": 111,
    "vulnerablePackages": 12
  }
}
```

**Flow:**
1. Validate authentication and SBOM ownership
2. Check SBOM generation status
3. If complete, fetch SBOM data
4. Convert to requested format if needed
5. Return SBOM file or data

---

### GET /api/sbom
**Description:** Lists all generated SBOMs for the user.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `scanId` (optional): string (filter by scan)
- `format` (optional): `"spdx" | "cyclonedx" | "swid" | "json"`
- `limit` (optional): number (default: 20)
- `offset` (optional): number (default: 0)

**Response 200:**
```json
{
  "sboms": [
    {
      "sbomId": "sbom_1234567890",
      "scanId": "scan_1234567890",
      "format": "spdx",
      "status": "complete",
      "createdAt": "2024-01-20T17:00:00Z",
      "downloadUrl": "/api/sbom/sbom_1234567890"
    }
  ],
  "total": 10,
  "limit": 20,
  "offset": 0
}
```

---

## Real-time Updates

### WebSocket Connection
**Description:** Establishes a WebSocket connection for real-time updates on scan progress, vulnerability detection, and other events.

**Connection URL:**
```
wss://api.chainguard.com/ws?token=<jwt_token>
```

**Authentication:**
- Token passed as query parameter or in initial handshake
- Token validated on connection

**Message Types:**

#### 1. Scan Progress Update
```json
{
  "type": "scan.progress",
  "scanId": "scan_1234567890",
  "data": {
    "progress": 65,
    "packagesScanned": 130,
    "totalPackages": 200,
    "currentPackage": "express@4.18.0",
    "status": "scanning"
  },
  "timestamp": "2024-01-20T15:02:30Z"
}
```

#### 2. Package Scanned
```json
{
  "type": "scan.package",
  "scanId": "scan_1234567890",
  "data": {
    "package": {
      "id": 2,
      "name": "express",
      "version": "4.18.0",
      "status": "vulnerable"
    },
    "vulnerabilities": [
      {
        "cve": "CVE-2024-1234",
        "severity": "critical"
      }
    ]
  },
  "timestamp": "2024-01-20T15:02:35Z"
}
```

#### 3. Scan Complete
```json
{
  "type": "scan.complete",
  "scanId": "scan_1234567890",
  "data": {
    "status": "complete",
    "totalPackages": 200,
    "vulnerablePackages": 12,
    "completedAt": "2024-01-20T15:05:00Z"
  },
  "timestamp": "2024-01-20T15:05:00Z"
}
```

#### 4. Vulnerability Detected
```json
{
  "type": "vulnerability.detected",
  "scanId": "scan_1234567890",
  "data": {
    "vulnerability": {
      "id": 1,
      "package": {
        "name": "lodash",
        "version": "4.17.20"
      },
      "cve": "CVE-2024-1234",
      "severity": "critical"
    }
  },
  "timestamp": "2024-01-20T15:03:00Z"
}
```

#### 5. Anomaly Detected
```json
{
  "type": "anomaly.detected",
  "data": {
    "anomaly": {
      "id": 1,
      "type": "Behavioral Pattern",
      "severity": "critical",
      "confidence": 94.2,
      "package": {
        "name": "express",
        "version": "4.18.0"
      }
    }
  },
  "timestamp": "2024-01-20T15:10:00Z"
}
```

**Client Subscription:**
```json
{
  "action": "subscribe",
  "channels": ["scan.scan_1234567890", "vulnerabilities", "anomalies"]
}
```

**Client Unsubscribe:**
```json
{
  "action": "unsubscribe",
  "channels": ["scan.scan_1234567890"]
}
```

**Flow:**
1. Client establishes WebSocket connection with JWT token
2. Server validates token
3. Client subscribes to channels (scan IDs, vulnerability updates, etc.)
4. Server sends real-time updates for subscribed channels
5. Client can unsubscribe from channels
6. Connection closes on logout or timeout

**Security:**
- Token required for connection
- Users can only subscribe to their own scans/projects
- Rate limiting on message frequency
- Connection timeout after 1 hour of inactivity

---

### GET /api/scan/:scanId/stream
**Description:** Server-Sent Events (SSE) alternative to WebSocket for scan progress updates.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `scanId`: string

**Response:**
Stream of Server-Sent Events with `Content-Type: text/event-stream`

**Event Format:**
```
event: progress
data: {"progress": 65, "packagesScanned": 130, "totalPackages": 200}

event: package
data: {"package": {"name": "express", "version": "4.18.0", "status": "vulnerable"}}

event: complete
data: {"status": "complete", "totalPackages": 200}
```

**Flow:**
1. Validate authentication and scan ownership
2. Open SSE connection
3. Stream progress updates every 2-3 seconds
4. Close connection when scan completes or client disconnects

---

## Reports

### POST /api/reports/generate
**Description:** Generates a security report based on specified parameters.

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "reportType": "executive",
  "dateRange": {
    "start": "2024-01-01T00:00:00Z",
    "end": "2024-01-20T00:00:00Z"
  },
  "includeCharts": true,
  "includeRecommendations": true,
  "format": "json"
}
```

**Report Types:** `"executive" | "detailed" | "compliance" | "trends"`

**Date Range Options:**
- `"last7"` - Last 7 days
- `"last30"` - Last 30 days
- `"last90"` - Last 90 days
- `"custom"` - Custom date range

**Response 200:**
```json
{
  "reportId": "report_1234567890",
  "status": "generating",
  "estimatedCompletion": "2024-01-20T15:10:00Z",
  "message": "Report generation started"
}
```

---

### GET /api/reports/:reportId
**Description:** Retrieves a generated report.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `reportId`: string

**Query Parameters:**
- `format` (optional): `"json" | "pdf" | "csv"` (default: "json")

**Response 200 (JSON format):**
```json
{
  "reportId": "report_1234567890",
  "reportType": "executive",
  "generatedAt": "2024-01-20T15:10:00Z",
  "dateRange": {
    "start": "2024-01-01T00:00:00Z",
    "end": "2024-01-20T00:00:00Z"
  },
  "summary": {
    "totalVulnerabilities": 366,
    "issuesResolved": 353,
    "avgResolutionTime": "2.9 days",
    "securityScore": 7.8
  },
  "severityDistribution": [
    {
      "severity": "Critical",
      "count": 24,
      "percentage": 15
    },
    {
      "severity": "High",
      "count": 156,
      "percentage": 42
    },
    {
      "severity": "Medium",
      "count": 128,
      "percentage": 28
    },
    {
      "severity": "Low",
      "count": 58,
      "percentage": 15
    }
  ],
  "trends": [
    {
      "period": "Week 1",
      "vulnerabilities": 42,
      "resolved": 28,
      "newIssues": 12
    },
    {
      "period": "Week 2",
      "vulnerabilities": 45,
      "resolved": 32,
      "newIssues": 15
    }
  ],
  "complianceMetrics": [
    {
      "standard": "OWASP Top 10",
      "score": 78,
      "status": "good"
    },
    {
      "standard": "CWE/SANS",
      "score": 82,
      "status": "good"
    }
  ],
  "recommendations": [
    {
      "id": 1,
      "priority": "critical",
      "title": "Update critical dependencies immediately",
      "description": "24 critical vulnerabilities require immediate attention",
      "impact": "Prevents remote code execution",
      "effort": "High"
    }
  ],
  "monthlyMetrics": [
    {
      "month": "January",
      "issues": 156,
      "resolved": 98,
      "avgTime": "4.2 days"
    }
  ]
}
```

**Response 200 (PDF format):** Returns PDF file with `Content-Type: application/pdf`

**Response 200 (CSV format):** Returns CSV file with `Content-Type: text/csv`

---

### GET /api/reports
**Description:** Lists all generated reports for the user.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `limit` (optional): number (default: 20)
- `offset` (optional): number (default: 0)
- `reportType` (optional): `"executive" | "detailed" | "compliance" | "trends"`

**Response 200:**
```json
{
  "reports": [
    {
      "reportId": "report_1234567890",
      "reportType": "executive",
      "generatedAt": "2024-01-20T15:10:00Z",
      "dateRange": {
        "start": "2024-01-01T00:00:00Z",
        "end": "2024-01-20T00:00:00Z"
      },
      "status": "complete",
      "downloadUrl": "/api/reports/report_1234567890/download"
    }
  ],
  "total": 15,
  "limit": 20,
  "offset": 0
}
```

---

## Export & Download

### GET /api/export/scan/:scanId
**Description:** Exports scan results in various formats.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `scanId`: string

**Query Parameters:**
- `format`: `"json" | "csv" | "pdf"` (required)

**Response 200:**
- JSON: Returns JSON with `Content-Type: application/json`
- CSV: Returns CSV file with `Content-Type: text/csv`
- PDF: Returns PDF file with `Content-Type: application/pdf`

**Response 404:** Scan not found
```json
{
  "error": "Scan not found"
}
```

---

### GET /api/export/vulnerabilities
**Description:** Exports all vulnerabilities in various formats.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `format`: `"json" | "csv" | "pdf"` (required)
- `severity` (optional): `"critical" | "high" | "medium" | "low"`
- `status` (optional): `"open" | "fixed" | "ignored"`

**Response 200:**
- Returns file in requested format

---

### GET /api/reports/:reportId/download
**Description:** Downloads a generated report file.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `reportId`: string

**Query Parameters:**
- `format` (optional): `"pdf" | "json" | "csv"` (default: "pdf")

**Response 200:**
- Returns file download with appropriate `Content-Type` and `Content-Disposition: attachment`

---

## Error Responses

All endpoints may return the following error responses:

### 400 Bad Request
```json
{
  "error": "Invalid request parameters",
  "details": {
    "field": "email",
    "message": "Email is required"
  }
}
```

### 401 Unauthorized
```json
{
  "error": "Unauthorized",
  "message": "Invalid or expired token"
}
```

### 403 Forbidden
```json
{
  "error": "Forbidden",
  "message": "You don't have permission to access this resource"
}
```

### 404 Not Found
```json
{
  "error": "Not Found",
  "message": "Resource not found"
}
```

### 500 Internal Server Error
```json
{
  "error": "Internal Server Error",
  "message": "An unexpected error occurred"
}
```

---

## API Flow Examples

### Complete Scan Flow

1. **Upload Project Files**
   ```
   POST /api/scan/upload
   → Returns: scanId
   ```

2. **Start Scanning**
   ```
   POST /api/scan/start
   Body: { scanId: "scan_1234567890" }
   → Returns: { status: "scanning" }
   ```

3. **Poll Scan Status** (while scanning)
   ```
   GET /api/scan/scan_1234567890/status
   → Returns: { progress: 65, status: "scanning" }
   ```

4. **Get Scan Results** (when complete)
   ```
   GET /api/scan/scan_1234567890/results
   → Returns: Complete scan results with vulnerabilities
   ```

5. **View Detailed Vulnerability**
   ```
   GET /api/vulnerabilities/1
   → Returns: Detailed CVE information
   ```

6. **Apply Fix**
   ```
   POST /api/vulnerabilities/1/fix
   → Returns: Fix applied confirmation
   ```

7. **Export Results**
   ```
   GET /api/export/scan/scan_1234567890?format=pdf
   → Returns: PDF file download
   ```

---

### Dashboard Flow

1. **Get Dashboard Stats**
   ```
   GET /api/dashboard/stats
   → Returns: Overview statistics
   ```

2. **Get Activity Feed**
   ```
   GET /api/dashboard/activity?limit=20
   → Returns: Recent activities
   ```

3. **Get Dependency Health**
   ```
   GET /api/dashboard/dependency-health
   → Returns: Health scores for dependencies
   ```

---

### Report Generation Flow

1. **Generate Report**
   ```
   POST /api/reports/generate
   Body: { reportType: "executive", dateRange: "last30" }
   → Returns: { reportId: "report_1234567890" }
   ```

2. **Get Report** (when ready)
   ```
   GET /api/reports/report_1234567890?format=pdf
   → Returns: PDF report file
   ```

---

## Authentication Flow

1. **Login**
   ```
   POST /api/auth/login
   Body: { email: "user@example.com", password: "password123" }
   → Returns: { token: "jwt.token.here", user: {...} }
   ```

2. **Store Token** (in localStorage or httpOnly cookie)

3. **Use Token** (in all subsequent requests)
   ```
   Authorization: Bearer <token>
   ```

4. **Get Current User**
   ```
   GET /api/auth/me
   Headers: { Authorization: "Bearer <token>" }
   → Returns: User profile
   ```

5. **Logout**
   ```
   POST /api/auth/logout
   Headers: { Authorization: "Bearer <token>" }
   → Invalidates token
   ```

---

## Test Data Examples

### Login Credentials
- Email: `test@test.com`
- Password: `123456`

### Sample Scan ID
- `scan_1234567890`

### Sample Report ID
- `report_1234567890`

### Sample Vulnerability ID
- `1`

### Sample Package Data
```json
{
  "name": "express",
  "version": "4.18.0",
  "status": "vulnerable"
}
```

---

## CVE Database & Threat Intelligence

### POST /api/cve/sync
**Description:** Triggers a manual sync with external CVE databases (NVD, GitHub Security Advisories, etc.). Typically run as a scheduled job.

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "sources": ["nvd", "github", "npm"],
  "forceFullSync": false,
  "dateRange": {
    "start": "2024-01-01T00:00:00Z",
    "end": "2024-01-20T00:00:00Z"
  }
}
```

**Response 200:**
```json
{
  "syncId": "sync_1234567890",
  "status": "started",
  "sources": ["nvd", "github", "npm"],
  "estimatedCompletion": "2024-01-20T18:00:00Z",
  "message": "CVE sync started"
}
```

**Flow:**
1. Validate authentication (admin only)
2. Queue CVE sync job
3. Fetch CVEs from external sources
4. Parse and normalize CVE data
5. Update internal database
6. Return sync status

**Security:** Admin-only endpoint

---

### GET /api/cve/sync/status
**Description:** Gets the status of the latest CVE database sync.

**Headers:**
```
Authorization: Bearer <token>
```

**Response 200:**
```json
{
  "syncId": "sync_1234567890",
  "status": "complete",
  "sources": ["nvd", "github", "npm"],
  "statistics": {
    "totalCVEs": 150000,
    "newCVEs": 1250,
    "updatedCVEs": 340,
    "deletedCVEs": 5
  },
  "startedAt": "2024-01-20T17:00:00Z",
  "completedAt": "2024-01-20T18:00:00Z",
  "duration": "1h 0m"
}
```

---

### GET /api/cve/:cveId
**Description:** Retrieves detailed information about a specific CVE from the database.

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `cveId`: string (e.g., "CVE-2024-1234")

**Response 200:**
```json
{
  "cve": "CVE-2024-1234",
  "title": "Remote Code Execution in Lodash",
  "description": "Lodash versions before 4.17.21 are vulnerable to RCE through prototype pollution.",
  "severity": "critical",
  "cvssScore": 9.8,
  "cvssVector": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H",
  "publishedAt": "2024-01-10T00:00:00Z",
  "modifiedAt": "2024-01-15T00:00:00Z",
  "affectedPackages": [
    {
      "name": "lodash",
      "affectedVersions": ["<4.17.21"],
      "fixedVersions": [">=4.17.21"]
    }
  ],
  "references": [
    {
      "url": "https://nvd.nist.gov/vuln/detail/CVE-2024-1234",
      "source": "NVD"
    },
    {
      "url": "https://github.com/lodash/lodash/issues/1234",
      "source": "GitHub"
    }
  ],
  "exploitability": {
    "status": "public",
    "description": "Public exploit available"
  },
  "patchAvailable": true
}
```

---

### GET /api/cve/search
**Description:** Searches for CVEs by package name, severity, or keywords.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `packageName` (optional): string
- `severity` (optional): `"critical" | "high" | "medium" | "low"`
- `q` (optional): string (keyword search)
- `limit` (optional): number (default: 50)
- `offset` (optional): number (default: 0)

**Response 200:**
```json
{
  "cves": [
    {
      "cve": "CVE-2024-1234",
      "title": "Remote Code Execution in Lodash",
      "severity": "critical",
      "cvssScore": 9.8,
      "affectedPackages": ["lodash"],
      "publishedAt": "2024-01-10T00:00:00Z"
    }
  ],
  "total": 25,
  "limit": 50,
  "offset": 0
}
```

---

## Notes for Backend Implementation

1. **Authentication**: Implement JWT-based authentication. Tokens should expire after 24 hours.

2. **File Upload**: Support multipart/form-data for file uploads. Validate file types and sizes. Maximum file size: 50MB per file.

3. **Scanning**: Implement async scanning process. Use job queues (e.g., Bull, BullMQ) for long-running scans. Support parallel package scanning for performance.

4. **CVE Database**: 
   - Integrate with NVD (National Vulnerability Database) API
   - Integrate with GitHub Security Advisories API
   - Integrate with npm security advisories
   - Schedule daily sync jobs
   - Cache CVE data for performance

5. **Dependency Tree Parsing**: 
   - Parse package.json, package-lock.json, yarn.lock, pnpm-lock.yaml
   - Build dependency graph using graph algorithms
   - Store relationships in graph database (Neo4j) or relational DB with proper indexing

6. **ML Models**: 
   - Train models for suspicious package classification
   - Use features: version age, maintenance frequency, contributor activity, etc.
   - Implement model versioning and A/B testing
   - Cache model predictions

7. **Rate Limiting**: Implement rate limiting to prevent abuse:
   - Authentication endpoints: 5 requests per minute
   - Scan endpoints: 10 requests per minute
   - Dashboard/Insights: 60 requests per minute
   - Reports: 5 requests per minute
   - Export: 10 requests per minute
   - Package classification: 20 requests per minute

8. **Caching**: Cache frequently accessed data:
   - Dashboard stats: 30 seconds
   - Dependency health: 5 minutes
   - ML metrics: 5 minutes
   - Package metadata: 1 hour
   - CVE data: 24 hours

9. **WebSockets/SSE**: 
   - Implement WebSocket server for real-time updates
   - Fallback to SSE for clients that don't support WebSockets
   - Implement connection pooling and load balancing

10. **Database Schema**: Design database tables for:
    - Users (id, email, name, role, createdAt)
    - Projects (id, userId, name, packageManager, createdAt)
    - Scans (id, projectId, status, startedAt, completedAt)
    - Packages (id, name, version, type, status)
    - PackageRelationships (sourceId, targetId, type, depth)
    - Vulnerabilities (id, packageId, cve, severity, status)
    - Reports (id, userId, type, status, generatedAt)
    - Anomalies (id, packageId, type, severity, confidence)
    - MLModelMetrics (id, metricName, value, timestamp)
    - SBOMs (id, scanId, format, status, generatedAt)
    - SuspiciousPackageReports (id, packageName, reason, status)
    - CVEs (cve, title, description, severity, cvssScore, publishedAt)

11. **Graph Database**: Consider using Neo4j or similar for dependency graph storage and traversal:
    - Nodes: Packages, Projects
    - Relationships: DEPENDS_ON, INTRODUCES, VULNERABLE_IN

12. **Error Handling**: Implement comprehensive error handling and logging:
    - Structured logging (JSON format)
    - Error tracking (Sentry, Rollbar)
    - Alerting for critical errors

13. **Validation**: Use validation libraries (e.g., Zod, Joi) to validate all request bodies and query parameters.

14. **Security**: 
    - Sanitize all user inputs
    - Implement CSRF protection
    - Use HTTPS in production
    - Implement proper CORS policies
    - Rate limiting on all endpoints
    - Input validation and sanitization
    - SQL injection prevention (use parameterized queries)
    - XSS prevention

15. **SBOM Generation**: 
    - Support SPDX 2.3 and 3.0 formats
    - Support CycloneDX 1.4 and 1.5 formats
    - Support SWID tags
    - Include license information
    - Include vulnerability data

16. **Package Classification**: 
    - Implement feature extraction pipeline
    - Train and deploy ML models
    - Monitor model performance
    - Implement feedback loop for model improvement

17. **Performance Optimization**:
    - Use database indexes on frequently queried fields
    - Implement pagination for large result sets
    - Use connection pooling
    - Implement query optimization
    - Cache expensive computations

18. **Monitoring & Observability**:
    - API response time monitoring
    - Error rate tracking
    - Database query performance
    - Scan processing time
    - ML model inference time

---

## API Versioning

Consider implementing API versioning:
- `/api/v1/...` for current version
- Future versions: `/api/v2/...`

---

## Rate Limits

Recommended rate limits per user:
- Authentication endpoints: 5 requests per minute
- Scan endpoints: 10 requests per minute
- Dashboard/Insights: 60 requests per minute
- Reports: 5 requests per minute
- Export: 10 requests per minute

---

## Response Times

Target response times:
- Authentication: < 200ms
- Dashboard stats: < 500ms
- Scan status: < 100ms
- Vulnerability details: < 300ms
- Report generation: Async (notify when complete)
- Export: < 2s for small files, async for large files

---

This documentation should provide a complete blueprint for implementing the backend APIs required by the ChainGuard frontend.

