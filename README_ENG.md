# Materiality Level Calculation API

## Table of Contents
1. [Overview](#overview)
2. [Technical Requirements](#technical-requirements)
3. [Installation & Running](#installation--running)
4. [API Endpoints](#api-endpoints)
   - [POST /api/v1/calculate](#post-apiv1calculate-1)
   - [GET /api/v1/generate-form](#get-apiv1generate-form-1)
   - [GET /form/{session_id}](#get-formsession_id-1)
5. [Data Models](#data-models)
6. [Usage Examples](#usage-examples)
7. [Business Logic](#business-logic)
8. [Deployment](#deployment)
9. [Testing](#testing)
10. [License](#license)

## Overview
This API calculates materiality level based on financial indicators. Key features:
- Materiality level calculation from financial indicators
- Outlier detection and exclusion
- Word report generation
- Web interface for data input

## Technical Requirements
- Python 3.7+
- Dependencies (see requirements.txt)
- FastAPI framework
- Jinja2 templating
- python-docx (Word document generation)
- numpy (mathematical computations)

## Installation & Running

1. Clone repository:
```bash
git clone https://github.com/your-repo/materiality-api.git
cd materiality-api
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Run server:
```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

4. For production:
```bash
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
```

## API Endpoints

### POST /api/v1/calculate
Main endpoint for materiality calculation.

**Request:**
```json
{
  "indicators": [
    {
      "name": "string",
      "value": 0
    }
  ],
  "deviation_threshold": 50,
  "rounding_limit": 50,
  "with_docx": false
}
```

**Response:**
```json
{
  "materiality_level": 0,
  "calculation_steps": {
    "initial_mean": 0,
    "filtered_mean": 0,
    "excluded_count": 0,
    "excluded_values": [],
    "indicators": [
      {
        "name": "string",
        "value": 0,
        "deviation": {
          "absolute": 0,
          "percent": 0
        }
      }
    ],
    "rounded_value": 0
  },
  "indicators": [
    {
      "name": "string",
      "value": 0
    }
  ],
  "message": "string"
}
```

### GET /api/v1/generate-form
Generates unique session for web form access.

**Response:**
```json
{
  "form_url": "string",
  "expires_at": 0
}
```

### GET /form/{session_id}
Returns HTML form for materiality calculation.

## Data Models

### Indicator
```python
class Indicator(BaseModel):
    name: str = Field(..., example="Revenue", description="Financial indicator name")
    value: float = Field(..., example=1800000, description="Value in rubles")
```

### CalculationRequest
```python
class CalculationRequest(BaseModel):
    indicators: List[Indicator]
    deviation_threshold: float = 50
    rounding_limit: float = 50
    with_docx: bool = False
```

### CalculationResponse
```python
class CalculationResponse(BaseModel):
    materiality_level: float
    calculation_steps: CalculationSteps
    indicators: List[Indicator]
    message: str = "Calculation successful"
```

## Usage Examples

### API Request Example
```bash
curl -X POST "http://localhost:8000/api/v1/calculate" \
-H "Content-Type: application/json" \
-d '{
  "indicators": [
    {"name": "Revenue", "value": 1800000},
    {"name": "Profit", "value": 480000}
  ],
  "deviation_threshold": 50,
  "rounding_limit": 50,
  "with_docx": false
}'
```

### Response Example
```json
{
  "materiality_level": 857100.0,
  "calculation_steps": {
    "initial_mean": 857142.8571428571,
    "filtered_mean": 857142.8571428571,
    "excluded_count": 0,
    "excluded_values": [],
    "indicators": [
      {
        "name": "Revenue",
        "value": 1800000.0,
        "deviation": {
          "absolute": 942857.1428571428,
          "percent": 110.0
        }
      },
      {
        "name": "Profit",
        "value": 480000.0,
        "deviation": {
          "absolute": -377142.8571428571,
          "percent": -44.0
        }
      }
    ],
    "rounded_value": 857100.0
  },
  "indicators": [
    {"name": "Revenue", "value": 1800000.0},
    {"name": "Profit", "value": 480000.0}
  ],
  "message": "Calculation successful"
}
```

## Business Logic

1. **Mean Calculation**:
   - Compute arithmetic mean of all indicators

2. **Deviation Analysis**:
   - Calculate percentage deviation for each indicator
   - Exclude indicators with deviation > threshold

3. **Recalculation**:
   - Compute new mean from remaining indicators

4. **Rounding**:
   - Round result to nearest 100 rubles
   - Skip rounding if deviation > limit

5. **Reporting**:
   - Generate JSON with calculation details
   - Optionally create Word report

## Deployment

### Docker
```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY . .

RUN pip install -r requirements.txt

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Kubernetes
Example deployment.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: materiality-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: materiality-api
  template:
    metadata:
      labels:
        app: materiality-api
    spec:
      containers:
      - name: materiality-api
        image: your-repo/materiality-api:latest
        ports:
        - containerPort: 8000
        resources:
          limits:
            memory: "256Mi"
            cpu: "500m"
```

## Testing

Run tests:
```bash
pytest tests/
```

Example test:
```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_calculate_materiality():
    response = client.post(
        "/api/v1/calculate",
        json={
            "indicators": [
                {"name": "Test1", "value": 1000},
                {"name": "Test2", "value": 2000}
            ],
            "deviation_threshold": 50,
            "rounding_limit": 50
        }
    )
    assert response.status_code == 200
    assert "materiality_level" in response.json()
```

## License

Copyright (c) 2025 CARL_TECH
