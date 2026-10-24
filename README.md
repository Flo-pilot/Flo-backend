# Personalized email workflow

This workflow uses Gemini to draft one personalized plain-text email per client
and Resend to deliver it. It can be run as a CLI or as a FastAPI service.

## Setup

1. Copy `clients.example.json` to `clients.json` and replace the sample data.
2. Set `GEMINI_API_KEY`, `RESEND_API_KEY`, and `EMAIL_FROM` in the environment.
   The sender domain must be verified in Resend.
3. Run a draft-only preview first:

```powershell
$env:GEMINI_API_KEY = "..."
python .\main.py --input .\clients.json --goal "Offer a 20-minute product consultation" --dry-run
```

4. After reviewing the drafts, send them:

```powershell
$env:RESEND_API_KEY = "..."
$env:EMAIL_FROM = "Your Name <hello@your-verified-domain.com>"
python .\main.py --input .\clients.json --goal "Offer a 20-minute product consultation"
```

The input JSON must be an array with `name` and `email`; `company` and `context`
are optional. The workflow stops on the first provider error rather than
silently reporting a successful send.

## API

Install the API dependencies and start the service:

```powershell
python -m pip install -r .\requirements.txt
uvicorn main:app --reload
```

Check that the service is running:

```powershell
Invoke-RestMethod http://127.0.0.1:8000/health
```

Create a draft-only campaign (the default) with `POST /campaigns`:

```powershell
$body = @{
    campaign_goal = "Offer a 20-minute product consultation"
    dry_run = $true
    clients = @(
        @{
            name = "Avery Johnson"
            email = "avery@example.com"
            company = "Example Co"
            context = "Interested in reducing manual reporting."
        }
    )
} | ConvertTo-Json -Depth 5

Invoke-RestMethod -Uri http://127.0.0.1:8000/campaigns `
    -Method Post -ContentType "application/json" -Body $body
```

Set `dry_run` to `false` to send the generated emails. Sending requires both
`RESEND_API_KEY` and `EMAIL_FROM`; draft requests only require
`GEMINI_API_KEY`. Interactive API documentation is available at
`http://127.0.0.1:8000/docs`.
