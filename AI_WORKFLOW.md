# AI Workflow (SmartMonitor)

This document defines the AI assistant workflow used by SmartMonitor at the system level.

> Status in this repository: **reference architecture only**. The AI server implementation is not present in this codebase, but the app/system contract is documented here for integration.

---

## 1. Purpose

The AI assistant enables:
- natural-language control for device/appliance operations
- contextual advice about energy usage
- conversational support with memory
- logging and observability for actions and responses

---

## 2. Actors & Responsibilities

- **User (mobile app chat UI)**  
  Sends message/commands and receives responses.

- **Flutter App**  
  Sends `userId + message` to AI server and renders response.

- **AI Assistant Server (Node.js/Express, external)**  
  - endpoint handling (`POST /ask-ai`)
  - intent parsing
  - command execution through Firebase when needed
  - LLM prompting for advisory/general queries
  - memory handling and cleanup
  - interaction logging

- **Firebase Realtime Database**  
  Source of truth for device/appliance state and updates.

- **LLM Runtime (e.g., local Ollama/llama3, external)**  
  Generates contextual natural-language responses.

- **Firestore / logging store (external integration)**  
  Persists chat/action logs for analytics and traceability.

---

## 3. API Contract

## Primary endpoint
`POST /ask-ai`

### Request (example)
```json
{
  "userId": "uid_123",
  "message": "Set washing machine peak current to 6A"
}
```

### Response (example)
```json
{
  "type": "command_result",
  "reply": "Updated Washing Machine peak current to 6A.",
  "action": {
    "operation": "update_appliance",
    "status": "success"
  }
}
```

or

```json
{
  "type": "advice",
  "reply": "Your monthly trend suggests a 12% increase. Consider reducing peak-hour usage."
}
```

## Auxiliary endpoints (recommended)
- `GET /health`
- `GET /memory/stats?userId=...`
- `POST /memory/clear` (with user context)

---

## 4. Processing Pipeline

1. **Receive request** from app (`userId`, `message`).
2. **Load context**: recent conversation memory for `userId`.
3. **Intent classification**:
   - command intent (add/update/delete appliance/device)
   - advisory/query intent
4. **If command intent**:
   - parse entities (device, appliance, fields, values)
   - validate operation + permissions
   - execute Firebase update
   - build confirmation response
5. **If advisory/query intent**:
   - build prompt with context + relevant usage summary
   - invoke LLM
   - return natural-language response
6. **Persist interaction logs** (request, intent, action, response metadata).
7. **Update memory** and apply cleanup policy for idle users.

---

## 5. Sequence (Textual UML Style)

1. App → AI Server: `POST /ask-ai` with user message.
2. AI Server → Memory Store: fetch recent context.
3. AI Server:
   - parse intent
   - either:
     - AI Server → Firebase: execute command
     - or AI Server → LLM: generate answer
4. AI Server → Log Store: persist interaction.
5. AI Server → App: return response payload.
6. App → User: display assistant reply / action confirmation.

---

## 6. Memory Model

- Memory is maintained per user.
- Rolling context window (bounded token/message count).
- Idle-session cleanup removes stale memory to reduce costs.
- User-level clear/reset endpoint for privacy and recovery.

---

## 7. Guardrails & Validation

- Reject malformed payloads (missing `userId` / `message`).
- Validate action schema before Firebase writes.
- Restrict command surface to allowed operations.
- Sanitize natural-language extracted identifiers.
- Use explicit success/error responses for deterministic app handling.

---

## 8. Security Considerations

- Authenticate app-to-server requests.
- Authorize all write operations by user ownership.
- Never expose secrets or raw credentials in logs.
- Rate limit AI endpoints and enforce request size limits.
- Keep audit logs for command execution and failures.

---

## 9. Observability

Recommended metrics:
- request count / latency
- command success vs failure rate
- top user intents
- memory size and cleanup frequency
- LLM error rate / timeout rate

Recommended logs:
- request correlation ID
- parsed intent
- command payload + sanitized result
- model call duration and status

---

## 10. Integration Notes for This Repository

- Flutter data layer currently uses Firebase directly for appliance management.
- AI server endpoint client integration is not present in current `lib/` code.
- This workflow document defines the target contract for future integration without changing current app behavior.
