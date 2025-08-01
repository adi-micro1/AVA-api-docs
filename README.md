# AVA Proctor API Documentation

## Overview

The AI Proctor API is a machine learning service that analyzes interview recordings and documents to detect potential cheating behavior. It processes multiple modalities including video, audio, and text to generate a cheating probability score.

## Endpoint

**URL:** `https://model-7qk66ye3.api.baseten.co/environments/production/predict`  
**Method:** POST  
**Content-Type:** application/json

## Authentication

Include the API key in the Authorization header:
```
Authorization: Api-Key <your_baseten_api_key>
```

## Request Format

```json
{
  "report_url": "link/to/candidate/report.pdf",
  "resume_url": "link/to/candidate/resume.pdf",
  "webcam_url": "link/to/candidate/webcam_video.mp4(or.m3u8)",
  "screenshare_url": "link/to/candidate/screenshare_video.mp4(or.m3u8)"
}
```

## Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `report_url` | string | URL to the interview report PDF |
| `resume_url` | string | URL to the candidate's resume PDF |
| `webcam_url` | string | URL to the webcam recording video |
| `screenshare_url` | string | URL to the screen sharing recording video | 

## Successful Response Format

```json
{
  "request_id": "req_abc12345",
  "status": "success",
  "cheating_probability": 0.2345,
  "embedding_dimensions": {
    "screen_video": 768,
    "webcam_video": 768,
    "resume_text": 768,
    "report_text": 768,
    "audio": 768,
    "total": 3840
  }
}
```

### Error Response

```json
{
  "request_id": "req_abc12345",
  "status": "error",
  "error": "Missing required field: report_url"
}
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `request_id` | string | Unique identifier for the request |
| `status` | string | Either "success" or "error" |
| `cheating_probability` | float | Probability score between 0 and 1 indicating likelihood of cheating |
| `embedding_dimensions` | object | Dimensions of the feature vectors used for analysis |
| `error` | string | Error message (only present when status is "error") |

## Usage Examples

### Python Example (Single Request)

```python
import requests

# API configuration
url = "https://model-7qk66ye3.api.baseten.co/environments/production/predict"
headers = {
    "Authorization": "Api-Key t66JsdoW.KgHE2QjSuLw3UIGrhN007PJohA8e9yiU"
}

# Request data
data = {
  "report_url": "link/to/candidate/report.pdf",
  "resume_url": "link/to/candidate/resume.pdf",
  "webcam_url": "link/to/candidate/webcam_video.mp4(or.m3u8)",
  "screenshare_url": "link/to/candidate/screenshare_video.mp4(or.m3u8)"
}

# Make request
response = requests.post(url, headers=headers, json=data, timeout=None)
result = response.json()

# Extract results
if result["status"] == "success":
    cheating_prob = result["cheating_probability"]
    print(f"Cheating probability: {cheating_prob:.4f}")
    
    # Access embedding dimensions
    dims = result["embedding_dimensions"]
    print(f"Total embedding dimensions: {dims['total']}")
else:
    print(f"Error: {result['error']}")
```


### cURL Example

```bash
curl -X POST \
  "https://model-7qk66ye3.api.baseten.co/environments/production/predict" \
  -H "Authorization: Api-Key <your_baseten_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "report_url": "link/to/candidate/report.pdf",
    "resume_url": "link/to/candidate/resume.pdf",
    "webcam_url": "link/to/candidate/webcam_video.mp4(or.m3u8)",
    "screenshare_url": "link/to/candidate/screenshare_video.mp4(or.m3u8)"
}'
```

## Processing Details

The API processes the following modalities:

1. **Screen Video** (768 dimensions): Analyzes screen sharing recordings for suspicious activity
2. **Webcam Video** (768 dimensions): Analyzes webcam recordings for behavioral patterns
3. **Resume Text** (768 dimensions): Extracts and analyzes text from candidate resumes
4. **Report Text** (768 dimensions): Analyzes interview report content
5. **Audio** (768 dimensions): Processes audio from webcam recordings

The final feature vector combines all modalities (3840 total dimensions) and feeds into a machine learning classifier to produce the cheating probability score.

## Error Handling

Common error scenarios:

- **Missing required fields**: Ensure all four URL fields are provided
- **Invalid URLs**: URLs must be accessible and contain valid file formats
- **Processing errors**: Large files or network issues may cause processing failures

## Rate Limits and Performance

- **Requests**: Processed sequentially
- **Timeout**: Set to `None` to allow for long processing times
- **Memory management**: GPU memory is cleared between modality processing

## File Format Requirements

- **PDFs**: Interview reports and resumes must be in PDF format
- **Videos**: MP4 format recommended for video recordings while it also supports .m3u8
- **URLs**: Must be publicly accessible or have appropriate authentication
- **File sizes**: Large files may increase processing time 
