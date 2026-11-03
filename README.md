]633;E;head -n 876 README.md;03d4d042-e87b-48c3-92f2-ef49dbb3f268]633;C# Reflect Client Data API Reference

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
  - [Get 3D Model Download URL](#get-3d-model-download-url)
  - [Export Data](#export-data)
- [Data Models](#data-models)
- [Error Handling](#error-handling)
- [Rate Limits](#rate-limits)
- [Code Examples](#code-examples)
- [Versioning and Compatibility](#versioning-and-compatibility)
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

All endpoint paths in this document are relative to this base URL. The API is served over HTTPS only.

### Your Organization ID

Every data endpoint includes your organization identifier, `{org_id}`, in its path (for example `/orgs/ABC/athletes`). It is your organization's short abbreviation and is provided together with your API credentials. Your access token is tied to that one organization: a request for any other `org_id` returns `403 ORG_ACCESS_DENIED`.

### Field Naming

Athlete and scan objects use `camelCase` field names (`athleteId`, `scanStatus`). Measurement fields, pagination, export rows, and the 3D model URL response use `snake_case` (`body_height`, `has_more`, `scan_id`, `download_url`). Each field's name is fixed, so map them exactly as shown in this document.

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

If the request is rejected (for example, an incorrect `client_id` or `client_secret`), the endpoint returns a non-`200` status with a JSON body of the form `{"message": "..."}`. Don't retry a rejected credential request in a loop: check your credentials, and contact support if they should be valid.

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
      "lastScannedDate": "2026-03-15T14:30:00+00:00",
      "createdAt": "2025-06-01T10:00:00+00:00"
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
    "notes": null,
    "athleteEmail": "john.doe@example.com",
    "isTestAthlete": false,
    "organization": "ABC",
    "organizationName": "Example Sports Club",
    "lastScannedDate": "2026-03-15T14:30:00+00:00",
    "createdAt": "2025-06-01T10:00:00+00:00",
    "customAthleteId": "PT-00451"
  },
  "requested_at": "2026-04-29T12:00:00.000000Z"
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
| `status` | query | no | Filter by status: `pending`, `queued`, `processing`, `completed`, `error` |
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
          "body_height": 1.855
        },
        "torso": {
          "chest_circumference": 0.985
        }
      },
      "processingTime": 12.5,
      "scanFileUrl": "https://...",
      "isTestScan": false,
      "timestamp": "2026-03-15T14:30:00+00:00"
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
        "body_height": 1.855,
        "seated_height": 0.952
      },
      "torso": {
        "chest_circumference": 0.985,
        "waist_circumference": 0.823,
        "hip_circumference": 0.957
      },
      "arm": {
        "upper_arm_circumference_left": 0.321,
        "upper_arm_circumference_right": 0.324,
        "forearm_circumference_left": 0.278,
        "forearm_circumference_right": 0.280
      },
      "legs": {
        "upper_thigh_circumference_left": 0.582,
        "upper_thigh_circumference_right": 0.585,
        "calf_circumference_left": 0.389,
        "calf_circumference_right": 0.391
      },
      "ratios": {
        "waist_to_hip_ratio": 0.86,
        "waist_to_height_ratio": 0.44
      },
      "physique_metrics": {
        "bmi": 23.9,
        "bri": 3.45
      },
      "user_inputs": {
        "gender": "Male",
        "weight": 181,
        "height_ft": 6,
        "height_in": 1
      }
    },
    "processingTime": 12.5,
    "scanFileUrl": "https://...",
    "isTestScan": false,
    "timestamp": "2026-03-15T14:30:00+00:00"
  },
  "requested_at": "2026-04-29T12:00:00.000000Z"
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
      "body_height": 1.855,
      "seated_height": 0.952
    },
    "torso": {
      "chest_circumference": 0.985,
      "waist_circumference": 0.823
    }
  },
  "measurement_count": 2,
  "requested_types": ["height", "torso"],
  "requested_at": "2026-04-29T12:00:00.000000Z"
}
```

When `types` is omitted, all available measurement groups are returned and `requested_types` is the string `"all"`. `measurement_count` is the number of measurement groups returned.

---

### Get 3D Model Download URL

Generate a secure, time-limited download URL for a scan's 3D model file. This endpoint returns a link, not the file itself.

**`GET /orgs/{org_id}/scans/{scan_id}/model/url`**

| Parameter | In | Required | Description |
|---|---|---|---|
| `org_id` | path | yes | Your organization identifier |
| `scan_id` | path | yes | Scan UUID |
| `extension` | query | no | File format: `pcd`, `usdz`, or `obj` (default `pcd`) |
| `expiration` | query | no | URL lifetime in seconds (default `3600`, max `86400`) |

**Response** `200 OK`

```json
{
  "download_url": "https://<pre-signed-url>...",
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
| `filters.date_range.start` | string | no | ISO 8601 start date (inclusive) |
| `filters.date_range.end` | string | no | ISO 8601 end date. A date without a time is treated as midnight at the start of that day — to include a whole day, pass the following day or a full timestamp. |
| `filters.status` | string | no | Filter scans by status (`scans` export only) |

**Response** `200 OK`

For JSON format, the response body is a flat array of rows. Export rows use `snake_case` field names.

`data_type: "scans"` — one row per scan:

```json
[
  {
    "scan_id": "5bb5e4d3-005d-4f30-8ce2-616a4b2d1825",
    "status": "completed",
    "measurements": { "..." : "..." },
    "processing_time": 12.5,
    "created_at": "2026-03-15T14:30:00+00:00",
    "athlete_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "first_name": "John",
    "last_name": "Doe",
    "team_abbreviation": "ABC",
    "custom_id": "PT-00451",
    "operator_name": "Jane Trainer"
  }
]
```

`data_type: "measurements"` — one row per measurement group per scan; `measurement_type` is the group name (e.g. `torso`) and `measurement_value` is that group's object:

```json
[
  {
    "scan_id": "5bb5e4d3-005d-4f30-8ce2-616a4b2d1825",
    "athlete_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "athlete_name": "John Doe",
    "measurement_type": "torso",
    "measurement_value": { "chest_circumference": 0.985, "waist_circumference": 0.823 },
    "scan_date": "2026-03-15T14:30:00+00:00"
  }
]
```

`data_type: "athletes"` — one row per athlete profile in your organization.

For CSV format, the response is returned with `Content-Type: text/csv` and a `Content-Disposition` header for download. Read the body as text, not JSON. Nested values such as `measurements` are written as JSON strings inside their CSV cell. If no rows match your filters, the CSV body is empty (no header row).

**Date ranges:** to export a whole period, set `end` to the first day *after* it. For example, all of Q1 2026 is `"start": "2026-01-01", "end": "2026-04-01"`.

---

## Data Models

Timestamps on athletes and scans (`createdAt`, `lastScannedDate`, `timestamp`, and export `created_at` / `scan_date`) are ISO 8601 in UTC with an explicit offset, e.g. `2026-03-15T14:30:00.123456+00:00`. Fractional seconds may be present or absent. `requested_at` uses the `Z` suffix. Use a standard ISO 8601 parser.

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
| `isTestAthlete` | `boolean` | `true` for test/demo profiles |
| `organization` | `string \| null` | Organization abbreviation |
| `organizationName` | `string \| null` | Organization display name |
| `lastScannedDate` | `string \| null` | Most recent scan timestamp (ISO 8601) |
| `createdAt` | `string` | Profile creation timestamp (ISO 8601) |
| `customAthleteId` | `string` | Your custom identifier for this athlete. Omitted when not set. |

### Scan

| Field | Type | Description |
|---|---|---|
| `scanId` | `string` | Unique identifier (UUID) |
| `athleteId` | `string` | Associated athlete UUID |
| `teamId` | `string` | Organization identifier |
| `scanStatus` | `string` | `pending`, `queued`, `processing`, `completed`, or `error` |
| `operatorName` | `string \| null` | Name of the person who performed the scan |
| `customId` | `string \| null` | Your custom identifier for this scan (e.g. internal tracking number) |
| `measurements` | `object` | Body measurements grouped by region (see below) |
| `processingTime` | `number \| null` | Processing duration in seconds |
| `scanFileUrl` | `string \| null` | Storage location of the raw scan file. Not directly downloadable — use [Get 3D Model Download URL](#get-3d-model-download-url) for a download link. |
| `isTestScan` | `boolean` | `true` for test/demo scans |

### Test Data

Test and demo records are returned alongside real ones: list endpoints and exports do not filter them out. On athlete and scan objects, use `isTestAthlete` and `isTestScan` to exclude them from your own reporting. `scans` and `measurements` export rows do not carry a test flag, so if you need to exclude test scans from an export, identify them with [List Scans](#list-scans) and match on `scan_id`.
| `timestamp` | `string` | Scan capture time (ISO 8601) |

### Measurements

Measurements are returned as a nested object grouped by body region (`height`, `head`, `torso`, `back`, `arm`, `legs`, `ratios`, `physique_metrics`, `user_inputs`). Each value is a bare number (or `null`); units are **not** included in the response — use the table below.

**Which fields you receive depends on your organization's configuration.** The table below lists every measurement the API can return. Your organization is configured to receive a subset of these, so a given response will contain only some groups and fields. A field may also be absent, or `null`, when it could not be computed for a particular scan. Contact your FitMatch representative to review or change the measurements enabled for your organization.

Build your integration to tolerate missing groups, missing fields, and `null` values.

**Units**

| Unit | Applies to |
|---|---|
| `m` (meters) | All lengths, heights, circumferences, and breadths. Multiply by `100` for centimeters or by `39.3701` for inches. |
| `kg` (kilograms) | Mass values in `physique_metrics` (`estimated_weight`, `lean_mass`, `total_fat`, `bone_mineral_content`). |
| `years` | `physique_metrics.phv` |
| `%` | `physique_metrics.total_fat_percent` |
| unitless | Ratios and indices (`ratios.*`, `bmi`, `bri`) |
| `lb`, `ft`, `in` | `user_inputs` values, as entered by the operator at scan time |

**Field reference**

Fields listed as `name`, `name_left`, `name_right` are available as a combined value and as separate left/right values; your configuration determines which of these you receive. `user_inputs` is always included and contains only the values entered at scan time.

| Group | Field(s) | Description | Unit |
|---|---|---|---|
| `height` | `body_height` | Standing body height | m |
| `height` | `seated_height` | Seated height | m |
| `height` | `predicted_adult_height_phv` | Predicted adult height (maturation-based) | m |
| `height` | `predicted_adult_height_parents` | Predicted adult height (parental-height method; only when both parent heights were entered) | m |
| `head` | `head_circumference` | Head circumference | m |
| `head` | `neck_circumference` | Neck circumference | m |
| `torso` | `chest_circumference` | Chest circumference | m |
| `torso` | `underbust_circumference` | Underbust circumference | m |
| `torso` | `waist_circumference` | Low waist circumference | m |
| `torso` | `narrowest_waist_circumference` | Narrowest waist circumference | m |
| `torso` | `fullest_waist_circumference` | Fullest waist circumference | m |
| `torso` | `hip_circumference` | Hip circumference | m |
| `torso` | `transverse_chest_breadth` | Transverse (side-to-side) chest breadth | m |
| `torso` | `anterior_posterior_chest_breadth` | Anterior–posterior (front-to-back) chest breadth | m |
| `torso` | `waist_breadth` | Waist breadth | m |
| `torso` | `hip_breadth` | Hip breadth | m |
| `back` | `back_length` | Back length | m |
| `back` | `back_width` | Back width | m |
| `arm` | `wingspan` | Wingspan | m |
| `arm` | `arm_length`, `arm_length_left`, `arm_length_right` | Arm length | m |
| `arm` | `upper_arm_circumference`, `upper_arm_circumference_left`, `upper_arm_circumference_right` | Upper arm circumference | m |
| `arm` | `forearm_circumference`, `forearm_circumference_left`, `forearm_circumference_right` | Forearm circumference | m |
| `arm` | `wrist_circumference`, `wrist_circumference_left`, `wrist_circumference_right` | Wrist circumference | m |
| `arm` | `elbow_breadth`, `elbow_breadth_left`, `elbow_breadth_right` | Elbow breadth | m |
| `legs` | `leg_length`, `leg_length_left`, `leg_length_right` | Leg length | m |
| `legs` | `pant_length` | Pant (inseam) length | m |
| `legs` | `upper_thigh_circumference`, `upper_thigh_circumference_left`, `upper_thigh_circumference_right` | Upper thigh circumference | m |
| `legs` | `mid_thigh_circumference`, `mid_thigh_circumference_left`, `mid_thigh_circumference_right` | Mid thigh circumference | m |
| `legs` | `calf_circumference`, `calf_circumference_left`, `calf_circumference_right` | Calf circumference | m |
| `legs` | `ankle_circumference`, `ankle_circumference_left`, `ankle_circumference_right` | Ankle circumference | m |
| `ratios` | `waist_to_hip_ratio` | Waist-to-hip ratio | unitless |
| `ratios` | `waist_to_height_ratio` | Waist-to-height ratio | unitless |
| `ratios` | `seated_to_standing_ratio` | Seated-to-standing height ratio | unitless |
| `ratios` | `muscle_to_bone_ratio` | Muscle-to-bone ratio | unitless |
| `physique_metrics` | `bmi` | Body Mass Index | unitless (kg/m²) |
| `physique_metrics` | `bri` | Body Roundness Index | unitless |
| `physique_metrics` | `estimated_weight` | Estimated body weight | kg |
| `physique_metrics` | `lean_mass` | Lean mass | kg |
| `physique_metrics` | `total_fat` | Total fat mass | kg |
| `physique_metrics` | `total_fat_percent` | Body fat percentage | % |
| `physique_metrics` | `bone_mineral_content` | Bone mineral content | kg |
| `physique_metrics` | `phv` | Estimated age at Peak Height Velocity (PHV) | years |
| `user_inputs` | `gender` | Gender entered at scan time | — |
| `user_inputs` | `weight` | Body weight entered at scan time | lb |
| `user_inputs` | `height_ft`, `height_in` | Body height entered at scan time (feet + inches) | ft, in |
| `user_inputs` | `father_height_in`, `mother_height_in` | Parent heights, when entered at scan time | in |

---

## Error Handling

### Status Codes

| Code | Meaning |
|---|---|
| `200` | Success |
| `400` | Bad request — invalid or missing parameters |
| `401` | Unauthorized — missing or expired token |
| `403` | Forbidden — token valid but not permitted for this organization or resource, or request rate-limited (see [Rate Limits](#rate-limits)) |
| `404` | Not found — resource or route does not exist |
| `500` | Internal server error |

### Error Response Format

Errors from the data endpoints (`/orgs/...`) use this structure:

```json
{
  "error": {
    "code": "SCAN_NOT_FOUND",
    "message": "Scan not found",
    "timestamp": "2026-04-29T12:00:00Z"
  }
}
```

The token endpoint returns errors as `{"message": "..."}` instead. Rate-limit responses (see [Rate Limits](#rate-limits)) are not JSON.

### Common Error Codes

| Code | Status | Description |
|---|---|---|
| `MISSING_AUTH` | 401 | No `Authorization` header provided |
| `INVALID_TOKEN` | 401 | Token is expired or malformed |
| `ORG_ACCESS_DENIED` | 403 | Token does not grant access to the requested organization |
| `ATHLETE_NOT_FOUND` | 404 | No athlete with the given ID exists |
| `ATHLETE_ACCESS_DENIED` | 403 | The athlete exists but belongs to a different organization |
| `SCAN_NOT_FOUND` | 404 | No scan with the given ID exists |
| `SCAN_ACCESS_DENIED` | 403 | The scan exists but belongs to a different organization |
| `MODEL_FILE_NOT_FOUND` | 404 | No 3D model file of the requested format exists for this scan |
| `INVALID_PAGINATION` | 400 | `limit` or `offset` is not a valid value |
| `MISSING_DATA_TYPE` | 400 | Export request has no `data_type` |
| `INVALID_DATA_TYPE` | 400 | Export `data_type` is not `athletes`, `scans`, or `measurements` |
| `INVALID_FORMAT` | 400 | Export `format` is not `json` or `csv` |
| `INVALID_JSON` | 400 | Request body is not valid JSON |
| `ROUTE_NOT_FOUND` | 404 | No endpoint matches the method and path |
| `INTERNAL_ERROR` | 500 | Unexpected server error — safe to retry |

---

## Rate Limits

Requests are rate-limited per client IP address. If you exceed the limit, requests are rejected with HTTP `403` and a non-JSON body until the request rate drops; the block clears automatically within a few minutes. Responses do not include rate-limit headers.

To stay within limits, page through results with `limit` / `offset` rather than issuing many parallel requests, use [Export Data](#export-data) for bulk pulls, and back off when you receive a `403` that does not have a JSON `error` body:

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const response = await fetch(url, options);
    if (response.status !== 403) return response;
    const isJson = (response.headers.get('content-type') || '').includes('application/json');
    if (isJson) return response; // a real permission error — do not retry

    const delay = Math.pow(2, attempt) * 30000; // 30 s, 60 s, 120 s
    await new Promise(r => setTimeout(r, delay));
  }
  throw new Error('Rate limit: max retries exceeded');
}
```

If your integration needs a higher sustained request rate, contact your FitMatch representative.

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
  -d '{"data_type":"scans","format":"json","filters":{"date_range":{"start":"2026-01-01","end":"2026-04-01"}}}'

# 8. Export the same scans as CSV and save to a file
curl -s -X POST "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/export" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"data_type":"scans","format":"csv","filters":{"date_range":{"start":"2026-01-01","end":"2026-04-01"}}}' \
  -o scans_q1_2026.csv
```

### Paging Through Results

List endpoints return at most `100` items per call. To fetch everything, advance `offset` by `limit` until `pagination.has_more` is `false`:

```bash
OFFSET=0
while : ; do
  PAGE=$(curl -s "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/athletes?limit=100&offset=$OFFSET" \
    -H "Authorization: Bearer $TOKEN")
  echo "$PAGE" | jq -c '.athletes[]'
  [ "$(echo "$PAGE" | jq -r '.pagination.has_more')" = "true" ] || break
  OFFSET=$((OFFSET + 100))
done
```

The JavaScript and Python clients below include a `listAllAthletes` / `list_all_athletes` helper that does the same thing.


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
    if (!res.ok) {
      const body = await res.json().catch(() => ({}));
      throw new Error(`Auth failed: ${res.status} ${body.message || ''}`.trim());
    }

    const data = await res.json();
    this.token = data.access_token;
    // Refresh 30 seconds early so a token never expires mid-request
    this.tokenExpiresAt = Date.now() + (data.expires_in - 30) * 1000;
  }

  async request(path, options = {}) {
    if (!this.token || Date.now() >= this.tokenExpiresAt) {
      await this.authenticate();
    }
    const res = await fetch(`${this.baseUrl}/client/v1${path}`, {
      ...options,
      headers: { Authorization: `Bearer ${this.token}`, ...options.headers },
    });
    const isJson = (res.headers.get('content-type') || '').includes('application/json');
    if (!res.ok) {
      // Data endpoints return {"error": {"code", "message"}}; rate-limit responses are not JSON
      const body = isJson ? await res.json() : null;
      const err = new Error(body?.error?.message || `HTTP ${res.status}`);
      err.status = res.status;
      err.code = body?.error?.code || (res.status === 403 ? 'RATE_LIMITED' : null);
      throw err;
    }
    // CSV exports come back as text
    return isJson ? res.json() : res.text();
  }

  // Athletes
  listAthletes(orgId, params = {}) {
    const qs = new URLSearchParams(params).toString();
    return this.request(`/orgs/${orgId}/athletes${qs ? `?${qs}` : ''}`);
  }

  // Fetch every athlete, one page at a time
  async listAllAthletes(orgId, params = {}) {
    const all = [];
    let offset = 0;
    while (true) {
      const { athletes, pagination } = await this.listAthletes(orgId, { ...params, limit: 100, offset });
      all.push(...athletes);
      if (!pagination.has_more) return all;
      offset += pagination.limit;
    }
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

// Every athlete in the organization, excluding test profiles
const everyone = (await api.listAllAthletes('ABC')).filter(a => !a.isTestAthlete);

// Export Q1 2026 scans as CSV (returned as a string)
const csv = await api.exportData('ABC', {
  data_type: 'scans',
  format: 'csv',
  filters: { date_range: { start: '2026-01-01', end: '2026-04-01' } },
});

// Handle errors by code
try {
  await api.getScan('ABC', '00000000-0000-0000-0000-000000000000');
} catch (err) {
  if (err.code === 'SCAN_NOT_FOUND') console.log('No such scan');
  else throw err;
}
```

### Python

```python
import requests
from datetime import datetime, timedelta


class ReflectAPIError(Exception):
    """Raised for non-2xx responses. `code` is the API error code, e.g. SCAN_NOT_FOUND."""

    def __init__(self, status: int, code: str | None, message: str):
        super().__init__(" ".join(str(p) for p in (status, code, message) if p))
        self.status, self.code, self.message = status, code, message


class ReflectAPI:
    """Lightweight client for the Reflect Client Data API."""

    BASE = "https://prod-wl.fitmatch.io/reflect"
    TIMEOUT = 30  # seconds

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
            timeout=self.TIMEOUT,
        )
        if not resp.ok:
            try:
                message = resp.json().get("message", "")
            except ValueError:
                message = resp.text
            raise ReflectAPIError(resp.status_code, None, f"authentication failed: {message}")
        data = resp.json()
        self._token = data["access_token"]
        # Refresh 30 seconds early so a token never expires mid-request
        self._expires_at = datetime.now() + timedelta(seconds=data["expires_in"] - 30)

    # --- one place for every API call ---

    def _request(self, method: str, path: str, **kwargs):
        self._ensure_token()
        resp = requests.request(
            method,
            f"{self.BASE}/client/v1{path}",
            headers={"Authorization": f"Bearer {self._token}"},
            timeout=self.TIMEOUT,
            **kwargs,
        )
        is_json = "application/json" in resp.headers.get("Content-Type", "")
        if not resp.ok:
            # Data endpoints return {"error": {"code", "message"}}; rate-limit responses are not JSON
            error = resp.json().get("error", {}) if is_json else {}
            code = error.get("code") or ("RATE_LIMITED" if resp.status_code == 403 else None)
            raise ReflectAPIError(resp.status_code, code, error.get("message", resp.reason))
        # CSV exports come back as text
        return resp.json() if is_json else resp.text

    # --- athletes ---

    def list_athletes(self, org_id: str, **params) -> dict:
        return self._request("GET", f"/orgs/{org_id}/athletes", params=params)

    def list_all_athletes(self, org_id: str, **params) -> list[dict]:
        """Fetch every athlete, one page at a time."""
        athletes, offset = [], 0
        while True:
            page = self.list_athletes(org_id, **params, limit=100, offset=offset)
            athletes.extend(page["athletes"])
            if not page["pagination"]["has_more"]:
                return athletes
            offset += page["pagination"]["limit"]

    def get_athlete(self, org_id: str, athlete_id: str) -> dict:
        return self._request("GET", f"/orgs/{org_id}/athletes/{athlete_id}")

    # --- scans ---

    def list_scans(self, org_id: str, **params) -> dict:
        return self._request("GET", f"/orgs/{org_id}/scans", params=params)

    def get_scan(self, org_id: str, scan_id: str) -> dict:
        return self._request("GET", f"/orgs/{org_id}/scans/{scan_id}")

    # --- measurements ---

    def get_measurements(self, org_id: str, scan_id: str, types: str | None = None) -> dict:
        params = {"types": types} if types else {}
        return self._request("GET", f"/orgs/{org_id}/scans/{scan_id}/measurements", params=params)

    # --- 3D model ---

    def get_model_url(self, org_id: str, scan_id: str, extension: str = "pcd") -> dict:
        return self._request("GET", f"/orgs/{org_id}/scans/{scan_id}/model/url",
                             params={"extension": extension})

    # --- export ---

    def export_data(self, org_id: str, data_type: str, fmt: str = "json",
                    filters: dict | None = None) -> list | str:
        """Returns a list of rows for JSON, or the CSV text for fmt="csv"."""
        body = {"data_type": data_type, "format": fmt}
        if filters:
            body["filters"] = filters
        return self._request("POST", f"/orgs/{org_id}/export", json=body)


# --- Usage ---

api = ReflectAPI("YOUR_CLIENT_ID", "YOUR_CLIENT_SECRET")

# List athletes
data = api.list_athletes("ABC", limit=100, sort="last_name", order="asc")
for athlete in data["athletes"]:
    print(athlete["firstName"], athlete["lastName"])

# Every athlete in the organization, excluding test profiles
everyone = [a for a in api.list_all_athletes("ABC") if not a["isTestAthlete"]]

# Get a single scan — includes operatorName and customId
scan_data = api.get_scan("ABC", "5bb5e4d3-005d-4f30-8ce2-616a4b2d1825")
scan = scan_data["scan"]
print(scan["operatorName"])  # "Jane Trainer"
print(scan["customId"])       # "PT-00451"

# Export all completed scans for Q1 2026 (end is the day after the period)
rows = api.export_data(
    "ABC",
    data_type="scans",
    filters={
        "date_range": {"start": "2026-01-01", "end": "2026-04-01"},
        "status": "completed",
    },
)

# The same export as a CSV file
csv_text = api.export_data("ABC", data_type="scans", fmt="csv",
                           filters={"date_range": {"start": "2026-01-01", "end": "2026-04-01"}})
with open("scans_q1_2026.csv", "w", newline="") as f:
    f.write(csv_text)

# Handle errors by code
try:
    api.get_scan("ABC", "00000000-0000-0000-0000-000000000000")
except ReflectAPIError as err:
    if err.code == "SCAN_NOT_FOUND":
        print("No such scan")
    else:
        raise
```

---

## Versioning and Compatibility

The API version is part of the base URL (`/v1`). Within `v1`:

- Existing endpoints, parameters, and fields keep their names and meaning.
- New endpoints, optional parameters, and response fields may be added at any time. Ignore fields you don't recognize rather than failing on them.
- Measurement groups and fields can vary by organization (see [Measurements](#measurements)), so treat any single field as optional.

A breaking change would ship as a new version (`/v2`), with advance notice through your FitMatch representative.

---

## Security Best Practices

**Credentials**
- Store `client_id` and `client_secret` in environment variables or a secrets manager. Never commit them to source control or embed them in client-side code (browser or mobile apps). Call the API from your servers only.
- If you believe your credentials have been exposed, contact your FitMatch representative to have them replaced.

**Tokens**
- Cache tokens for their full 1-hour lifetime to avoid unnecessary auth calls.
- Implement automatic refresh: when you receive a `401 INVALID_TOKEN`, request a new token and retry the call once.

**Network**
- Always use HTTPS.
- Set reasonable request timeouts (we recommend 30 seconds).
- Never log tokens or credentials.

**Access control**
- Tokens are scoped to your organization. You can only access data belonging to your `org_id`.
- All API access is logged for audit purposes.

---

## Support

If you have questions or run into issues, reach out to your FitMatch account representative.

To help us resolve a problem quickly, include:

- the endpoint and method (for example `GET /orgs/ABC/scans`) and any query parameters or request body, **without** your token or credentials
- the HTTP status and the `error.code` and `error.timestamp` from the response
- the approximate time of the request, with time zone
