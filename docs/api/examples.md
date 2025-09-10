# API Examples

## Getting Started

### Authentication Setup
```bash
# Set your API key as an environment variable
export NASA_AI_API_KEY="your-api-key-here"
```

## Basic Examples

### Example 1: Fetch Mars Photos
```bash
# Get recent Mars rover photos
curl -X GET "https://api.nasaai.app/v1/nasa/data?dataset=mars-photos&limit=5" \
  -H "Authorization: Bearer $NASA_AI_API_KEY" \
  -H "Content-Type: application/json"
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "results": [
      {
        "id": "mars_photo_001",
        "url": "https://mars.nasa.gov/msl-raw-images/proj/msl/redops/ods/surface/sol/01000/opgs/edr/fcam/FLB_486265257EDR_F0481570FHAZ00323M_.JPG",
        "sol": 1000,
        "camera": "FHAZ",
        "earth_date": "2015-05-30",
        "rover": "Curiosity"
      }
    ]
  }
}
```

### Example 2: Submit Image for Analysis
```bash
# Submit an image for AI analysis
curl -X POST "https://api.nasaai.app/v1/nasa/analysis" \
  -H "Authorization: Bearer $NASA_AI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "data_type": "image",
    "data_url": "https://example.com/mars-surface.jpg",
    "analysis_type": "terrain_classification"
  }'
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "analysis_id": "analysis_abc123",
    "status": "processing",
    "estimated_completion": "2024-01-01T12:05:00Z"
  },
  "message": "Analysis request submitted successfully"
}
```

### Example 3: Check Analysis Results
```bash
# Check the status and results of your analysis
curl -X GET "https://api.nasaai.app/v1/analysis/analysis_abc123" \
  -H "Authorization: Bearer $NASA_AI_API_KEY"
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "analysis_id": "analysis_abc123",
    "status": "completed",
    "results": {
      "terrain_type": "rocky_surface",
      "confidence": 0.92,
      "features_detected": ["rocks", "sand", "shadows"],
      "geological_composition": {
        "basalt": 0.65,
        "sand": 0.30,
        "unknown": 0.05
      }
    },
    "processing_time": 45.2,
    "completed_at": "2024-01-01T12:05:00Z"
  }
}
```

## Language-Specific Examples

### Python Examples

#### Basic Setup
```python
import requests
import os

class NASAClient:
    def __init__(self, api_key=None):
        self.api_key = api_key or os.getenv('NASA_AI_API_KEY')
        self.base_url = 'https://api.nasaai.app/v1'
        self.headers = {
            'Authorization': f'Bearer {self.api_key}',
            'Content-Type': 'application/json'
        }
    
    def get_mars_photos(self, sol=None, camera=None, limit=20):
        params = {'dataset': 'mars-photos', 'limit': limit}
        if sol:
            params['sol'] = sol
        if camera:
            params['camera'] = camera
            
        response = requests.get(
            f'{self.base_url}/nasa/data',
            headers=self.headers,
            params=params
        )
        return response.json()
    
    def submit_analysis(self, data_url, data_type='image', analysis_type='classification'):
        payload = {
            'data_type': data_type,
            'data_url': data_url,
            'analysis_type': analysis_type
        }
        response = requests.post(
            f'{self.base_url}/nasa/analysis',
            headers=self.headers,
            json=payload
        )
        return response.json()
```

#### Usage Example
```python
# Initialize client
client = NASAClient()

# Get Mars photos from Sol 1000
photos = client.get_mars_photos(sol=1000, camera='FHAZ', limit=10)
print(f"Found {len(photos['data']['results'])} photos")

# Submit image for analysis
analysis = client.submit_analysis(
    data_url='https://example.com/mars-image.jpg',
    analysis_type='terrain_classification'
)
print(f"Analysis ID: {analysis['data']['analysis_id']}")
```

### JavaScript Examples

#### Node.js Setup
```javascript
const axios = require('axios');

class NASAClient {
    constructor(apiKey = null) {
        this.apiKey = apiKey || process.env.NASA_AI_API_KEY;
        this.baseURL = 'https://api.nasaai.app/v1';
        this.headers = {
            'Authorization': `Bearer ${this.apiKey}`,
            'Content-Type': 'application/json'
        };
    }
    
    async getMarsPhotos(options = {}) {
        const params = {
            dataset: 'mars-photos',
            limit: options.limit || 20,
            ...options
        };
        
        try {
            const response = await axios.get(`${this.baseURL}/nasa/data`, {
                headers: this.headers,
                params
            });
            return response.data;
        } catch (error) {
            console.error('Error fetching Mars photos:', error.response?.data || error.message);
            throw error;
        }
    }
    
    async submitAnalysis(dataUrl, options = {}) {
        const payload = {
            data_type: options.dataType || 'image',
            data_url: dataUrl,
            analysis_type: options.analysisType || 'classification'
        };
        
        try {
            const response = await axios.post(`${this.baseURL}/nasa/analysis`, payload, {
                headers: this.headers
            });
            return response.data;
        } catch (error) {
            console.error('Error submitting analysis:', error.response?.data || error.message);
            throw error;
        }
    }
}
```

#### Usage Example
```javascript
// Initialize client
const client = new NASAClient();

// Get Mars photos
async function fetchMarsPhotos() {
    try {
        const photos = await client.getMarsPhotos({
            sol: 1000,
            camera: 'FHAZ',
            limit: 10
        });
        console.log(`Found ${photos.data.results.length} photos`);
        return photos;
    } catch (error) {
        console.error('Failed to fetch photos:', error);
    }
}

// Submit analysis
async function analyzeImage() {
    try {
        const analysis = await client.submitAnalysis(
            'https://example.com/mars-image.jpg',
            { analysisType: 'terrain_classification' }
        );
        console.log(`Analysis ID: ${analysis.data.analysis_id}`);
        return analysis;
    } catch (error) {
        console.error('Failed to submit analysis:', error);
    }
}
```

## Advanced Examples

### Batch Processing
```python
import asyncio
import aiohttp

async def process_multiple_images(image_urls):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in image_urls:
            task = submit_analysis_async(session, url)
            tasks.append(task)
        
        results = await asyncio.gather(*tasks)
        return results

async def submit_analysis_async(session, image_url):
    payload = {
        'data_type': 'image',
        'data_url': image_url,
        'analysis_type': 'terrain_classification'
    }
    
    async with session.post(
        'https://api.nasaai.app/v1/nasa/analysis',
        json=payload,
        headers={'Authorization': f'Bearer {os.getenv("NASA_AI_API_KEY")}'}
    ) as response:
        return await response.json()
```

### Error Handling
```python
def robust_api_call(client, operation, **kwargs):
    max_retries = 3
    retry_delay = 1
    
    for attempt in range(max_retries):
        try:
            return operation(**kwargs)
        except requests.exceptions.RequestException as e:
            if attempt == max_retries - 1:
                raise e
            
            if hasattr(e, 'response') and e.response.status_code == 429:
                # Rate limited - wait longer
                time.sleep(retry_delay * (2 ** attempt))
            else:
                time.sleep(retry_delay)
```

---
*Last updated: [Date]*