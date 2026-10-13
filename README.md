# Vyogo Frappe AI Suite

Welcome to the **Vyogo Frappe AI Suite** monorepo! This repository unifies the three core pillars of the Frappe AI ecosystem maintained by [VyogoTech](https://github.com/vyogotech). 

This suite brings advanced AI capabilities, agentic workflows, and Model Context Protocol (MCP) support to the Frappe Framework and ERPNext.

---

## Architecture Overview

Here is a high-level overview of how the components within the Frappe AI Suite interact with one another and your existing Frappe/ERPNext application:

```text
      [ User ]
         │
         │ Interacts via
         ▼
 ┌──────────────────────────────────────────────────┐
 │ Vyogo Frappe AI Suite                            │
 │                                                  │
 │   [ AI Sidebar ]                                 │
 │         │                                        │
 │         │ Sends prompt & context                 │
 │         ▼                                        │
 │   [ Frappe AI Agent ]                            │
 │         │                                        │
 │         │ Requests tools & context via MCP       │
 │         ▼                                        │
 │   [ Frappe MCP Server ]                          │
 │                                                  │
 └──────────────────────────────────────────────────┘
         │
         │ Reads/Writes via Frappe ORM API
         ▼
 [ Frappe / ERPNext ]
```

The suite is broken down into three decoupled yet highly synergistic components, included here as Git Submodules:

### 1. [Frappe MCP Server](https://github.com/vyogotech/frappe-mcp-server)
**The Data & Capability Layer**
- Implements the [Model Context Protocol (MCP)](https://modelcontextprotocol.io).
- Exposes Frappe ORM, Document interactions, and Meta-data as standardized tools and resources to LLMs.
- Acts as the secure bridge between your Frappe data and any MCP-compatible AI client.

### 2. [Frappe AI Agent](https://github.com/vyogotech/frappe-ai-agent)
**The Orchestration & Reasoning Layer**
- The intelligent agent that orchestrates complex workflows within the Frappe ecosystem.
- Consumes the MCP server to read contexts, execute Frappe actions, and formulate structured plans.
- Capable of autonomous task execution tailored to Frappe/ERPNext business logic.

### 3. [AI Sidebar](https://github.com/vyogotech/frappe_ai)
**The Interaction & Presentation Layer** (repository and app module: `frappe_ai`)
- AI Sidebar by Vyogo Technologies: a conversational AI interface built into the Frappe Desk.
- Plugs directly into your ERPNext/Frappe workspace to allow users to interact with the AI Agent and MCP server in real-time.
- Features intuitive chat UI, prompt templates, and seamless context awareness of the active Frappe page.

---

## Multi-tenancy

One deployment of the suite can serve many Frappe sites, whether they live on one bench or across several. A single-site install needs nothing extra.

- **Every request names its site.** The UI app tells the agent which site the user is on, the agent passes that on to the MCP server, and both use it on every call back to Frappe. A user's session is only ever accepted for the site that issued it.
- **Each site can live on a different bench.** The agent and the MCP server look up a site's bench in their own configuration. A site they have not been told about is refused, so a request can never be steered to an arbitrary backend.
- **Tenants never share state.** Session caches, CSRF tokens, rate limits and chat-history lookups are kept per site, and session ids are never held in clear.

| Deployment | What to configure |
| --- | --- |
| One site on one bench (self-hosted, local compose) | Nothing beyond the single Frappe URL each service already takes |
| Several sites on one shared bench | The same single URL; the site name routes the request inside the bench |
| Many benches behind one suite deployment | A site-to-bench routes file, picked up without a restart when it changes |

See [DEPLOYMENT_PLAN.md](DEPLOYMENT_PLAN.md) for the Kubernetes rollout and each component's README for its settings.

---

## Getting Started

### Prerequisites
- Node.js (v18+)
- Python 3.10+
- Frappe Framework v14/v15
- Git

### Cloning the Suite
When cloning this repository, ensure you pull the submodules:

```bash
git clone --recurse-submodules https://github.com/vyogotech/frappe-ai-suite.git
cd frappe-ai-suite
```

If you have already cloned it without submodules, run:
```bash
git submodule update --init --recursive
```

---

## Documentation
- [MCP Server Docs](https://github.com/vyogotech/frappe-mcp-server/blob/main/README.md)
- [AI Agent Docs](https://github.com/vyogotech/frappe-ai-agent/blob/main/README.md)
- [AI Sidebar Docs](https://github.com/vyogotech/frappe_ai/blob/main/README.md)

## Contributing
Please review the contribution guidelines in the respective submodule repositories. We follow standard Git Flow and Conventional Commits.

---
*Built by [VyogoTech](https://github.com/vyogotech)*
