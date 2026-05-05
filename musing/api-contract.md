# API Contract Draft

This is a draft contract for a musing memory service that can be used through ChatGPT, MCP clients, or other AI surfaces.

The contract is intentionally small. The service should preserve and reshape attention before it collapses into advice, action, or self-analysis. It should combat slop by holding most AI observations silently in its own graph.

## Design Principles

- The public primitive is the fragment.
- The system should preserve before interpreting.
- AI stewardship is silent by default.
- The service should hold back most of what it infers.
- Advice, action, and self-analysis require explicit user release.
- The user can inspect, edit, hide, delete, and reshape memory.
- Branching should be lightweight and should not require committing to a workflow.
- The API should be transport-agnostic and easy to expose as MCP tools.

## Core Primitive

### Fragment

A fragment is a preserved unit of attention before it has been forced into meaning.

Examples:

- A thought
- A link
- A quote
- A question
- A tangent
- A loop
- A phrase
- A contradiction
- A half-formed idea
- A chat excerpt

```ts
type Fragment = {
  id: string;
  space_id: string;
  author_id: string;
  content: FragmentContent;
  source: FragmentSource;
  created_at: string;
  updated_at: string;
  visibility: "private" | "shared";
  state: "active" | "hidden" | "deleted";
  parent_id?: string;
  branch_id?: string;
  tags?: string[];
  user_note?: string;
};

type FragmentContent =
  | { type: "text"; text: string }
  | { type: "link"; url: string; title?: string; note?: string }
  | { type: "quote"; text: string; attribution?: string }
  | { type: "file_ref"; uri: string; title?: string };

type FragmentSource = {
  client: "chatgpt" | "claude" | "local" | "api" | "other";
  channel?: string;
  conversation_id?: string;
  message_id?: string;
};
```

## Modes

Modes are user-declared attentional contracts.

They let the user say how the system should treat attention without requiring a complex workflow.

Examples:

- Hold this as a fragment.
- Branch this.
- Do not analyze yet.
- No advice.
- Only reflect patterns.
- You are free for one round.

```ts
type Mode = {
  id: string;
  space_id: string;
  name: string;
  instruction: string;
  scope: "next_message" | "conversation" | "branch" | "space";
  status: "active" | "expired" | "revoked";
  created_at: string;
  expires_at?: string;
  release_phrase?: string;
};
```

Modes are how the system implements ack-registered behavior.

## Latent Stewardship

Latent stewardship is the system's private graph of quiet observations.

Most of it should not be surfaced. The service can keep possible connections, recurring patterns, unresolved tensions, and weak signals internally, but the visible surface should be sparse.

```ts
type StewardshipObservation = {
  id: string;
  space_id: string;
  kind:
    | "possible_connection"
    | "recurring_phrase"
    | "unresolved_tension"
    | "branch_candidate"
    | "loop_candidate"
    | "framing_shift"
    | "ontology_candidate";
  summary: string;
  fragment_ids: string[];
  confidence: number;
  maturity: "weak" | "repeated" | "stable";
  visibility: "latent" | "surfaced" | "dismissed";
  created_at: string;
  surfaced_at?: string;
};
```

Default rule:

- Store many.
- Surface few.
- Prefer silence over mediocre insight.

## Public Tools

These names are MCP-friendly, but the same contract can be exposed over HTTP or another transport.

### `capture_fragment`

Preserve a fragment.

Request:

```json
{
  "space_id": "space_123",
  "author_id": "user_123",
  "content": {
    "type": "text",
    "text": "I keep thinking about changing how change happens."
  },
  "source": {
    "client": "chatgpt",
    "conversation_id": "conv_123",
    "message_id": "msg_123"
  },
  "parent_id": null,
  "branch_id": null,
  "mode_ids": ["mode_no_advice"]
}
```

Response:

```json
{
  "fragment_id": "frag_123",
  "captured": true,
  "surfaced": []
}
```

The empty `surfaced` array is important. Capture should usually be quiet.

### `branch_fragment`

Create or attach to a branch without forcing resolution.

Request:

```json
{
  "space_id": "space_123",
  "fragment_id": "frag_123",
  "label": "changing how change happens"
}
```

Response:

```json
{
  "branch_id": "branch_123",
  "fragment_id": "frag_123"
}
```

### `link_fragments`

Record a user-approved connection between fragments.

Request:

```json
{
  "space_id": "space_123",
  "from_fragment_id": "frag_123",
  "to_fragment_id": "frag_456",
  "relation": "resonates_with",
  "note": "These both circle around not collapsing into product too early."
}
```

Response:

```json
{
  "link_id": "link_123"
}
```

### `search_fragments`

Retrieve fragments and minimal context.

Request:

```json
{
  "space_id": "space_123",
  "query": "what keeps returning around advice and slop?",
  "limit": 10,
  "include_latent": false
}
```

Response:

```json
{
  "fragments": [
    {
      "id": "frag_123",
      "snippet": "Advice, action, or self-analysis is not where the value lives.",
      "created_at": "2026-05-05T12:00:00Z"
    }
  ],
  "surfaced": []
}
```

### `register_mode`

Set an attentional contract.

Request:

```json
{
  "space_id": "space_123",
  "name": "no_advice",
  "instruction": "Do not provide advice unless explicitly released.",
  "scope": "conversation",
  "release_phrase": "YOU ARE FREE"
}
```

Response:

```json
{
  "mode_id": "mode_123",
  "status": "active"
}
```

### `release_mode_once`

Temporarily release a mode for one response.

Request:

```json
{
  "space_id": "space_123",
  "mode_id": "mode_123",
  "release_phrase": "YOU ARE FREE"
}
```

Response:

```json
{
  "released": true,
  "scope": "next_message"
}
```

### `request_reflection`

Ask the stewardship layer to surface a small amount of context.

Request:

```json
{
  "space_id": "space_123",
  "prompt": "What am I circling around here?",
  "max_observations": 3,
  "allow_advice": false,
  "allow_action_items": false
}
```

Response:

```json
{
  "reflection": "You seem to be circling around restraint: preserving attention, resisting slop, and letting stewardship hold more than it says.",
  "observations": [
    {
      "id": "obs_123",
      "kind": "recurring_phrase",
      "summary": "Several fragments contrast preservation with collapse into advice or action.",
      "fragment_ids": ["frag_123", "frag_456"]
    }
  ]
}
```

### `inspect_stewardship`

Expose latent observations only when explicitly requested.

Request:

```json
{
  "space_id": "space_123",
  "query": "show me latent observations about anti-slop",
  "limit": 20
}
```

Response:

```json
{
  "observations": [
    {
      "id": "obs_123",
      "kind": "possible_connection",
      "summary": "Anti-slop and no-advice appear to be the same restraint principle at different layers.",
      "fragment_ids": ["frag_123", "frag_456"],
      "maturity": "weak"
    }
  ]
}
```

### `update_fragment`

Let the user edit memory directly.

Request:

```json
{
  "space_id": "space_123",
  "fragment_id": "frag_123",
  "patch": {
    "user_note": "This is more about attention than product."
  }
}
```

Response:

```json
{
  "fragment_id": "frag_123",
  "updated": true
}
```

### `hide_or_delete_fragment`

Remove a fragment from normal use.

Request:

```json
{
  "space_id": "space_123",
  "fragment_id": "frag_123",
  "state": "hidden"
}
```

Response:

```json
{
  "fragment_id": "frag_123",
  "state": "hidden"
}
```

## Surface Policy

The service should surface an observation only when at least one of these is true:

- The user explicitly asks for reflection.
- The user asks to inspect latent stewardship.
- A registered mode permits that kind of surfacing.
- The observation is mature enough and directly relevant to the current musing.

The service should not surface:

- Generic advice
- Action items by default
- Psychological interpretation as fact
- Weak observations framed as certainty
- Product framing unless the user is asking for it

## Minimal ChatGPT Flow

1. User muses in ChatGPT.
2. ChatGPT calls `capture_fragment`.
3. The service stores the fragment and silently updates latent stewardship.
4. The service usually returns no surfaced observation.
5. If the user says "branch this," ChatGPT calls `branch_fragment`.
6. If the user asks "what am I circling around?", ChatGPT calls `request_reflection`.
7. If the user asks "show me what you are seeing silently," ChatGPT calls `inspect_stewardship`.
8. The user can edit, hide, delete, or reshape anything.

## Non-Goals

- Not a therapy API.
- Not a productivity task system.
- Not a generic chatbot memory store.
- Not an advice engine.
- Not an autonomous agent that decides what matters.

