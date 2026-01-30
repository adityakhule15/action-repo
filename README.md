# GitHub Webhook Events 🚀

This repository contains a **GitHub Actions workflow** that captures **Push**, **Pull Request**, and **Merge** events from a GitHub repository and sends them as a structured webhook payload to a live Flask backend deployed on Render.

---

## 🌐 Live Webhook Receiver

- **Webhook URL:** https://webhook-repo-aditya.onrender.com/webhook  
- **Dashboard:** https://webhook-repo-aditya.onrender.com  
- **Health Check:** https://webhook-repo-aditya.onrender.com/health  

---

## ✨ What This Workflow Does

- ✅ Listens to **push events on all branches**
- ✅ Listens to **pull request opened events**
- ✅ Detects **merge events** (PR closed & merged)
- ✅ Builds a clean, structured JSON payload
- ✅ Sends the payload to a Render-hosted webhook endpoint
- ✅ Logs payload & response for debugging

---

## ⚙️ GitHub Actions Workflow

Save the following file as:

```
.github/workflows/github-webhook-events.yml
```

```yaml
name: GitHub Webhook Events

on:
  push:
    branches: ["*"]
  pull_request:
    types: [opened, closed]

jobs:
  send-webhook:
    runs-on: ubuntu-latest
    steps:
      - name: Send Webhook for Push, PR, or Merge
        run: |
          WEBHOOK_URL="https://webhook-repo-aditya.onrender.com/webhook"

          EVENT_TYPE="unknown"
          FROM_BRANCH=""
          TO_BRANCH=""
          TIMESTAMP=""
          TITLE=""

          if [ "${{ github.event_name }}" = "push" ]; then
            EVENT_TYPE="push"
            FROM_BRANCH="${{ github.ref_name }}"
            TO_BRANCH="${{ github.ref_name }}"
            TIMESTAMP="${{ github.event.head_commit.timestamp }}"
            TITLE="${{ github.event.head_commit.message }}"

          elif [ "${{ github.event_name }}" = "pull_request" ] && [ "${{ github.event.action }}" = "opened" ]; then
            EVENT_TYPE="pull_request"
            FROM_BRANCH="${{ github.event.pull_request.head.ref }}"
            TO_BRANCH="${{ github.event.pull_request.base.ref }}"
            TIMESTAMP="${{ github.event.pull_request.created_at }}"
            TITLE="${{ github.event.pull_request.title }}"

          elif [ "${{ github.event_name }}" = "pull_request" ] && [ "${{ github.event.action }}" = "closed" ] && [ "${{ github.event.pull_request.merged }}" = "true" ]; then
            EVENT_TYPE="merge"
            FROM_BRANCH="${{ github.event.pull_request.head.ref }}"
            TO_BRANCH="${{ github.event.pull_request.base.ref }}"
            TIMESTAMP="${{ github.event.pull_request.merged_at }}"
            TITLE="${{ github.event.pull_request.title }}"
          else
            exit 0
          fi

          if [ -z "$TIMESTAMP" ]; then
            TIMESTAMP=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
          fi

          PAYLOAD=$(cat << EOF
          {
            "event_type": "$EVENT_TYPE",
            "author": "${{ github.actor }}",
            "from_branch": "$FROM_BRANCH",
            "to_branch": "$TO_BRANCH",
            "timestamp": "$TIMESTAMP",
            "repository": "${{ github.repository }}",
            "commit_id": "${{ github.sha }}",
            "title": "$TITLE",
            "action": "${{ github.event.action }}"
          }
          EOF
          )

          curl -X POST \
            -H "Content-Type: application/json" \
            -d "$PAYLOAD" \
            "$WEBHOOK_URL"
```

---

## 📦 Payload Structure

```json
{
  "event_type": "push | pull_request | merge",
  "author": "github-username",
  "from_branch": "source-branch",
  "to_branch": "target-branch",
  "timestamp": "ISO-8601 UTC time",
  "repository": "owner/repository",
  "commit_id": "commit SHA",
  "title": "commit / PR title",
  "action": "opened | closed"
}
```

---

## 🧪 How to Test

1. Push a commit to any branch  
2. Open a pull request  
3. Merge the pull request  

Then visit:
```
https://webhook-repo-aditya.onrender.com
```

You’ll see events appear in real time 🎉

---

## 🛠 Requirements

- GitHub repository
- GitHub Actions enabled
- Public webhook receiver (already deployed)

---

## 👨‍💻 Author

**Aditya Khule**  
GitHub: https://github.com/adityakhule15  

---

## 📜 License

MIT License

---

Happy Automating 🚀
