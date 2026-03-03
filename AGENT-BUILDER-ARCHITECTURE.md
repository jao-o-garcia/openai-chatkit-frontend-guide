## Architecture Comparison: Agent Builder vs Own Server (Advanced)

### High-Level Summary

| Aspect | **Agent Builder** (your project) | **Own Server / Advanced** (reference project) |
|---|---|---|
| Backend | **OpenAI-hosted** (Agent Builder workflow) | **Self-hosted** Python (FastAPI + Agents SDK) |
| Auth mechanism | `getClientSecret` → exchanges a `client_secret` via OpenAI API | `api.url` + `domainKey` → proxied to your own server |
| ChatKit API route | Next.js API route `/api/chatkit/session` calls `https://api.openai.com/v1/chatkit/sessions` | No session route; Next.js `rewrites` proxy `/chatkit` to `localhost:8000/chatkit` |
| `next.config` | No rewrites needed | **Must have** rewrite rules to proxy to backend |
| `useChatKit` config | Uses `api.getClientSecret()` | Uses `api.url` + `api.domainKey` |
| Packages needed | `@openai/chatkit-react` only | `@openai/chatkit-react` **and** `@openai/chatkit` |
| Agent state / orchestration | Handled entirely by OpenAI (opaque) | Exposed — you manage threads, events, agents, guardrails via `src/lib/` |
| Frontend complexity | Minimal — just mount `<ChatKit>` | Complex — `AgentPanel`, thread hydration, event normalization, multiple callbacks |

---

### Where the Requests Flow

**Agent Builder (your project):**

```
Browser (ask-agent/page.tsx)
  │
  │  fetch('/api/chatkit/session')
  ▼
Next.js API Route (/api/chatkit/session/route.ts)
  │
  │  POST https://api.openai.com/v1/chatkit/sessions
  │  with WORKFLOW_ID + OPENAI_API_KEY
  ▼
OpenAI returns client_secret
  │
  ▼
Browser receives client_secret
  │
  │  ChatKit JS SDK uses client_secret
  │  to talk DIRECTLY to OpenAI servers
  ▼
OpenAI-hosted Agent Builder workflow
```

**Own Server (reference project):**

```
Browser (chatkit-panel.tsx)
  │
  │  ChatKit SDK sends to "/chatkit" (relative URL)
  ▼
Next.js rewrite rule (next.config.js)
  │
  │  Proxy to http://127.0.0.1:8000/chatkit
  ▼
Python FastAPI backend (main.py)
  │
  │  OpenAI Agents SDK processes the request
  │  (agent orchestration, tools, guardrails)
  ▼
Streaming SSE response back through the same chain
```

---

### Key Structural Differences in Code

**1. `useChatKit` hook configuration**

Agent Builder — uses `getClientSecret`:

```14:56:src/app/ask-agent/page.tsx
    api: {
      async getClientSecret(existingSecret) {
        // ...
        const res = await fetch('/api/chatkit/session', {
          method: 'POST',
          // ...
        });
        const data = await res.json();
        return data.client_secret;
      },
    },
```

Own Server — uses `url` + `domainKey`:

```typescript
api: {
  url: "/chatkit",
  domainKey: CHATKIT_DOMAIN_KEY,
},
```

**2. `next.config` — rewrites**

Agent Builder: **No rewrites at all.** Your `next.config.ts` has no proxy rules because the ChatKit SDK talks directly to OpenAI after getting the `client_secret`.

Own Server: **Must have rewrites** to proxy ChatKit traffic to the Python backend:

```javascript
async rewrites() {
  return [
    { source: "/chatkit", destination: "http://127.0.0.1:8000/chatkit" },
    { source: "/chatkit/:path*", destination: "http://127.0.0.1:8000/chatkit/:path*" },
  ];
},
```

**3. API route vs no API route**

Agent Builder: Has `src/app/api/chatkit/session/route.ts` that calls OpenAI's sessions endpoint with `WORKFLOW_ID` and `OPENAI_API_KEY`, then returns the `client_secret`.

Own Server: **No API route.** Instead, the Python backend at port 8000 handles everything. The reference project has `src/lib/api.ts` and `src/lib/types.ts` for thread/agent state management.

**4. Package dependencies**

Agent Builder: Only needs `@openai/chatkit-react`.

Own Server: Needs **both** `@openai/chatkit-react` **and** `@openai/chatkit` (the base SDK that the Python backend's `openai-chatkit` package communicates with).

**5. Environment variables**

Agent Builder: `OPENAI_API_KEY` + `WORKFLOW_ID`

Own Server: `NEXT_PUBLIC_CHATKIT_DOMAIN_KEY` (frontend) + whatever the Python backend needs (`OPENAI_API_KEY` on the Python side)

**6. Component complexity**

Agent Builder: Single page component (`ask-agent/page.tsx`) with just `<ChatKit control={control} />`. No props for thread management, no agent panel, no event handling.

Own Server: `ChatKitPanel` accepts multiple callback props (`onThreadChange`, `onResponseEnd`, `onRunnerUpdate`, `onRunnerEventDelta`, `onRunnerBindThread`). A separate `AgentPanel` component visualizes agent state, events, and guardrails. The main `page.tsx` maintains heavy state for threads, agents, events, guardrails, and context.

---

## Checklist for Agent Builder Architecture

Here's the equivalent checklist for your Agent Builder approach:

```
## Checklist for Agent Builder 
- [ ] Is architecture Agent Builder (getClientSecret)?
    - [ ] Does package.json have @openai/chatkit-react?
          (Does NOT need @openai/chatkit — that's for Own Server only)
    - [ ] Does next.config have NO rewrite/proxy rules for /chatkit?
          (No proxy needed — ChatKit SDK talks directly to OpenAI after auth)
    - [ ] Does app/layout.tsx load the ChatKit CDN script?
          (`<Script src="https://cdn.platform.openai.com/deployments/chatkit/chatkit.js" />`)
    - [ ] Does app/layout.tsx render {children}?
    - [ ] Does an API route exist at app/api/chatkit/session/route.ts?
          - [ ] Does it call `https://api.openai.com/v1/chatkit/sessions`?
          - [ ] Does it send the `OpenAI-Beta: chatkit_beta=v1` header?
          - [ ] Does it send `Authorization: Bearer ${OPENAI_API_KEY}`?
          - [ ] Does it send `workflow: { id: WORKFLOW_ID }` in the body?
          - [ ] Does it send a unique `user` identifier in the body?
          - [ ] Does it return `{ client_secret }` to the frontend?
    - [ ] Are env vars set? (`OPENAI_API_KEY`, `WORKFLOW_ID`)
      - Trace where ChatKit is rendered (starting from app page):
          - [ ] Does the page with ChatKit have "use client" at the top?
          - [ ] Does the component use the `useChatKit` hook?
            - [ ] Is `api.getClientSecret` defined (NOT api.url / domainKey)?
            - [ ] Does getClientSecret fetch from your API route 
                  (e.g., `/api/chatkit/session`)?
            - [ ] Does it return the `client_secret` string?
            - [ ] Does the hook return a control object?
          - [ ] Does it render `<ChatKit control={control} />`?
          - [ ] No domainKey prop needed?
          - [ ] No api.url prop needed?
```

---

## Side-by-Side Quick Reference

| Check Item | Agent Builder | Own Server |
|---|---|---|
| `@openai/chatkit-react` | Required | Required |
| `@openai/chatkit` | **Not needed** | Required |
| CDN script in layout | Required | Required |
| `next.config` rewrites | **None** | Required (`/chatkit` -> backend) |
| `useChatKit` auth | `getClientSecret()` | `api.url` + `domainKey` |
| API route for session | Yes (`/api/chatkit/session`) | **None (for rewrites), can be Yes if App Route** |
| Separate backend server | **None** (OpenAI hosts it) | Yes (Python FastAPI) |
| `WORKFLOW_ID` env var | Required | **Not needed** |
| `CHATKIT_DOMAIN_KEY` | **Not needed** | Required |
| Thread/agent state mgmt | **None** (OpenAI handles) | Manual (`src/lib/api.ts`, types, etc.) |
| Agent visualization | **Not available** (opaque) | Available (AgentPanel, events, guardrails) |
| Props/callbacks on ChatKit panel | Minimal (just `control`) | Many (`onThreadChange`, `onResponseEnd`, etc.) |

The fundamental trade-off: Agent Builder is simpler to set up and maintain (OpenAI handles orchestration, scaling, and state), but you lose visibility into and control over the agent internals. The Own Server approach gives you full observability (agent panel, guardrail visualization, event streams) and control (custom tools, custom orchestration logic) at the cost of running and maintaining your own Python backend.
