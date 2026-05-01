# Flow — Frontend

A high-level overview of [`akd-flow`](https://github.com/NASA-IMPACT/akd-flow), the Next.js frontend for AKD Flow.

## Tech stack

- **Next.js 15.4** (App Router).
- **React 19** + **TypeScript 5**.
- **Tailwind CSS 4** + **Radix UI** (headless primitives) — visual system.
- **Zustand 5** — lightweight client-side state store.
- **React Flow** (`@xyflow/react` 12.8+) — the visual workflow canvas.
- **React Hook Form 7** + **Zod 4** — form validation for agent parameter fields.
- **react-markdown** + **remark-gfm** — render agent Markdown outputs.
- **Streamdown** — parse Server-Sent Events from the backend.
- **Vitest 4** + **React Testing Library** — testing.
- Native **`fetch()`** for HTTP — no heavy HTTP client.

## The user surfaces

Three main UI regions appear on the main workspace (`src/app/mainPage/page.tsx`):

- **Chat list sidebar** (`ChatListSidebar`) — left. All the user's past chats with rename / share / import / delete affordances.
- **Planner chat** (`Planner`) — center. Renders the ongoing conversation with the LLM planner, shows `RetrievedWorkflowCard` when a new workflow is generated.
- **Workflow canvas** (`workflowCanvas`) — right. The graph editor. Users can drag nodes, connect edges, edit parameters per node, and watch execution.

Additional surfaces:

- **Landing page** (`/landingPage`) — entry for non-authenticated users.
- **Markdown viewer** (`/markdownViewer`) — public shared-chat view for users who received a shared URL.
- **Fact verification panel** (`FactReasonerReport`) — inline fact-checking UI that talks to the separate FactReasoner service.

## Streaming and state

When a workflow runs:

1. `lib/api/workflows.ts` opens a streaming fetch against `/api/v1/workflows/{id}/run`.
2. `lib/streaming.ts` parses each SSE message into a normalized `NormalizedStreamEvent`.
3. Events update the Zustand store (`store/index.ts`) — active node, partial outputs, progress.
4. React components re-render: the canvas highlights the active node, the progress panel appends log entries, the node's output area fills in as the structured output materializes.

No WebSockets are used — streaming is pure HTTP SSE.

## Auth

- JWT obtained from `/api/v1/auth/jwt/login`, stored in `localStorage`.
- `authService.getToken()` reads it on every API call.
- All requests include an `Authorization: Bearer <token>` header.
- 401 responses bounce the user to the login page — no refresh-token flow today; tokens live for the lifetime of a session.

## Environment

Configured via `.env.local`:

- `NEXT_PUBLIC_BACKEND_URL` — the Flow backend (`akd-services`) base URL.
- `NEXT_PUBLIC_FACTUALITY_PIPELINE_ENDPOINT` — the FactReasoner service URL.
- Additional provider / region vars as needed.

## Agent discovery

The frontend ships a static list (`src/constants/agents.ts`, `AVAILABLE_AGENTS`) mapping agent ids to UI metadata (display name, icon, input schema description, output field name). This list must be kept in sync with the Flow backend's registered agents — there is no runtime discovery endpoint today.

## File uploads

Nodes that accept file attachments surface a file picker. Uploaded files go to `/api/v1/files/upload`; the backend stores them and the agent receives them as inline context at runtime.

## Chat sharing and import

- Share: the UI calls a share endpoint that marks a chat public and returns a public URL.
- Public URL: loads the chat via the `/markdownViewer` route with read-only rendering.
- Import: another authenticated user can copy a shared chat into their own workspace.

## Testing

- **Vitest** for unit and integration tests.
- **React Testing Library** for component tests.

## Where to read next

- [`user-guide.md`](./user-guide.md) — the user-facing description.
- [`streaming.md`](./streaming.md) — the SSE event details.
- [`backend.md`](./backend.md) — what the frontend talks to.
- Upstream repo: [`NASA-IMPACT/akd-flow`](https://github.com/NASA-IMPACT/akd-flow).
