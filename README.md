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
  "request_id":"",
  "callback_url":"link/to/your/callback"
  "interview_transcript": [JSON],
  "resume_url": "link/to/candidate/resume.pdf",
  "webcam_url": "link/to/candidate/webcam_video.mp4(or.m3u8)",
  "screenshare_url": "link/to/candidate/screenshare_video.mp4(or.m3u8)"
}
```

## Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `request_id` | string | Your request identifier |
| `callback_url` | string | A callback url to send your responses to |
| `interview_transcript` | [JSON] | A list of JSON |
| `screenshare_url` | string | URL to the screen sharing recording video | 

## Successful Response Format

```json
{
  "request_id": "req_abc12345",
  "status": "success",
  "cheating_probability": 0.05345,
  "embedding_dimensions": {
    "screen_video": 768,
    "webcam_video": 768,
    "resume_text": 768,
    "report_text": 768,
    "audio": 768,
    "total": 3840
    },
    "embedding_sources": {
        "screen_video": "processed",
        "interview_transcript": "processed",
        "webcam_video": "processed" if webcam_url else "replaced_with_screen",
        "resume_text": "processed" if resume_url else "replaced_with_transcript", 
        "audio": "processed" if webcam_url else "replaced_with_screenshare"
    },
    "optional_fields_used": {
        "webcam_url": webcam_url is not None,
        "resume_url": resume_url is not None
    }
}
```
> [!NOTE]  
> If either webcam url or resume aren't provided by the user, their embeddings are replaced with screen video and interview transcript embeddings respectively.
> And the audio embeddings are extracted from the screenshare video instead of webcam video.

### Error Response

```json
{
  "request_id": "req_abc12345",
  "status": "error",
  "error": "Missing required field: interview_transcript"
}
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `request_id` | string | Unique identifier for the request |
| `status` | string | Either "completed" or "error" |
| `cheating_probability` | float | Probability score between 0 and 1 indicating likelihood of cheating |
| `embedding_dimensions` | object | Dimensions of the feature vectors used for analysis |
| `error` | string | Error message (only present when status is "error") |
|`embedding_sources` | dict | Status of the attributes that have been processed and replaced |
|`optional_fields_used` | dict | Status of the optional attributes |

## Usage Examples

### Python Example (Single Request)

```python
import requests

# API configuration
url = "https://model-7qk66ye3.api.baseten.co/environments/production/predict"
headers = {
    "Authorization": "Api-Key <YOUR_BASETEN_API_KEY>"
}

# Request data
data = {
  "request_id": str
  "callback_url":"link/to/callback"
  "interview_transcript": [JSON],
  "resume_url": "link/to/candidate/resume.pdf",
  "webcam_url": "link/to/candidate/webcam_video.mp4(or.m3u8)",
  "screenshare_url": "link/to/candidate/screenshare_video.mp4(or.m3u8)"
}

# Make request
response = requests.post(url, headers=headers, json=data, timeout=None)
result = response.json()

# Extract results
if result["status"] == "completed":
    cheating_prob = result["cheating_probability"]
    print(f"Cheating probability: {cheating_prob:.4f}")
    
    # Access embedding dimensions
    dims = result["embedding_dimensions"]
    print(f"Total embedding dimensions: {dims['total']}")
else:
    print(f"Error: {result['error']}")
```


### Postman cURL Example

```bash
curl --location 'https://model-7qk66ye3.api.baseten.co/environments/production/predict' \
--header 'Content-Type: application/json' \
--header 'Authorization: Api-Key <YOUR_BASETEN_API_KEY>' \
--header 'Content-Type: Content-Type' \
--data '{
    "request_id": "",
    "callback_url": "",
    "interview_transcript": [],
    "resume_url": "",
    "webcam_url": "",
    "screenshare_url": ""
}'
```

## Processing Details

The API processes the following modalities:

1. **Screen Video** (768 dimensions): Analyzes screen sharing recordings for suspicious activity
2. **Webcam Video** (768 dimensions): Analyzes webcam recordings for behavioral patterns
3. **Resume Text** (768 dimensions): Extracts and analyzes text from candidate resumes
4. **Interview Transcript** (768 dimensions): Generates an interview report content based on transcript
5. **Audio** (768 dimensions): Processes audio from webcam recordings

The final feature vector combines all modalities (3840 total dimensions) and feeds into a machine learning classifier to produce the cheating probability score.

## Error Handling

Common error scenarios:

- **Missing required fields**: Ensure all four URL fields are provided
- **Invalid URLs**: URLs must be accessible and contain valid file formats

## Rate Limits and Performance

- **Requests**: Processed sequentially
- **Memory management**: GPU memory is cleared between modality processing

## File Format Requirements

- **PDFs**: Interview reports and resumes must be in PDF format
- **Videos**: MP4 format recommended for video recordings while it also supports .m3u8
- **URLs**: Must be publicly accessible or have appropriate authentication
- **JSON**: Especially to process interview transcripts
- **File sizes**: Large files may increase processing time 
