# AI Agent Documentation

> Project: `ai-agent-aria` | Version: `1.0.0` | Server: `http://localhost:3000`

---

## 1. Overview: What is an AI Agent?

An **AI Agent** is an autonomous software system that perceives its environment, reasons using a Large Language Model (LLM), and takes actions to accomplish goals — without requiring step-by-step human instructions.

Unlike a simple chatbot that responds to a single message, an AI Agent can:

- **Plan** multi-step tasks to reach a goal
- **Remember** context across a conversation (session memory)
- **Act** by calling tools, APIs, or external services
- **Reflect** on results and adjust its approach

### Key Properties

| Property | Description |
|---|---|
| **Autonomy** | Operates independently within defined boundaries |
| **Reactivity** | Responds to user input and environment changes |
| **Proactivity** | Takes initiative to achieve goals |
| **Memory** | Maintains context within and across sessions |

---

## 2. Architecture Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                      AI AGENT SYSTEM                         │
│                                                              │
│  ┌────────────┐        ┌──────────────────────────────       │
│  │   User     │◀──────▶│        REST API Layer       │      │
│  │ (Client/UI)│        │  POST /chat                  │      │
│  └────────────┘        │  GET  /session/:id           │      │
│                        │  DELETE /session/:id         │      │
│                        │  GET  /health                │      │
│                        └──────────────┬───────────────┘      │
│                                       │                      │
│                      ┌────────────────▼───────────────┐      │
│                      │          Agent Core            │      │
│                      │                                │      │
│                      │  ┌──────────────┐              │      │
│                      │  │   Planner     │             │      │
│                      │  │ (Decides flow)│             │      │
│                      │  └──────┬───────┘              │      │
│                      │         │                      │      │
│                      │  ┌──────▼────────┐             │      │
│                      │  │     LLM       │             │      │
│                      │  │ (Claude/GPT)  │             │      │
│                      │  └──────┬────────┘             │      │
│                      │         │                      │      │
│                      │  ┌──────▼────────┐             │      │
│                      │  │ Tool Router   │             │      │
│                      │  │ (APIs/Logic)  │             │      │
│                      │  └──────┬────────┘             │      │
│                      └─────────┼──────────────────────┘      │
│                                │                             │
│     ┌──────────────────────────┼──────────────────────────┐  │
│     │                          │                          │  │
│ ┌───▼──────────┐     ┌─────────▼──────────┐     ┌────────▼──┐│
│ │   Memory     │     │  External Tools     │     │  Session  ││
│ │ (Short-term  │     │ (Search, APIs, DB,  │     │  Store    ││
│ │  Context)    │     │  Code Execution)    │     │ (per user)││
│ └──────────────┘     └─────────────────────┘     └───────────┘│
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Component Breakdown

**LLM (Large Language Model)**  
The reasoning brain. Processes conversation history + system instructions and generates the next action or response. Examples: Claude (Anthropic), GPT-4 (OpenAI).

**Memory**  
Stores context so the agent "remembers" across turns:
- *In-context memory* — the conversation history sent with each request
- *Session store* — server-side state keyed by session ID
- *Vector DB* (optional) — long-term semantic memory for retrieval

**Planner / Tool Router**  
Decides whether to respond directly or call a tool (e.g., web search, code execution, database query).

**Session Store**  
Maps each user session (`/session/:id`) to its conversation history, so multiple users are isolated.

---

## 3. Setup Guide

### Prerequisites

| Requirement | Version |
|---|---|
| Node.js | v18 or higher |
| npm | v9 or higher |
| OS | Windows / macOS / Linux |

> **Tip:** Check your versions with `node -v` and `npm -v` before starting.

---

### Step 1 — Download the Project

Download or clone the project folder. The folder name may contain spaces or parentheses (e.g., `ai-agent-project(P1)`), so always wrap it in quotes when using the terminal:

```bash
cd "ai-agent-project(P1)"
```

> ⚠️ **Common mistake:** Running `npm start` from the wrong directory (e.g., `Downloads/` instead of the project folder) causes an `ENOENT: no such file or directory` error. Always `cd` into the project folder first.

---

### Step 2 — Install Dependencies

```bash
npm install
```

This reads `package.json` and installs all required packages into `node_modules/`.

---

### Step 3 — Configure Environment Variables

Create a `.env` file in the project root:

```env
# .env
ANTHROPIC_API_KEY=your_api_key_here
PORT=3000
```

Replace `your_api_key_here` with your actual API key.

---

### Step 4 — Start the Agent Server

```bash
npm start
```

You should see:

```
🚀 Server running on http://localhost:3000
📋 POST  /chat
📋 GET   /session/:id
🗑️  DELETE /session/:id
❤️  GET   /health
```

The agent is now running and ready to accept requests.

---

### Step 5 — Verify the Server is Healthy

```bash
curl http://localhost:3000/health
```

Expected response:

```json
{ "status": "ok" }
```

---

### Available npm Scripts

| Command | Description |
|---|---|
| `npm start` | Start the agent server (runs `node agent/agent.js`) |
| `npm test` | Run the test suite |

> **Note:** There is no `npm run server` script. Only `npm start` is defined. Running `npm run server` will produce a `Missing script: "server"` error.

---

## 4. Example API Request & Response

### Endpoint: `POST /chat`

Send a message to the agent. If a `sessionId` is provided, the agent recalls prior conversation history.

#### Request

```http
POST http://localhost:3000/chat
Content-Type: application/json

{
  "message": "What is the capital of France, and what is 12 multiplied by 8?",
  "sessionId": "user-session-abc123"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `message` | `string` | ✅ Yes | The user's input message |
| `sessionId` | `string` | ❌ Optional | ID to maintain conversation context |

---

#### Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "reply": "The capital of France is **Paris**. And 12 multiplied by 8 equals **96**.",
  "sessionId": "user-session-abc123",
  "usage": {
    "input_tokens": 42,
    "output_tokens": 28
  }
}
```

| Field | Type | Description |
|---|---|---|
| `reply` | `string` | The agent's response (may include Markdown) |
| `sessionId` | `string` | Echo of the session ID for client tracking |
| `usage` | `object` | Token counts for billing/monitoring |

---

### Endpoint: `GET /session/:id`

Retrieve conversation history for a session.

#### Request

```http
GET http://localhost:3000/session/user-session-abc123
```

#### Response

```json
{
  "sessionId": "user-session-abc123",
  "history": [
    { "role": "user", "content": "What is the capital of France?" },
    { "role": "assistant", "content": "The capital of France is Paris." }
  ],
  "createdAt": "2026-04-01T10:49:00Z"
}
```

---

### Endpoint: `DELETE /session/:id`

Clear a session and its conversation history.

#### Request

```http
DELETE http://localhost:3000/session/user-session-abc123
```

#### Response

```json
{
  "message": "Session user-session-abc123 deleted successfully."
}
```

---

### Error Responses

| HTTP Status | Meaning | Example Cause |
|---|---|---|
| `400 Bad Request` | Invalid input | Missing `message` field |
| `401 Unauthorized` | Bad API key | Incorrect `ANTHROPIC_API_KEY` |
| `404 Not Found` | Session missing | Session ID does not exist |
| `500 Internal Server Error` | Agent failure | LLM API timeout or crash |

---

*Last updated: April 1, 2026*
