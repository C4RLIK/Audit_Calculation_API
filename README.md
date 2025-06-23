# Audit_Calculation_API
# Документация для API расчёта уровня существенности

## Оглавление
1. [Общее описание](#общее-описание)
2. [Технические требования](#технические-требования)
3. [Установка и запуск](#установка-и-запуск)
4. [API Endpoints](#api-endpoints)
   - [POST /api/v1/calculate](#post-apiv1calculate)
   - [GET /api/v1/generate-form](#get-apiv1generate-form)
   - [GET /form/{session_id}](#get-formsession_id)
5. [Модели данных](#модели-данных)
6. [Примеры использования](#примеры-использования)
7. [Логика работы](#логика-работы)
8. [Развёртывание](#развёртывание)
9. [Тестирование](#тестирование)
10. [Лицензия](#лицензия)

## Общее описание
API предназначено для автоматического расчёта уровня существенности на основе финансовых показателей. Сервис предоставляет:
- Расчёт уровня существенности по заданным показателям
- Исключение нерепрезентативных показателей (выбросов)
- Генерацию отчёта в формате Word
- Web-интерфейс для удобного ввода данных

## Технические требования
- Python 3.7+
- Установленные зависимости (см. requirements.txt)
- FastAPI (веб-фреймворк)
- Jinja2 (шаблонизатор)
- python-docx (генерация Word-документов)
- numpy (математические вычисления)

## Установка и запуск

1. Клонировать репозиторий:
```bash
git clone https://github.com/your-repo/materiality-api.git
cd materiality-api
```

2. Установить зависимости:
```bash
pip install -r requirements.txt
```

3. Запустить сервер:
```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

4. Для production рекомендуется использовать:
```bash
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
```

## API Endpoints

### POST /api/v1/calculate
Основной endpoint для расчёта уровня существенности.

**Параметры запроса:**
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

**Ответ:**
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
Генерирует уникальную сессию для доступа к веб-форме.

**Ответ:**
```json
{
  "form_url": "string",
  "expires_at": 0
}
```

### GET /form/{session_id}
Возвращает HTML-форму для расчёта уровня существенности.

## Модели данных

### Indicator
```python
class Indicator(BaseModel):
    name: str = Field(..., example="Выручка от продаж", description="Название финансового показателя")
    value: float = Field(..., example=1800000, description="Значение показателя в рублях")
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
    message: str = "Расчёт выполнен успешно"
```

## Примеры использования

### Пример запроса через API
```bash
curl -X POST "http://localhost:8000/api/v1/calculate" \
-H "Content-Type: application/json" \
-d '{
  "indicators": [
    {"name": "Выручка", "value": 1800000},
    {"name": "Прибыль", "value": 480000}
  ],
  "deviation_threshold": 50,
  "rounding_limit": 50,
  "with_docx": false
}'
```

### Пример ответа
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
        "name": "Выручка",
        "value": 1800000.0,
        "deviation": {
          "absolute": 942857.1428571428,
          "percent": 110.0
        }
      },
      {
        "name": "Прибыль",
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
    {"name": "Выручка", "value": 1800000.0},
    {"name": "Прибыль", "value": 480000.0}
  ],
  "message": "Расчёт выполнен успешно"
}
```

## Логика работы

1. **Расчёт среднего арифметического**:
   - Вычисляется среднее значение всех показателей

2. **Анализ отклонений**:
   - Для каждого показателя вычисляется процент отклонения от среднего
   - Показатели с отклонением > `deviation_threshold` исключаются

3. **Перерасчёт среднего**:
   - Среднее вычисляется заново по оставшимся показателям

4. **Округление результата**:
   - Результат округляется до ближайших 100 рублей
   - Если отклонение при округлении > `rounding_limit`, округление не применяется

5. **Формирование отчёта**:
   - Генерируется JSON с детализацией расчётов
   - По запросу создаётся Word-документ с полным отчётом

## Развёртывание

### Docker
```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY . .

RUN pip install -r requirements.txt

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Kubernetes
Пример deployment.yaml:
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

## Тестирование

Запуск тестов:
```bash
pytest tests/
```

Пример теста:
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

## Лицензия

Copyright (c) 2025 CARL_TECH

