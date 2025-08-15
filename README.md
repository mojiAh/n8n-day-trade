# n8n-day-trade
A n8n pipeline which analyse stocks for day trading.

### Project aim
A simple n8n pipeline to analyze stocks for day trading. This repo contains configuration, exported workflows and scripts to run n8n in Docker and expose it for webhook testing.
The pipeline is triggered by a stock symbol received via Telegram. It fetches 1-minute, 15-minute, and 1-hour candlestick data for the specified symbol, aggregates the data, and prepares features for AI analysis.

In parallel, the workflow collects the latest news for the target stock, runs a sentiment analysis node on those articles, and passes the resulting sentiment data to the AI alongside the aggregated market data.

### Get hands on (quick)
Prerequisites:
- Docker (and optionally Docker Compose)
- git
- ngrok (for exposing local webhooks)

Quick start:
1. Clone the repo:
    ```
    git clone <repo-url>
    cd <repo>
    ```
2. Create a local data directory to persist n8n state:
    ```
    mkdir -p ./n8n_data
    ```

### How to run Docker
Example Docker run (bind mount for persistence and to keep workflows editable locally):
```
docker run --rm -it \
        -p 5678:5678 \
        -v n8n_data:/home/node/.n8n \
        -v "$HOME/n8n/certs":/home/node/certs:ro \
        -e N8N_PROTOCOL=https \
        -e N8N_PORT=5678 \
        -e N8N_SSL_KEY=/home/node/certs/key.pem \
        -e N8N_SSL_CERT=/home/node/certs/cert.pem \
        -e WEBHOOK_URL="$1" \
        n8nio/n8n
```
Notes:
- Port 5678 is n8n's default UI and API port.
- Use Docker Compose for multi-service setups; mount n8n_data to `/home/node/.n8n` so workflows, credentials and settings persist on the host.

### How and why ngrok is used
Why:
- n8n receives webhooks (e.g., trading signals). ngrok exposes your local n8n instance to the public internet so external services can POST to your workflows for testing.

How to use:
1. Start n8n (see above).
2. Start an ngrok HTTP tunnel to the n8n port:
    ```
    ngrok http https://localhost:5678
    ```
3. Copy the generated public URL (https://xxxx.ngrok.io) and set it as the webhook URL as WEBHOOK_URL in docker command.

Security: use ngrok auth tokens and restrict endpoints if exposing sensitive data.

### How changes have been copied from the Docker container to the local git repo
Recommended (best practice):
- Use a host bind mount (as shown above) so all workflow exports and credentials are saved directly into ./n8n_data. Those files are then plain files you can add to git.

If changes already exist only inside a running container:
1. Copy files from the container to host:
    ```
    docker cp [container ID]:/home/node/.n8n ./n8n_data
    ```
2. Inspect the copied files, remove any secrets or credential files you don't want in GitHub (or use a credentials export mechanism).
3. Commit and push:
    ```
    git add config database.sqllite crash.journal
    git commit -m "Import n8n workflows from container"
    git push origin main
    ```

Notes:
- Prefer exported workflows (JSON) and environment/config files in Git rather than raw credential secrets.
- Consider using environment variables or secrets managers for production credentials.

- Typical workflow:
  1. Create a feature branch: `git checkout -b feature/xxx`
  2. Make changes locally (or export from n8n and save files)
  3. Commit: `git add . && git commit -m "Describe change"`
  4. Push and open a pull request for review: `git push origin feature/xxx`
  5. Merge to main when approved; tag releases if needed.
- Use .gitignore to exclude sensitive files. Store secrets safely (not in the repo).

### Quick command summary
- Run n8n: see Docker run above
- Start ngrok: `ngrok http 5678`
- Copy data from container: `docker cp n8n:/home/node/.n8n ./n8n_data`
- Git workflow: `git add/commit/push` on exported workflow files

Replace placeholders and adapt paths to your environment. Keep secrets out of the repository.
