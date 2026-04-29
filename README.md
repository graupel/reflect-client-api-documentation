---
title: Reflect Client Data API
description: API reference for accessing athlete biometric data and 3D body scan measurements.
---

# Reflect Client Data API Reference

Access athlete biometric data and 3D body scan measurements programmatically through our secure REST API.

## Table of Contents

- [Getting Started](#getting-started)
- [Authentication](#authentication)
- [Endpoints](#endpoints)
  - [List Athletes](#list-athletes)
  - [Get Athlete](#get-athlete)
  - [List Scans](#list-scans)
  - [Get Scan](#get-scan)
  - [Get Measurements](#get-measurements)
  - [Get 3D Model File](#get-3d-model-file)
  - [Export Data](#export-data)
- [Data Models](#data-models)
- [Error Handling](#error-handling)
- [Rate Limits](#rate-limits)
- [Code Examples](#code-examples)
- [Security Best Practices](#security-best-practices)
- [Support](#support)

---

## Getting Started

The Reflect Client Data API lets your organization pull athlete scan data directly into your own systems — analytics platforms, dashboards, custom applications, or anything else you build.

### Quick Start

1. **Get your credentials** — A `client_id` and `client_secret` are provisioned when you purchase a Reflect license.
2. **Authenticate** — Exchange your credentials for a short-lived access token via OAuth 2.0.
3. **Call the API** — Use the token to read athlete profiles, scan results, measurements, and 3D model files.

### Base URL

```
https://prod-wl.fitmatch.io/reflect/client/v1
```

All endpoint paths in this document are relative to this base URL.

### Authentication at a Glance

Every request must include a valid access token:

```
Authorization: Bearer <access_token>
```

See [Authentication](#authentication) for the full token flow.

---

## Authentication

### Obtain an Access Token

**`POST https://prod-wl.fitmatch.io/reflect-auth/client-credentials`**

Exchange your API credentials for a Bearer token.

**Request body**

```json
{
  "client_id": "your_client_id",
  "client_secret": "your_client_secret",
  "grant_type": "client_credentials"
}
```

**Response**

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Tokens are valid for **1 hour**. Cache and reuse them for the full lifetime — there is no need to request a new token for every call. When a token expires, request a new one with the same credentials.

---

## Endpoints

### List Athletes

Retrieve a paginated list of athletes in your organization.

**`GET /orgs/{org_id}/athletes`**

| Parameter | In | Required | Description |
|---|---|---|---|
| `org_id` | path | yes | Your organization identifier |
| `limit` | query | no | Results per page (default `50`, max `100`) |
| `offset` | query | no | Results to skip (default `0`) |
| `sort` | query | no | Sort field: `created_at`, `first_name`, `last_name`, `last_scanned_date` (default `created_at`) |
| `order` | query | no | `asc` or `desc` (default `desc`) |

**Response** `200 OK`

```json
{
  "athletes": [
    {
      "athleteId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "firstName": "John",
      "lastName": "Doe",
      "position": "Forward",
      "birthday": "2000-05-15",
      "teamId": "ABC",
      "teamName": "Example SC",
      "biometrics": {
        "height": 185.5,
        "weight": 82.3
      },
      "organization": "ABC",
      "organizationName": "Example Sports Club",
      "lastScannedDate": "2026-03-15T14:30:00Z",
      "createdAt": "2025-06-01T10:00:00Z"
    }
  ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 125,
    "has_more": true
  },
  "requested_at": "2026-04-29T12:00:00.000000Z"
}
```

---

### Get Athlete

Retrieve a single athlete profile with full biometric detail.

**`GET /orgs/{org_id}/athletes/{athlete_id}`**

| Parameter | In | Required | Description |
|---|---|---|---|
| `org_id` | path | yes | Your organization identifier |
| `athlete_id` | path | yes | Athlete UUID |

**Response** `200 OK`

```json
{
  "athlete": {
    "athleteId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "firstName": "John",
    "lastName": "Doe",
    "position": "Forward",
    "birthday": "2000-05-15",
    "teamId": "ABC",
    "teamName": "Example SC",
    "biometrics": {
      "height": 185.5,
      "weight": 82.3,
      "wingspan": 195.2
    },
    "athleteEmail": "john.doe@example.com",
    "organization": "ABC",
    "organizationName": "Example Sports Club",
    "lastScannedDate": "2026-03-15T14:30:00Z",
    "createdAt": "2025-06-01T10:00:00Z"
  }
}
```

---

### List Scans

Retrieve a paginated list of scans for your organization. Supports filtering by athlete, status, and date range.

**`GET /orgs/{org_id}/scans`**

| Parameter | In | Required | Description |
|---|---|---|---|
| `org_id` | path | yes | Your organization identifier |
| `limit` | query | no | Results per page (default `50`, max `100`) |
| `offset` | query | no | Results to skip (default `0`) |
| `athlete_id` | query | no | Filter by athlete UUID |
| `status` | query | no | Filter by status: `pending`, `processing`, `completed`, `error` |
| `start_date` | query | no | Return scans created after this date (ISO 8601) |
| `end_date` | query | no | Return scans created before this date (ISO 8601) |
| `sort` | query | no | Sort field: `created_at`, `status`, `processing_time` (default `created_at`) |
| `order` | query | no | `asc` or `desc` (default `desc`) |

**Response** `200 OK`

```json
{
  "scans": [
    {
      "scanId": "5bb5e4d3-005d-4f30-8ce2-616a4b2d1825",
      "athleteId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "teamId": "ABC",
      "scanStatus": "completed",
      "operatorName": "Jane Trainer",
      "customId": "PT-00451",
      "measurements": {
        "height": {
          "body_height": 185.5
        },
        "torso": {
          "chest_circumference": 98.5
        }
      },
      "processingTime": 12.5,
      "timestamp": "2026-03-15T14:30:00Z"
    }
  ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 342,
    "has_more": true
  },
  "filters": {
    "athlete_id": null,
    "status": null,
    "start_date": null,
    "end_date": null
  },
  "requested_at": "2026-04-29T12:00:00.000000Z"
}
```

---

### Get Scan

Retrieve complete scan data including all body measurements.

**`GET /orgs/{org_id}/scans/{scan_id}`**

| Parameter | In | Required | Description |
|---|---|---|---|
| `org_id` | path | yes | Your organization identifier |
| `scan_id` | path | yes | Scan UUID |

**Response** `200 OK`

```json
{
  "scan": {
    "scanId": "5bb5e4d3-005d-4f30-8ce2-616a4b2d1825",
    "athleteId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "teamId": "ABC",
    "scanStatus": "completed",
    "operatorName": "Jane Trainer",
    "customId": "PT-00451",
    "measurements": {
      "height": {
        "body_height": 185.5,
        "seated_height": 95.2
      },
      "torso": {
        "chest_circumference": 98.5,
        "waist_circumference": 82.3,
        "hip_circumference": 95.7
      },
      "arm": {
        "upper_arm_circumference": 32.1,
        "forearm_circumference": 27.8
      },
      "legs": {
        "upper_thigh_circumference_left": 58.2,
        "upper_thigh_circumference_right": 58.5,
        "calf_circumference_left": 38.9,
        "calf_circumference_right": 39.1
      },
      "ratios": {
        "shoulder_to_hip_ratio": 1.42,
        "waist_to_hip_ratio": 0.86
      },
      "physique_metrics": {
        "bmi": 23.9,
        "bri": 3.45
      }
    },
    "processingTime": 12.5,
    "timestamp": "2026-03-15T14:30:00Z"
  }
}
```

---

### Get Measurements

Retrieve only the measurement data for a scan. Use the optional `types` filter to request specific measurement groups.

**`GET /orgs/{org_id}/scans/{scan_id}/measurements`**

| Parameter | In | Required | Description |
|---|---|---|---|
| `org_id` | path | yes | Your organization identifier |
| `scan_id` | path | yes | Scan UUID |
| `types` | query | no | Comma-separated measurement groups to include (e.g. `height,torso,arm`) |

**Response** `200 OK`

```json
{
  "scan_id": "5bb5e4d3-005d-4f30-8ce2-616a4b2d1825",
  "measurements": {
    "height": {
      "body_height": 185.5,
      "seated_height": 95.2
    },
    "torso": {
      "chest_circumference": 98.5,
      "waist_circumference": 82.3
    }
  },
  "measurement_count": 2,
  "requested_types": "height,torso",
  "requested_at": "2026-04-29T12:00:00.000000Z"
}
```

When `types` is omitted, all available measurement groups are returned.

---

### Get 3D Model File

Generate a secure, time-limited download URL for a 3D model file.

**`GET /orgs/{org_id}/scans/{scan_id}/model/url`**

| Parameter | In | Required | Description |
|---|---|---|---|
| `org_id` | path | yes | Your organization identifier |
| `scan_id` | path | yes | Scan UUID |
| `extension` | query | no | File format: `pcd` or `usdz` (default `pcd`) |
| `expiration` | query | no | URL lifetime in seconds (default `3600`, max `86400`) |

**Response** `200 OK`

```json
{
  "download_url": "https://storage.fitmatch.io/...",
  "expires_in": 3600,
  "expires_at": "2026-04-29T13:00:00Z",
  "scan_id": "5bb5e4d3-005d-4f30-8ce2-616a4b2d1825"
}
```

The `download_url` is a pre-signed URL. Use a simple `GET` request (no auth header needed) to download the file before it expires.

---

### Export Data

Bulk export athlete or scan data in JSON or CSV format.

**`POST /orgs/{org_id}/export`**

| Parameter | In | Required | Description |
|---|---|---|---|
| `org_id` | path | yes | Your organization identifier |

**Request body**

```json
{
  "data_type": "scans",
  "format": "json",
  "filters": {
    "date_range": {
      "start": "2026-01-01",
      "end": "2026-04-29"
    }
  }
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `data_type` | string | yes | `athletes`, `scans`, or `measurements` |
| `format` | string | no | `json` (default) or `csv` |
| `filters.date_range.start` | string | no | ISO 8601 start date |
| `filters.date_range.end` | string | no | ISO 8601 end date |
| `filters.status` | string | no | Filter scans by status |

**Response** `200 OK`

For JSON format, the response body is a flat array:

```json
[
  {
    "scan_id": "5bb5e4d3-005d-4f30-8ce2-616a4b2d1825",
    "status": "completed",
    "measurements": { "..." : "..." },
    "athlete_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "first_name": "John",
    "last_name": "Doe",
    "team_abbreviation": "ABC",
    "operator_name": "Jane Trainer",
    "custom_id": "PT-00451",
    "created_at": "2026-03-15T14:30:00Z"
  }
]
```

For CSV format, the response is returned with `Content-Type: text/csv` and a `Content-Disposition` header for download.

---

## Data Models

### Athlete

| Field | Type | Description |
|---|---|---|
| `athleteId` | `string` | Unique identifier (UUID) |
| `firstName` | `string` | First name |
| `lastName` | `string` | Last name |
| `position` | `string \| null` | Playing position |
| `birthday` | `string \| null` | Date of birth (ISO 8601) |
| `teamId` | `string` | Team identifier (abbreviation) |
| `teamName` | `string` | Team display name |
| `notes` | `string \| null` | Additional notes |
| `biometrics` | `object` | Key-value biometric data (height, weight, wingspan, etc.) |
| `athleteEmail` | `string \| null` | Contact email |
| `organization` | `string \| null` | Organization abbreviation |
| `organizationName` | `string \| null` | Organization display name |
| `lastScannedDate` | `string \| null` | Most recent scan timestamp (ISO 8601) |
| `createdAt` | `string` | Profile creation timestamp (ISO 8601) |

### Scan

| Field | Type | Description |
|---|---|---|
| `scanId` | `string` | Unique identifier (UUID) |
| `athleteId` | `string` | Associated athlete UUID |
| `teamId` | `string` | Organization identifier |
| `scanStatus` | `string` | `pending`, `processing`, `completed`, or `error` |
| `operatorName` | `string \| null` | Name of the person who performed the scan |
| `customId` | `string \| null` | Your custom identifier for this scan (e.g. internal tracking number) |
| `measurements` | `object` | Body measurements grouped by region (see below) |
| `processingTime` | `number \| null` | Processing duration in seconds |
| `timestamp` | `string` | Scan capture time (ISO 8601) |

### Measurements

Measurements are grouped by anatomical region. The exact set of fields depends on the scan and processing configuration.

| Group | Example Fields |
|---|---|
| `height` | `body_height`, `seated_height` |
| `torso` | `chest_circumference`, `waist_circumference`, `hip_circumference`, `shoulder_width` |
| `arm` | `upper_arm_circumference`, `forearm_circumference`, `arm_length` |
| `legs` | `upper_thigh_circumference_left`, `upper_thigh_circumference_right`, `calf_circumference_left`, `calf_circumference_right`, `leg_length` |
| `ratios` | `shoulder_to_hip_ratio`, `waist_to_hip_ratio` |
| `physique_metrics` | `bmi` (Body Mass Index), `bri` (Body Roundness Index) |

All measurement values are numeric (centimeters for lengths/circumferences, unitless for ratios and indices).

---

## Error Handling

### Status Codes

| Code | Meaning |
|---|---|
| `200` | Success |
| `400` | Bad request — invalid or missing parameters |
| `401` | Unauthorized — missing or expired token |
| `403` | Forbidden — token valid but insufficient permissions |
| `404` | Not found — resource does not exist |
| `429` | Too many requests — rate limit exceeded |
| `500` | Internal server error |

### Error Response Format

All errors follow a consistent structure:

```json
{
  "error": {
    "code": "SCAN_NOT_FOUND",
    "message": "Scan not found",
    "timestamp": "2026-04-29T12:00:00Z"
  }
}
```

### Common Error Codes

| Code | Status | Description |
|---|---|---|
| `MISSING_AUTH` | 401 | No `Authorization` header provided |
| `INVALID_TOKEN` | 401 | Token is expired or malformed |
| `ORG_ACCESS_DENIED` | 403 | Token does not grant access to the requested organization |
| `ATHLETE_NOT_FOUND` | 404 | No athlete with the given ID exists in your organization |
| `SCAN_NOT_FOUND` | 404 | No scan with the given ID exists in your organization |
| `RATE_LIMIT_EXCEEDED` | 429 | You have exceeded your plan's request quota |

---

## Rate Limits

Request quotas depend on your subscription tier:

| Tier | Requests / Minute | Requests / Hour | Data Export / Day |
|---|---|---|---|
| Basic | 100 | 5,000 | 100 MB |
| Premium | 500 | 25,000 | 1 GB |
| Enterprise | 1,000 | 50,000 | 5 GB |

Every response includes rate-limit headers:

```
X-RateLimit-Limit: 500
X-RateLimit-Remaining: 495
X-RateLimit-Reset: 1745928000
```

When you receive a `429` response, back off and retry. We recommend exponential backoff:

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const response = await fetch(url, options);
    if (response.status !== 429) return response;

    const delay = Math.pow(2, attempt) * 1000; // 1 s, 2 s, 4 s
    await new Promise(r => setTimeout(r, delay));
  }
  throw new Error('Rate limit: max retries exceeded');
}
```

---

## Code Examples

### cURL

```bash
# 1. Authenticate
TOKEN=$(curl -s -X POST https://prod-wl.fitmatch.io/reflect-auth/client-credentials \
  -H "Content-Type: application/json" \
  -d '{
    "client_id": "YOUR_CLIENT_ID",
    "client_secret": "YOUR_CLIENT_SECRET",
    "grant_type": "client_credentials"
  }' | jq -r '.access_token')

# 2. List athletes
curl -s "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/athletes?limit=25" \
  -H "Authorization: Bearer $TOKEN"

# 3. Get a single athlete
curl -s "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/athletes/a1b2c3d4-e5f6-7890-abcd-ef1234567890" \
  -H "Authorization: Bearer $TOKEN"

# 4. List completed scans
curl -s "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/scans?status=completed&limit=50" \
  -H "Authorization: Bearer $TOKEN"

# 5. Get a single scan (includes operatorName and customId)
curl -s "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/scans/5bb5e4d3-005d-4f30-8ce2-616a4b2d1825" \
  -H "Authorization: Bearer $TOKEN"

# 6. Download a 3D model file
URL=$(curl -s "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/scans/5bb5e4d3-005d-4f30-8ce2-616a4b2d1825/model/url?extension=pcd" \
  -H "Authorization: Bearer $TOKEN" | jq -r '.download_url')
curl -o model.pcd "$URL"

# 7. Export scan data
curl -s -X POST "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/export" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"data_type":"scans","format":"json","filters":{"date_range":{"start":"2026-01-01","end":"2026-04-29"}}}'
```


### JavaScript / Node.js

```javascript
class ReflectAPI {
  constructor(clientId, clientSecret) {
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.baseUrl = 'https://prod-wl.fitmatch.io/reflect';
    this.token = null;
    this.tokenExpiresAt = 0;
  }

  async authenticate() {
    const res = await fetch(`${this.baseUrl}-auth/client-credentials`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        client_id: this.clientId,
        client_secret: this.clientSecret,
        grant_type: 'client_credentials',
      }),
    });
    if (!res.ok) throw new Error(`Auth failed: ${res.status}`);

    const data = await res.json();
    this.token = data.access_token;
    this.tokenExpiresAt = Date.now() + data.expires_in * 1000;
  }

  async request(path, options = {}) {
    if (!this.token || Date.now() >= this.tokenExpiresAt) {
      await this.authenticate();
    }
    const res = await fetch(`${this.baseUrl}/client/v1${path}`, {
      ...options,
      headers: { Authorization: `Bearer ${this.token}`, ...options.headers },
    });
    if (!res.ok) throw new Error(`API error: ${res.status}`);
    return res.json();
  }

  // Athletes
  listAthletes(orgId, params = {}) {
    const qs = new URLSearchParams(params).toString();
    return this.request(`/orgs/${orgId}/athletes${qs ? `?${qs}` : ''}`);
  }

  getAthlete(orgId, athleteId) {
    return this.request(`/orgs/${orgId}/athletes/${athleteId}`);
  }

  // Scans
  listScans(orgId, params = {}) {
    const qs = new URLSearchParams(params).toString();
    return this.request(`/orgs/${orgId}/scans${qs ? `?${qs}` : ''}`);
  }

  getScan(orgId, scanId) {
    return this.request(`/orgs/${orgId}/scans/${scanId}`);
  }

  // Measurements
  getMeasurements(orgId, scanId, types) {
    const qs = types ? `?types=${types}` : '';
    return this.request(`/orgs/${orgId}/scans/${scanId}/measurements${qs}`);
  }

  // 3D model download URL
  getModelUrl(orgId, scanId, extension = 'pcd') {
    return this.request(`/orgs/${orgId}/scans/${scanId}/model/url?extension=${extension}`);
  }

  // Export
  exportData(orgId, body) {
    return this.request(`/orgs/${orgId}/export`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(body),
    });
  }
}

// --- Usage ---

const api = new ReflectAPI('YOUR_CLIENT_ID', 'YOUR_CLIENT_SECRET');

// List athletes sorted by last name
const { athletes, pagination } = await api.listAthletes('ABC', {
  limit: 100,
  sort: 'last_name',
  order: 'asc',
});

// Get a single scan — response includes operatorName and customId
const { scan } = await api.getScan('ABC', '5bb5e4d3-005d-4f30-8ce2-616a4b2d1825');
console.log(scan.operatorName); // "Jane Trainer"
console.log(scan.customId);     // "PT-00451"

// Download a 3D model
const { download_url } = await api.getModelUrl('ABC', scan.scanId, 'pcd');
```

### Python

```python
import requests
from datetime import datetime, timedelta


class ReflectAPI:
    """Lightweight client for the Reflect Client Data API."""

    BASE = "https://prod-wl.fitmatch.io/reflect"

    def __init__(self, client_id: str, client_secret: str):
        self.client_id = client_id
        self.client_secret = client_secret
        self._token: str | None = None
        self._expires_at: datetime = datetime.min

    # --- auth ---

    def _ensure_token(self):
        if self._token and datetime.now() < self._expires_at:
            return
        resp = requests.post(
            f"{self.BASE}-auth/client-credentials",
            json={
                "client_id": self.client_id,
                "client_secret": self.client_secret,
                "grant_type": "client_credentials",
            },
        )
        resp.raise_for_status()
        data = resp.json()
        self._token = data["access_token"]
        self._expires_at = datetime.now() + timedelta(seconds=data["expires_in"] - 30)

    def _headers(self) -> dict:
        self._ensure_token()
        return {"Authorization": f"Bearer {self._token}"}

    # --- athletes ---

    def list_athletes(self, org_id: str, **params) -> dict:
        resp = requests.get(
            f"{self.BASE}/client/v1/orgs/{org_id}/athletes",
            headers=self._headers(),
            params=params,
        )
        resp.raise_for_status()
        return resp.json()

    def get_athlete(self, org_id: str, athlete_id: str) -> dict:
        resp = requests.get(
            f"{self.BASE}/client/v1/orgs/{org_id}/athletes/{athlete_id}",
            headers=self._headers(),
        )
        resp.raise_for_status()
        return resp.json()

    # --- scans ---

    def list_scans(self, org_id: str, **params) -> dict:
        resp = requests.get(
            f"{self.BASE}/client/v1/orgs/{org_id}/scans",
            headers=self._headers(),
            params=params,
        )
        resp.raise_for_status()
        return resp.json()

    def get_scan(self, org_id: str, scan_id: str) -> dict:
        resp = requests.get(
            f"{self.BASE}/client/v1/orgs/{org_id}/scans/{scan_id}",
            headers=self._headers(),
        )
        resp.raise_for_status()
        return resp.json()

    # --- measurements ---

    def get_measurements(self, org_id: str, scan_id: str, types: str | None = None) -> dict:
        params = {"types": types} if types else {}
        resp = requests.get(
            f"{self.BASE}/client/v1/orgs/{org_id}/scans/{scan_id}/measurements",
            headers=self._headers(),
            params=params,
        )
        resp.raise_for_status()
        return resp.json()

    # --- 3D model ---

    def get_model_url(self, org_id: str, scan_id: str, extension: str = "pcd") -> dict:
        resp = requests.get(
            f"{self.BASE}/client/v1/orgs/{org_id}/scans/{scan_id}/model/url",
            headers=self._headers(),
            params={"extension": extension},
        )
        resp.raise_for_status()
        return resp.json()

    # --- export ---

    def export_data(self, org_id: str, data_type: str, fmt: str = "json", filters: dict | None = None) -> dict:
        body = {"data_type": data_type, "format": fmt}
        if filters:
            body["filters"] = filters
        resp = requests.post(
            f"{self.BASE}/client/v1/orgs/{org_id}/export",
            headers=self._headers(),
            json=body,
        )
        resp.raise_for_status()
        return resp.json()


# --- Usage ---

api = ReflectAPI("YOUR_CLIENT_ID", "YOUR_CLIENT_SECRET")

# List athletes
data = api.list_athletes("ABC", limit=100, sort="last_name", order="asc")
for athlete in data["athletes"]:
    print(athlete["firstName"], athlete["lastName"])

# Get a single scan — includes operatorName and customId
scan_data = api.get_scan("ABC", "5bb5e4d3-005d-4f30-8ce2-616a4b2d1825")
scan = scan_data["scan"]
print(scan["operatorName"])  # "Jane Trainer"
print(scan["customId"])       # "PT-00451"

# Export all completed scans for Q1 2026
export = api.export_data(
    "ABC",
    data_type="scans",
    filters={
        "date_range": {"start": "2026-01-01", "end": "2026-03-31"},
        "status": "completed",
    },
)
```

---

## Security Best Practices

**Credentials**
- Store `client_id` and `client_secret` in environment variables or a secrets manager. Never commit them to source control or embed them in client-side code.
- Use separate credentials for development and production environments.

**Tokens**
- Cache tokens for their full 1-hour lifetime to avoid unnecessary auth calls.
- Implement automatic refresh: when you receive a `401`, request a new token and retry.

**Network**
- Always use HTTPS. The API rejects plain HTTP.
- Set reasonable request timeouts (we recommend 30 seconds).
- Never log tokens or credentials.

**Access control**
- Tokens are scoped to your organization. You can only access data belonging to your `org_id`.
- All API access is logged for audit purposes.

---

## Support

If you have questions or run into issues, reach out to your FitMatch account representative.
