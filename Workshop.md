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

### Common gotchas

- Not logged in via `az login`: credential failure
- Forgot `--pre` flag: preview package won't install
- Note: this SDK is in preview; the API may shift between versions

---

## Stage 4: Give the Agent a Tool (via MCP)

**What we're teaching:** Agents without tools can only talk. **Model Context Protocol (MCP)** lets them reach out, search docs, query systems, and call APIs.

### Copilot prompt

> **Add an MCP tool connection using a public MCP server and allow the agent to call it.**

### Example

```python
from mcp import McpTool

mcp_tool = McpTool(
    server_label="MicrosoftLearn",
    server_url="https://learn.microsoft.com/api/mcp",
)

agent = client.as_agent(
    name="tool-agent",
    instructions="Use tools when needed.",
    tools=mcp_tool.definitions,
)

response = agent.run(
    "Find official Microsoft documentation about Azure Container Apps",
    tool_resources=mcp_tool.resources,
)
print(response)
```

Ask the agent something that requires a real lookup. The answer cites real docs. It didn't hallucinate, it used a tool.

### Try it yourself

- Change instructions: "always use tools" vs "only when needed", and see how behavior shifts

### Common gotchas

- MCP server unreachable: check network/proxy
- Tool approval mode not configured
- Older models may not support tool calls

---

## Stage 5: Add a UI and Ground in Your Own Data (RAG)

**What we're teaching:** Two things finish the picture: a **human interface** and **enterprise knowledge**.

### Copilot prompt

> **Extend the FastAPI app with a POST `/chat` endpoint that sends user input to the agent and returns the response.**

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.post("/chat")
async def chat(req: Request):
    body = await req.json()
    user_input = body["message"]
    result = agent.run(user_input)
    return {"response": result}
```

### Minimal HTML front-end

```html
<form action="/chat" method="POST">
  <input name="message" />
</form>
```

### Add grounding (RAG) via Foundry

- Upload a document to your Foundry project (vector store / file search)
- Bind it to the agent as a knowledge source
- Foundry handles the indexing and retrieval for you

### Try it yourself

- Upload an SSAB document (a spec, a process doc, an internal handbook)
- Ask a question only that document can answer
- Compare with the generic answer from Stage 3

### Common gotchas

- File not yet indexed: wait a moment after upload
- Tool binding missed in `as_agent(...)`
- Document encoding issues (use UTF-8)

---

## Stage 6: Deploy and Secure (Azure + GitHub Advanced Security)

**What we're teaching:** From toy to production: hosting, security, DevSecOps lifecycle.

### Dockerfile (let Copilot generate it)

```dockerfile
FROM python:3.10
WORKDIR /app
COPY . .
RUN pip install fastapi uvicorn openai azure-identity agent-framework --pre
CMD ["uvicorn","main:app","--host","0.0.0.0","--port","8080"]
```

### Deploy to Azure Container Apps

```bash
az containerapp up \
  --name agent-app \
  --resource-group rg-demo \
  --environment env-demo \
  --source .
```

You get a public URL. A real running service.

### Enable GitHub Advanced Security (GHAS)

In the GitHub repo settings, turn on:

- **Code scanning** (CodeQL)
- **Secret scanning**

### Try it yourself

- Commit a fake API key and watch secret scanning catch it
- Review the CodeQL findings

### Common gotchas

- Missing `containerapp` Azure CLI extension: `az extension add --name containerapp`
- Region doesn't support Container Apps: pick another
- Docker build fails: check Python version pin

---

## What's Still Missing for Production?

Worth discussing at the end:

- Managed identity (no keys in env vars)
- Autoscaling rules
- Monitoring + logging (Application Insights)
- Cost controls + quotas

---

## If Something Breaks: Backup Plan

We have three fallbacks ready:

1. **Pre-built GitHub repo**: clone and skip ahead
2. **GitHub Codespaces**: runs in the browser, no local install
3. **Pre-deployed endpoint**: to demonstrate the final result if local deploy fails

Wi-Fi is the most common failure point. Pre-pulled containers and a USB stick with the repo are on hand.

---

## Recap: The Copilot Pattern You Just Used

At every stage, the workflow was the same:

1. **Describe the intent in plain language** (the Copilot prompt)
2. **Let Copilot generate the scaffold**
3. **Run it and observe**
4. **Refine via follow-up prompts in Copilot Chat**

This is the loop you'll take back to your day-to-day work. Copilot accelerates the *typing*, you stay in charge of the *thinking*.
