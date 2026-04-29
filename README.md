# Reflect Client Data API Reference

Access athlete biometric data and 3D body scan measurements programmatically with our secure REST API.

---

## Getting Started

The Reflect Client Data API enables sports organizations to integrate athlete scan data directly into their existing systems, analytics platforms, and custom applications.

### Quick Start

1. **Get API Credentials** - Automatically provisioned when you purchase a license
2. **Authenticate** - Obtain an access token using OAuth 2.0
3. **Make Requests** - Access your athlete and scan data

### Base URLs

- **Production**: `https://prod-wl.fitmatch.io/reflect/client/v1`

### Authentication

All API requests require OAuth 2.0 authentication. Include your access token in the Authorization header:

```bash
Authorization: Bearer YOUR_ACCESS_TOKEN
```

---

## Authentication

### Get Access Token

Obtain an access token using your API credentials:

**Endpoint:** `POST https://prod-wl.fitmatch.io/reflect-auth/client-credentials`

**Request:**
```json
{
  "client_id": "your_client_id",
  "client_secret": "your_client_secret",
  "grant_type": "client_credentials"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Tokens are valid for 1 hour and can be reused for multiple requests.

---

## Core Endpoints

### List Athletes

Retrieve a paginated list of all athletes in your organization.

**Endpoint:** `GET /orgs/{org_id}/athletes`

**Parameters:**
- `org_id` (path) - Your organization identifier

**Query Parameters:**
- `limit` (optional) - Number of results per page (default: 50, max: 100)
- `offset` (optional) - Number of results to skip for pagination (default: 0)
- `sort` (optional) - Sort field: `created_at`, `first_name`, `last_name`, `last_scanned_date` (default: `created_at`)
- `order` (optional) - Sort order: `asc` or `desc` (default: `desc`)

**Response:**
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
      "teamName": "ABC Sports Team",
      "biometrics": {
        "height": 185.5,
        "weight": 82.3
      },
      "isTestAthlete": false,
      "organization": "ABC",
      "organizationName": "ABC Sports Organization",
      "lastScannedDate": "2025-12-01T14:30:00Z",
      "createdAt": "2024-01-15T10:00:00Z"
    }
  ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 125,
    "has_more": true
  },
  "requested_at": "2025-12-04T03:30:00.123456Z"
}
```

---

### Get Athlete

Retrieve detailed athlete profile information including biometrics and team association.

**Endpoint:** `GET /orgs/{org_id}/athletes/{athlete_id}`

**Parameters:**
- `org_id` (path) - Your organization identifier
- `athlete_id` (path) - Unique athlete identifier

**Response:**
```json
{
  "athlete": {
    "athleteId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "firstName": "John",
    "lastName": "Doe",
    "position": "Forward",
    "birthday": "2000-05-15",
    "teamId": "ABC",
    "teamName": "ABC Sports Team",
    "biometrics": {
      "height": 185.5,
      "weight": 82.3,
      "wingspan": 195.2
    },
    "athleteEmail": "john.doe@example.com",
    "isTestAthlete": false,
    "organization": "ABC",
    "organizationName": "ABC Sports Organization",
    "lastScannedDate": "2025-12-01T14:30:00Z",
    "createdAt": "2024-01-15T10:00:00Z"
  }
}
```

---

### List Scans

Retrieve a paginated list of all scans for your organization with optional filtering.

**Endpoint:** `GET /orgs/{org_id}/scans`

**Parameters:**
- `org_id` (path) - Your organization identifier

**Query Parameters:**
- `limit` (optional) - Number of results per page (default: 50, max: 100)
- `offset` (optional) - Number of results to skip for pagination (default: 0)
- `athlete_id` (optional) - Filter by specific athlete ID
- `status` (optional) - Filter by scan status: `pending`, `processing`, `completed`, `error`
- `start_date` (optional) - Filter scans after this date (ISO 8601 format)
- `end_date` (optional) - Filter scans before this date (ISO 8601 format)
- `sort` (optional) - Sort field: `created_at`, `status`, `processing_time` (default: `created_at`)
- `order` (optional) - Sort order: `asc` or `desc` (default: `desc`)

**Response:**
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
      "timestamp": "2025-12-01T14:30:00Z"
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
  "requested_at": "2025-12-04T03:30:00.123456Z"
}
```

---

### Get Scan Results

Access complete scan data including all body measurements and processing metadata.

**Endpoint:** `GET /orgs/{org_id}/scans/{scan_id}`

**Parameters:**
- `org_id` (path) - Your organization identifier
- `scan_id` (path) - Unique scan identifier

**Response:**
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
    "timestamp": "2025-12-01T14:30:00Z"
  }
}
```

---

### Get Measurements

Retrieve only measurement data for a specific scan, with optional filtering by measurement type.

**Endpoint:** `GET /orgs/{org_id}/scans/{scan_id}/measurements`

**Parameters:**
- `org_id` (path) - Your organization identifier
- `scan_id` (path) - Unique scan identifier
- `types` (query, optional) - Comma-separated measurement types (e.g., `height,torso,arm`)

**Response:**
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
    },
    "etc...": {
      "etc..": 99.9,
      "etc..": 99.9,
    }
  },
  "measurement_count": 2,
  "requested_types": "height,torso",
  "requested_at": "2025-12-04T03:30:00.123456Z"
}
```

---

### Export Data

Bulk export athlete and scan data in JSON or CSV format with flexible filtering options.

**Endpoint:** `POST /orgs/{org_id}/export`

**Parameters:**
- `org_id` (path) - Your organization identifier

**Request:**
```json
{
  "data_type": "scans",
  "format": "json",
  "filters": {
    "date_range": {
      "start": "2025-01-01",
      "end": "2025-12-31"
    }
  }
}
```

**Response:**

The response format depends on the `data_type` requested. The data is returned directly as an array or object, not wrapped.

For `data_type: "scans"`:
```json
[
  {
    "scan_id": "...",
    "status": "completed",
    "measurements": {...},
    "athlete_id": "...",
    "first_name": "John",
    "last_name": "Doe",
    "team_abbreviation": "ABC",
    "operator_name": "Dr. Smith",
    "custom_id": "SCAN-2025-0042",
    "created_at": "2025-12-01T14:30:00Z"
  },
  ...
]
```

For CSV format, the response will be a CSV file with appropriate headers.

---

### Get 3D Model File

Generate a secure, time-limited download URL for 3D model files (PCD or USDZ format).

**Endpoint:** `GET /orgs/{org_id}/scans/{scan_id}/model/url`

**Parameters:**
- `org_id` (path) - Your organization identifier
- `scan_id` (path) - Unique scan identifier
- `extension` (query, optional) - File format: `pcd` or `usdz` (default: `pcd`)
- `expiration` (query, optional) - URL expiration in seconds (default: 3600, max: 86400)

**Response:**
```json
{
  "download_url": "https://...",
  "expires_in": 3600,
  "expires_at": "2025-12-04T04:30:00Z",
  "scan_id": "5bb5e4d3-005d-4f30-8ce2-616a4b2d1825"
}
```

---

## Code Examples

### JavaScript/Node.js

```javascript
// Initialize API client
class ReflectAPI {
  constructor(clientId, clientSecret, environment = 'prod-wl') {
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.baseUrl = `https://${environment}.fitmatch.io`;
    this.token = null;
  }

  async authenticate() {
    const response = await fetch(`${this.baseUrl}/reflect-auth/client-credentials`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        client_id: this.clientId,
        client_secret: this.clientSecret,
        grant_type: 'client_credentials'
      })
    });
    
    const data = await response.json();
    this.token = data.access_token;
    return this.token;
  }

  async getAthlete(orgId, athleteId) {
    if (!this.token) await this.authenticate();
    
    const response = await fetch(
      `${this.baseUrl}/client/v1/orgs/${orgId}/athletes/${athleteId}`,
      {
        headers: { 'Authorization': `Bearer ${this.token}` }
      }
    );
    
    return await response.json();
  }

  async listAthletes(orgId, options = {}) {
    if (!this.token) await this.authenticate();
    
    const params = new URLSearchParams();
    if (options.limit) params.append('limit', options.limit);
    if (options.offset) params.append('offset', options.offset);
    if (options.sort) params.append('sort', options.sort);
    if (options.order) params.append('order', options.order);
    
    const queryString = params.toString() ? `?${params.toString()}` : '';
    const response = await fetch(
      `${this.baseUrl}/client/v1/orgs/${orgId}/athletes${queryString}`,
      {
        headers: { 'Authorization': `Bearer ${this.token}` }
      }
    );
    
    return await response.json();
  }

  async getScan(orgId, scanId) {
    if (!this.token) await this.authenticate();
    
    const response = await fetch(
      `${this.baseUrl}/client/v1/orgs/${orgId}/scans/${scanId}`,
      {
        headers: { 'Authorization': `Bearer ${this.token}` }
      }
    );
    
    return await response.json();
  }

  async listScans(orgId, options = {}) {
    if (!this.token) await this.authenticate();
    
    const params = new URLSearchParams();
    if (options.limit) params.append('limit', options.limit);
    if (options.offset) params.append('offset', options.offset);
    if (options.athlete_id) params.append('athlete_id', options.athlete_id);
    if (options.status) params.append('status', options.status);
    if (options.start_date) params.append('start_date', options.start_date);
    if (options.end_date) params.append('end_date', options.end_date);
    if (options.sort) params.append('sort', options.sort);
    if (options.order) params.append('order', options.order);
    
    const queryString = params.toString() ? `?${params.toString()}` : '';
    const response = await fetch(
      `${this.baseUrl}/client/v1/orgs/${orgId}/scans${queryString}`,
      {
        headers: { 'Authorization': `Bearer ${this.token}` }
      }
    );
    
    return await response.json();
  }
}

// Usage
const api = new ReflectAPI('your_client_id', 'your_client_secret', 'prod-wl');

// List all athletes
const athletesList = await api.listAthletes('ABC', { limit: 100, sort: 'last_name', order: 'asc' });

// Get specific athlete
const athlete = await api.getAthlete('ABC', 'athlete-id');

// List all scans
const scansList = await api.listScans('ABC', { limit: 50, status: 'completed' });

// List scans for specific athlete
const athleteScans = await api.listScans('ABC', { athlete_id: 'athlete-id', limit: 20 });

// Get specific scan
const scan = await api.getScan('ABC', 'scan-id');
// scan.scan includes operatorName and customId fields
console.log(scan.scan.operatorName); // "Dr. Smith"
console.log(scan.scan.customId);     // "SCAN-2025-0042"
```

### Python

```python
import requests
from datetime import datetime, timedelta

class ReflectAPI:
    def __init__(self, client_id, client_secret, environment='prod-wl'):
        self.client_id = client_id
        self.client_secret = client_secret
        self.base_url = f'https://{environment}.fitmatch.io'
        self.token = None
        self.token_expiry = None
    
    def authenticate(self):
        """Get access token"""
        response = requests.post(
            f'{self.base_url}/reflect-auth/client-credentials',
            json={
                'client_id': self.client_id,
                'client_secret': self.client_secret,
                'grant_type': 'client_credentials'
            }
        )
        response.raise_for_status()
        
        data = response.json()
        self.token = data['access_token']
        self.token_expiry = datetime.now() + timedelta(seconds=data['expires_in'])
        
        return self.token
    
    def _get_headers(self):
        """Get authorization headers with valid token"""
        if not self.token or datetime.now() >= self.token_expiry:
            self.authenticate()
        
        return {'Authorization': f'Bearer {self.token}'}
    
    def get_athlete(self, org_id, athlete_id):
        """Get athlete details"""
        response = requests.get(
            f'{self.base_url}/client/v1/orgs/{org_id}/athletes/{athlete_id}',
            headers=self._get_headers()
        )
        response.raise_for_status()
        return response.json()
    
    def list_athletes(self, org_id, limit=50, offset=0, sort='created_at', order='desc'):
        """List all athletes with pagination"""
        params = {
            'limit': limit,
            'offset': offset,
            'sort': sort,
            'order': order
        }
        response = requests.get(
            f'{self.base_url}/client/v1/orgs/{org_id}/athletes',
            headers=self._get_headers(),
            params=params
        )
        response.raise_for_status()
        return response.json()
    
    def get_scan(self, org_id, scan_id):
        """Get scan results"""
        response = requests.get(
            f'{self.base_url}/client/v1/orgs/{org_id}/scans/{scan_id}',
            headers=self._get_headers()
        )
        response.raise_for_status()
        return response.json()
    
    def list_scans(self, org_id, limit=50, offset=0, athlete_id=None, status=None, 
                   start_date=None, end_date=None, sort='created_at', order='desc'):
        """List all scans with pagination and filtering"""
        params = {
            'limit': limit,
            'offset': offset,
            'sort': sort,
            'order': order
        }
        if athlete_id:
            params['athlete_id'] = athlete_id
        if status:
            params['status'] = status
        if start_date:
            params['start_date'] = start_date
        if end_date:
            params['end_date'] = end_date
            
        response = requests.get(
            f'{self.base_url}/client/v1/orgs/{org_id}/scans',
            headers=self._get_headers(),
            params=params
        )
        response.raise_for_status()
        return response.json()
    
    def export_data(self, org_id, data_type='scans', format='json', filters=None):
        """Export bulk data"""
        payload = {
            'data_type': data_type,
            'format': format
        }
        if filters:
            payload['filters'] = filters
        
        response = requests.post(
            f'{self.base_url}/client/v1/orgs/{org_id}/export',
            headers=self._get_headers(),
            json=payload
        )
        response.raise_for_status()
        return response.json()

# Usage
api = ReflectAPI('your_client_id', 'your_client_secret', 'prod-wl')

# List all athletes
athletes_list = api.list_athletes('ABC', limit=100, sort='last_name', order='asc')

# Get specific athlete
athlete = api.get_athlete('ABC', 'athlete-id')

# List all scans
scans_list = api.list_scans('ABC', limit=50, status='completed')

# List scans for specific athlete
athlete_scans = api.list_scans('ABC', athlete_id='athlete-id', limit=20)

# Get specific scan
scan = api.get_scan('ABC', 'scan-id')
# scan['scan'] includes operatorName and customId fields
print(scan['scan']['operatorName'])  # "Dr. Smith"
print(scan['scan']['customId'])      # "SCAN-2025-0042"

# Export all scans from 2025
export_data = api.export_data(
    'ABC',
    data_type='scans',
    filters={
        'date_range': {
            'start': '2025-01-01',
            'end': '2025-12-31'
        }
    }
)
# export_data is the raw array of scans
```

### cURL

```bash
# Get access token
TOKEN=$(curl -X POST https://prod-wl.fitmatch.io/reflect-auth/client-credentials \
  -H "Content-Type: application/json" \
  -d '{
    "client_id": "your_client_id",
    "client_secret": "your_client_secret",
    "grant_type": "client_credentials"
  }' | jq -r '.access_token')

# List all athletes
curl -X GET "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/athletes?limit=100&sort=last_name&order=asc" \
  -H "Authorization: Bearer $TOKEN"

# Get specific athlete
curl -X GET https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/athletes/athlete-id \
  -H "Authorization: Bearer $TOKEN"

# List all scans
curl -X GET "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/scans?limit=50&status=completed" \
  -H "Authorization: Bearer $TOKEN"

# List scans for specific athlete
curl -X GET "https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/scans?athlete_id=athlete-id&limit=20" \
  -H "Authorization: Bearer $TOKEN"

# Get specific scan (response includes operatorName and customId)
curl -X GET https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/scans/scan-id \
  -H "Authorization: Bearer $TOKEN"

# Export data
curl -X POST https://prod-wl.fitmatch.io/reflect/client/v1/orgs/ABC/export \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "data_type": "scans",
    "format": "json",
    "filters": {
      "date_range": {
        "start": "2025-01-01",
        "end": "2025-12-31"
      }
    }
  }'
```

---

## Data Models

### Athlete

```typescript
{
  athleteId: string;           // Unique identifier (caller-supplied UUID or auto-generated)
  firstName: string;           // Athlete first name
  lastName: string;            // Athlete last name
  position: string | null;     // Playing position
  birthday: string | null;     // Date of birth (ISO 8601)
  teamId: string;              // Team identifier (abbreviation)
  teamName: string;            // Team display name
  notes: string | null;        // Additional notes
  biometrics: {                // Custom biometric measurements
    height: number;
    weight: number;
    wingspan: number;
    // Additional custom fields
  };
  athleteEmail: string | null; // Contact email
  isTestAthlete: boolean;      // Whether this is a test athlete
  organization: string | null; // Organization abbreviation
  organizationName: string | null; // Organization display name
  lastScannedDate: string | null;  // Most recent scan date
  createdAt: string;           // Account creation date
}
```

### Scan

```typescript
{
  scanId: string;              // Unique identifier
  athleteId: string;           // Associated athlete UUID
  teamId: string;              // Organization identifier
  scanStatus: string;          // "pending" | "processing" | "completed" | "error"
  operatorName: string | null; // Name of the person who performed the scan
  customId: string | null;     // Your custom identifier for this scan (e.g., internal tracking ID)
  measurements: {              // Body measurements (see below)
    height: {...},
    torso: {...},
    arm: {...},
    legs: {...},
    ratios: {...},
    physique_metrics: {...}
  };
  processingTime: number | null;  // Processing duration (seconds)
  scanFileUrl: string | null;  // URL to the scan file in S3
  isTestScan: boolean;         // Whether this is a test scan
  timestamp: string;           // Scan capture time (ISO 8601)
}
```

### Measurements

Body measurements are organized by anatomical region:

```typescript
{
  height: {
    body_height: number; 
    seated_height: number;
  },
  torso: {
    chest_circumference: number;
    waist_circumference: number;
    hip_circumference: number;
    shoulder_width: number;
    // Additional torso measurements
  },
  arm: {
    upper_arm_circumference: number;
    forearm_circumference: number;
    arm_length: number;
    // Additional arm measurements
  },
  legs: {
    upper_thigh_circumference_left: number;
    upper_thigh_circumference_right: number;
    calf_circumference_left: number;
    calf_circumference_right: number;
    leg_length: number;
    // Additional leg measurements
  },
  ratios: {
    shoulder_to_hip_ratio: number;
    waist_to_hip_ratio: number;
    // Additional body ratios
  },
  physique_metrics: {
    bmi: number;               // Body Mass Index
    bri: number;               // Body Roundness Index
    // Additional metrics
  }
}
```

---

## Error Handling

### HTTP Status Codes

- `200` - Success
- `400` - Bad Request (invalid parameters)
- `401` - Unauthorized (missing or invalid token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found (resource doesn't exist)
- `429` - Too Many Requests (rate limit exceeded)
- `500` - Internal Server Error

### Error Response Format

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error description",
    "timestamp": "2025-12-04T03:30:00Z"
  }
}
```

### Common Errors

**Invalid Token:**
```json
{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Token validation failed",
    "timestamp": "2025-12-04T03:30:00Z"
  }
}
```

**Resource Not Found:**
```json
{
  "error": {
    "code": "SCAN_NOT_FOUND",
    "message": "Scan not found",
    "timestamp": "2025-12-04T03:30:00Z"
  }
}
```

**Rate Limit Exceeded:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Please try again later.",
    "timestamp": "2025-12-04T03:30:00Z"
  }
}
```

---

## Rate Limits

API usage is limited based on your subscription tier:

| Tier | Requests/Minute | Requests/Hour | Data Export/Day |
|------|----------------|---------------|-----------------|
| Basic | 100 | 5,000 | 100 MB |
| Premium | 500 | 25,000 | 1 GB |
| Enterprise | 1,000 | 50,000 | 5 GB |

Rate limit information is included in response headers:

```
X-RateLimit-Limit: 500
X-RateLimit-Remaining: 495
X-RateLimit-Reset: 1638360000
```

When rate limits are exceeded, implement exponential backoff:

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    const response = await fetch(url, options);
    
    if (response.status !== 429) {
      return response;
    }
    
    // Exponential backoff: 1s, 2s, 4s
    const delay = Math.pow(2, i) * 1000;
    await new Promise(resolve => setTimeout(resolve, delay));
  }
  
  throw new Error('Max retries exceeded');
}
```

---

## Security

### Best Practices

**Credential Management:**
- Store credentials in environment variables or secrets manager
- Never commit credentials to version control
- Never expose credentials in client-side code
- Use separate credentials for development and production

**Token Management:**
- Cache tokens for their full lifetime (1 hour)
- Implement automatic token refresh
- Handle 401 errors by refreshing and retrying

**Network Security:**
- Always use HTTPS
- Verify SSL certificates in production
- Implement request timeouts
- Log errors without exposing sensitive data

**Access Control:**
- Tokens are scoped to your organization
- You can only access your own data
- All API access is logged for audit purposes

---

