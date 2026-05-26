# Workshop: From Zero to a Deployed AI Agent

**Audience:** SSAB engineering team
**Duration:** ~6 hours
**Stack:** Python + FastAPI + Microsoft Agent Framework + Azure AI Foundry
**Goal:** Build, ground, and deploy a working AI agent, using GitHub Copilot as your pair-programmer the whole way.

---

## What You'll Build

A single narrative, six stages:

1. Scaffold a Python project with Copilot
2. Make your first call to an Azure AI model
3. Wrap that call into an Agent
4. Give the agent external tools (via MCP)
5. Add a chat UI and ground answers in your own documents
6. Deploy to Azure and secure with GitHub Advanced Security

Keep this map visible throughout. Every stage builds on the previous one.

---

## Before the Workshop: Pre-Flight Checklist

### Install on your laptop

- **VS Code** with extensions:
  - Python
  - **GitHub Copilot**
  - **GitHub Copilot Chat**
- Python 3.10 or newer
- Azure CLI (run `az login` once to confirm it works)
- Git
- Docker Desktop (needed for the deploy stage)

### Azure access you'll need

- An Azure subscription with permission to create resource groups
- An Azure AI Foundry project with a chat model already deployed
- A GitHub account with access to the workshop repo

### Quick validation (5 lines, run the day before)

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install openai azure-identity fastapi uvicorn
az login
python -c "from azure.identity import DefaultAzureCredential; print('ok')"
```

If the last line prints `ok`, you're ready.

---

## Stage 1: From Empty VS Code to a Running Project

**What we're teaching:** Copilot lives inside your normal dev workflow. It's not a separate tool. It's structured prompting at the cursor.

### Setup

```bash
mkdir agent-demo
cd agent-demo
python -m venv .venv
source .venv/bin/activate
pip install fastapi uvicorn openai azure-identity python-dotenv
```

### Copilot prompt: paste into Copilot Chat

> **Create a minimal FastAPI app with one endpoint `/health` that returns `{"status":"ok"}`. Use uvicorn for running.**

Copilot will generate `main.py`:

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/health")
def health():
    return {"status": "ok"}
```

### Run it

```bash
uvicorn main:app --reload
```

Open `http://localhost:8000/health`. You have a working service in 60 seconds.

### Try it yourself

- Rename the endpoint
- Add a second endpoint `/hello?name=` (let Copilot write it)

### Common gotchas

| Problem | Fix |
|---|---|
| Port already in use | `--port 8001` |
| "Module not found" | Activate your venv |
| Windows activation | Use `.venv\Scripts\activate` |

---

## Stage 2: Your First Call to Azure AI Foundry

**What we're teaching:** Before agents, understand the basics: input, model, output. One call, one answer.

### Add your credentials to `.env`

```
AZURE_AI_PROJECT_ENDPOINT=...
MODEL_DEPLOYMENT_NAME=...
```

### Copilot prompt

> **Write Python code using Azure Identity and Azure AI Foundry to call a chat model and send "Explain DevSecOps in one sentence".**

### Expected output

```python
from openai import AzureOpenAI
from azure.identity import DefaultAzureCredential
import os

client = AzureOpenAI(
    azure_endpoint=os.getenv("AZURE_AI_PROJECT_ENDPOINT"),
    api_key=os.getenv("AZURE_OPENAI_API_KEY"),
)

response = client.responses.create(
    model=os.getenv("MODEL_DEPLOYMENT_NAME"),
    input="Explain DevSecOps in one sentence",
)
print(response.output[0].content[0].text)
```

### Try it yourself

- Ask a question specific to SSAB's domain (steel, sustainability, manufacturing)
- Change the system tone (formal vs casual)
- Try the same prompt in Finnish vs English and compare the answers

### Common gotchas

- Wrong region: "model not found"
- Missing key or auth: 401
- Deployment name mismatch: copy the exact name from the Foundry portal

---

## Stage 3: Wrap the Call in an Agent

**What we're teaching:** An agent adds **state, intent, and instructions** on top of a raw model call. Same model, different behavior.

### Install

```bash
pip install agent-framework --pre
```

### Copilot prompt

> **Create a simple Agent Framework agent using FoundryChatClient that answers user questions.**

### Expected code

```python
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential
import os

client = FoundryChatClient(
    project_endpoint=os.getenv("AZURE_AI_PROJECT_ENDPOINT"),
    model=os.getenv("MODEL_DEPLOYMENT_NAME"),
    credential=AzureCliCredential(),
)

agent = client.as_agent(
    name="demo-agent",
    instructions="You are a precise enterprise developer assistant.",
)

result = agent.run("What is DevSecOps?")
print(result)
```

Same question as Stage 2, but the answer is now shaped by the agent's instructions.

### Try it yourself

- Rewrite the instructions for an SSAB-specific persona
- Compare "very concise advisor" vs "verbose, explain-everything advisor"
