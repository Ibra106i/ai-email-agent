# AI Email Agent

Fully automated email responder powered by LLM. Monitors your Gmail inbox, drafts replies using AI, and sends them automatically.

Uses [LiteLLM](https://github.com/BerriAI/litellm) as a unified gateway with Groq (primary), Mistral, and OpenRouter (fallbacks) — all free tiers available.

## Architecture

```
Gmail (unread) → n8n Workflow → LiteLLM Gateway → Groq/Mistral/OpenRouter
                                    ↓
                              Draft Reply → Send → Mark Read
```

## Quick Start

### 1. Clone & configure

```bash
git clone https://github.com/Ibra106i/ai-email-agent.git
cd ai-email-agent
cp .env.example .env
```

Edit `.env` with your API keys (at minimum, get a free key from [console.groq.com](https://console.groq.com)):

```
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxx
LITELLM_API_KEY=sk-your-random-key-here
```

### 2. Start services

```bash
docker compose up -d
```

This starts:
- **n8n** at `http://localhost:5678`
- **LiteLLM Gateway** at `http://localhost:4000`

### 3. Set up n8n

1. Open `http://localhost:5678` → create owner account
2. Go to **Settings → n8n API → Create API Key** (optional, for automation)

### 4. Create Gmail OAuth2 credential

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project (or use existing)
3. Enable **Gmail API** (APIs & Services → Library → search "Gmail API")
4. Set up **OAuth consent screen**:
   - Select **External**
   - App name: anything you like
   - Add your email as developer contact
5. Add **Test users**: your Gmail address
6. Create **OAuth 2.0 Client ID**:
   - Type: **Web application**
   - Redirect URI: `http://localhost:5678/rest/oauth2-credential/callback`
7. Copy the **Client ID** and **Client Secret**

Now in n8n:

8. Go to **Credentials → Add Credential → Gmail OAuth2**
9. Paste your Client ID and Client Secret
10. Click **Sign in with Google** → pick your account → Allow
11. Click **Save**

### 5. Import the workflow

1. In n8n, click **+ Add workflow → Import from File**
2. Select `workflow.json` from this repo
3. Open the **Draft Reply (LiteLLM)** node → update the Authorization header:
   ```
   Bearer sk-your-LITELLM-api-key
   ```
4. Open any **Gmail** node → select the credential you just created
5. Repeat for all 3 Gmail nodes (New Email, Send Reply, Mark as Read)
6. Click **Save** → toggle **Active**

### 6. Test it

Send an email to your Gmail from another account. Within 1 minute, the workflow will:
1. Detect the unread email
2. Draft a reply via AI
3. Send it
4. Mark as read

## Customization

### Change the AI signature

Edit the system prompt in the **Draft Reply (LiteLLM)** node. Find the `content` field and change:

```
Sign off with the name Ahmed.
```

to whatever name you want.

### Change email filters

In the **New Email** node, edit the query filter:

```
-label:inbox_sent -category:promotions -category:social
```

### Change polling frequency

In the **New Email** node, change `everyMinute` to:
- `every5Minutes`
- `every15Minutes`
- `every30Minutes`

## Model Costs

| Provider | Model | Free Tier | Cost After |
|----------|-------|-----------|------------|
| Groq | llama-3.3-70b | 14,400 req/day | $0.59/M tokens |
| Mistral | mistral-small | 1 req/sec | $0.10/M tokens |
| OpenRouter | various | varies | varies |

Groq free tier is generous enough for most use cases.

## Troubleshooting

**"Unable to sign without access token"**
→ Re-create the Gmail OAuth2 credential. Delete the old one, add new, sign in with Google again.

**LiteLLM returning errors**
→ Check `docker logs litellm-gateway`. Ensure at least one API key is valid.

**Workflow not triggering**
→ Make sure the workflow is toggled **Active** (top-right switch).

## License

MIT
