# MicrosoftAgent47

Workshop materials: **From Zero to a Deployed AI Agent**

## Stack

- Python + FastAPI
- Microsoft Agent Framework
- Azure AI Foundry
- GitHub Copilot

## Workshop Stages

1. Scaffold a Python project with Copilot
2. Make your first call to an Azure AI model
3. Wrap that call into an Agent
4. Give the agent external tools (via MCP)
5. Add a chat UI and ground answers in your own documents
6. Deploy to Azure and secure with GitHub Advanced Security

## Prerequisites

- VS Code with GitHub Copilot
- Python 3.10+
- Azure CLI (`az login`)
- Azure subscription with AI Foundry access

## Quick Start

```bash
python -m venv .venv
.venv\Scripts\activate  # Windows
pip install fastapi uvicorn openai azure-identity python-dotenv
```

See [Workshop.md](Workshop.md) for the full guide.
