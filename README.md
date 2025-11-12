# BigBrainLAW-IND FastAPI Service

FastAPI wrapper around the `Manav225/BigBrainLAW-IND` multi-label legal classifier.  
Send complaint text and receive predicted IPC sections together with their probabilities.

## Prerequisites

- Python 3.9+ (3.11 recommended)
- Valid Hugging Face access token with permission to download `Manav225/BigBrainLAW-IND`
- Optional: CUDA-capable GPU + PyTorch CUDA wheels if you want GPU inference
- Git (if cloning from a repository)

### Required Environment Variables

- `HUGGINGFACE_HUB_TOKEN`: Hugging Face access token; the API refuses to start if this variable is missing.
- `PORT` (optional): Port for Uvicorn (defaults to `8000`).
- `CORS_ALLOWED_ORIGINS` (optional): Comma separated origins for browser calls, defaults to `*`.

## Setup

```bash
git clone <repo-url>
cd FastAPI-BigBrainLAW-IND
python -m venv .venv
. .venv/Scripts/activate  # PowerShell: .\.venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install -r requirements.txt
```

If you need GPU acceleration, install the appropriate PyTorch build from https://pytorch.org after creating the environment.

## Running the API

```bash
set HUGGINGFACE_HUB_TOKEN=<your_token>  # PowerShell
uvicorn main:app --host 0.0.0.0 --port 8000
```

### Alternate: auto-run entry point

```bash
set HUGGINGFACE_HUB_TOKEN=<your_token>
python main.py
```

## API Endpoints

- `GET /` → Health check returning `{"message": "FastAPI is running!"}`
- `POST /predict`
  - Body:
    ```json
    {
      "text": "Complaint narrative here…",
      "threshold": 0.5
    }
    ```
  - Response:
    ```json
    {
      "complaint": "...",
      "predicted_labels": ["Section 420 in The Indian Penal Code"],
      "probabilities": [[0.12, 0.87, ...]]
    }
    ```
  - `threshold` is optional (default `0.5`).

## Deployment Notes

- Models are downloaded at startup; keep the token accessible (env var, `.env`, or secrets manager).
- For production, configure `CORS_ALLOWED_ORIGINS`, and use a process manager (e.g., Gunicorn + Uvicorn workers behind Nginx or Azure App Service).
- Bundle `id2label.json` and `label2id.json`; they must reside beside `main.py`.

## Testing Locally

Use `curl` or a REST client:

```bash
curl -X POST http://localhost:8000/predict ^
  -H "Content-Type: application/json" ^
  -d "{\"text\":\"The accused cheated me...\", \"threshold\":0.4}"
```

## Troubleshooting

- **`HUGGINGFACE_HUB_TOKEN not set`**: Ensure the environment variable is defined before starting the app.
- **Model download errors**: Confirm the token has access and network egress to `huggingface.co`.
- **Slow first request**: The first call includes model warm-up; subsequent calls use the cached model.
