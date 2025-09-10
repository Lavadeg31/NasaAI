# API Endpoints

## API Overview

### Base URL
```
https://api.nasaai.app/v1
```

### Authentication
<!-- TODO: Document authentication method -->
```
Authorization: Bearer [your-api-key]
```

### Response Format
All API responses follow this format:
```json
{
  "status": "success|error",
  "data": {...},
  "message": "Human-readable message",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

## Endpoints

### NASA Data Endpoints

#### GET /nasa/data
Retrieve NASA dataset information.

**Parameters:**
- `dataset` (string, required): Dataset identifier
- `limit` (integer, optional): Number of results (max 100, default 20)
- `offset` (integer, optional): Offset for pagination (default 0)

**Request Example:**
```bash
curl -X GET "https://api.nasaai.app/v1/nasa/data?dataset=mars-photos&limit=10" \
  -H "Authorization: Bearer your-api-key"
```

**Response Example:**
```json
{
  "status": "success",
  "data": {
    "results": [
      {
        "id": "mars_photo_001",
        "url": "https://mars.nasa.gov/photo1.jpg",
        "sol": 1000,
        "camera": "FHAZ",
        "earth_date": "2015-05-30"
      }
    ],
    "total": 1000,
    "limit": 10,
    "offset": 0
  },
  "message": "Mars photos retrieved successfully",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

#### POST /nasa/analysis
Submit data for NASA AI analysis.

**Request Body:**
```json
{
  "data_type": "image|telemetry|text",
  "data_url": "https://example.com/data.jpg",
  "analysis_type": "classification|detection|prediction"
}
```

**Response Example:**
```json
{
  "status": "success",
  "data": {
    "analysis_id": "analysis_123",
    "status": "processing",
    "estimated_completion": "2024-01-01T12:05:00Z"
  },
  "message": "Analysis request submitted successfully",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

### Analysis Endpoints

#### GET /analysis/{analysis_id}
Get analysis results by ID.

**Parameters:**
- `analysis_id` (string, required): Analysis identifier

**Response Example:**
```json
{
  "status": "success",
  "data": {
    "analysis_id": "analysis_123",
    "status": "completed",
    "results": {
      "confidence": 0.95,
      "classification": "Mars rover",
      "features": ["wheels", "antenna", "solar panels"]
    },
    "processing_time": 45.2,
    "completed_at": "2024-01-01T12:05:00Z"
  },
  "message": "Analysis completed successfully",
  "timestamp": "2024-01-01T12:05:30Z"
}
```

### User Management Endpoints

#### GET /user/profile
Get user profile information.

**Response Example:**
```json
{
  "status": "success",
  "data": {
    "user_id": "user_123",
    "username": "astronaut_jane",
    "email": "jane@nasa.gov",
    "api_quota": {
      "used": 150,
      "limit": 1000,
      "resets_at": "2024-02-01T00:00:00Z"
    }
  },
  "message": "Profile retrieved successfully",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

## Error Responses

### Error Format
```json
{
  "status": "error",
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {...}
  },
  "timestamp": "2024-01-01T12:00:00Z"
}
```

### Common Error Codes
- `INVALID_API_KEY`: Invalid or missing API key
- `QUOTA_EXCEEDED`: API quota exceeded
- `INVALID_PARAMETERS`: Invalid request parameters
- `RESOURCE_NOT_FOUND`: Requested resource not found
- `INTERNAL_ERROR`: Internal server error

## Rate Limiting

### Limits
- **Free Tier:** 100 requests/hour
- **Pro Tier:** 1,000 requests/hour
- **Enterprise:** Custom limits

### Headers
Rate limit information is included in response headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

## SDKs and Libraries

### Python SDK
```python
from nasaai import NASAClient

client = NASAClient(api_key='your-api-key')
photos = client.get_mars_photos(sol=1000, camera='FHAZ')
```

### JavaScript SDK
```javascript
import { NASAClient } from 'nasaai-js';

const client = new NASAClient('your-api-key');
const photos = await client.getMarsPhotos({ sol: 1000, camera: 'FHAZ' });
```

---
*Last updated: [Date]*