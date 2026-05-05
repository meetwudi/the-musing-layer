# API Contract Draft

This is a draft contract for a musing knowledge service.

The service is not the conversational agent. ChatGPT, Claude, or another AI surface talks with the user, explores, and compiles analysis. This service is the knowledge system underneath: a persistent, inspectable graph about becoming.

The system's main work is observation. Preservation is necessary, but comparatively easy. A rough split:

- 90% observation
- 10% preservation

Observation includes noticing recurring patterns, shifts in framing, unresolved tensions, changes in attention, and the evolution of the system's own understanding.

## Design Principles

- The service is a pure knowledge system about becoming.
- It stores raw traces, but its real work is observation.
- Most observations remain latent.
- GPT uses the service to retrieve knowledge and compose analysis.
- The service itself does not give advice, action items, therapy, or reflective essays.
- The service should hold back most of what it infers.
- Users can inspect, edit, hide, delete, and reshape the knowledge graph.
- The API should be transport-agnostic and easy to expose as MCP tools.

## Core Objects

### Trace

A trace is raw preserved material.

This is close to what we previously called a fragment, but the name is intentionally less loaded. A trace is not the main conceptual unit. It is evidence, source material, or residue.

Examples:

- A thought
- A chat excerpt
- A link
- A quote
- A question
- A tangent
- A phrase
- A contradiction
- A half-formed idea

```ts
type Trace = {
  id: string;
  space_id: string;
  author_id: string;
  content: TraceContent;
  source: TraceSource;
  created_at: string;
  updated_at: string;
  visibility: "private" | "shared";
  state: "active" | "hidden" | "deleted";
  parent_trace_id?: string;
  branch_id?: string;
  user_note?: string;
};

type TraceContent =
  | { type: "text"; text: string }
  | { type: "link"; url: string; title?: string; note?: string }
  | { type: "quote"; text: string; attribution?: string }
  | { type: "file_ref"; uri: string; title?: string };

type TraceSource = {
  client: "chatgpt" | "claude" | "local" | "api" | "other";
  channel?: string;
  conversation_id?: string;
  message_id?: string;
};
```

### Observation

An observation is the main knowledge object.

It is a structured noticing about becoming. It may be about a person, a project, a relationship, a repeated phrase, a branch of thought, an unresolved tension, or the system's own ontology.

Observations are not advice. They are not conclusions about who someone is. They are inspectable claims with provenance.

```ts
type Observation = {
  id: string;
  space_id: string;
  subject: ObservationSubject;
  kind: ObservationKind;
  statement: string;
  trace_ids: string[];
  related_observation_ids?: string[];
  confidence: number;
  maturity: "weak" | "repeated" | "stable" | "retired";
  visibility: "latent" | "surfaced" | "dismissed";
  created_at: string;
  updated_at: string;
};

type ObservationSubject =
  | { type: "person"; person_id: string }
  | { type: "project"; project_id: string }
  | { type: "relationship"; relationship_id: string }
  | { type: "branch"; branch_id: string }
  | { type: "space"; space_id: string }
  | { type: "system"; space_id: string };

type ObservationKind =
  | "recurrence"
  | "possible_connection"
  | "unresolved_tension"
  | "framing_shift"
  | "attention_shift"
  | "practice_change"
  | "loop_candidate"
  | "branch_candidate"
  | "ontology_candidate"
  | "self_observation";
```

### Relation

A relation connects traces, observations, branches, people, projects, or ontology terms.

Relations can be user-authored or system-observed. User-authored relations should carry more authority than latent system relations.

```ts
type Relation = {
  id: string;
  space_id: string;
  from: GraphRef;
  to: GraphRef;
  relation:
    | "resonates_with"
    | "contrasts_with"
    | "reframes"
    | "returns_to"
    | "branches_from"
    | "supports"
    | "complicates"
    | "supersedes";
  author: "user" | "system";
  confidence?: number;
  created_at: string;
};

type GraphRef =
  | { type: "trace"; id: string }
  | { type: "observation"; id: string }
  | { type: "branch"; id: string }
  | { type: "person"; id: string }
  | { type: "project"; id: string };
```

### Mode

A mode is a user-declared attentional contract.

The service stores modes so AI clients can retrieve and respect them. The service does not itself become the conversational enforcer; the client does that.

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

## Public Tools

These names are MCP-friendly, but the same contract can be exposed over HTTP or another transport.

### `capture_trace`

Preserve raw source material.

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
  "parent_trace_id": null,
  "branch_id": null
}
```

Response:

```json
{
  "trace_id": "trace_123",
  "captured": true
}
```

Capture is quiet. The response does not include reflection.

### `record_observation`

Record a system or client-generated observation.

This is the main write path for stewardship.

Request:

```json
{
  "space_id": "space_123",
  "subject": {
    "type": "space",
    "space_id": "space_123"
  },
  "kind": "framing_shift",
  "statement": "The user is moving away from product framing and toward a knowledge system about becoming.",
  "trace_ids": ["trace_123", "trace_456"],
  "confidence": 0.72,
  "maturity": "weak",
  "visibility": "latent"
}
```

Response:

```json
{
  "observation_id": "obs_123",
  "recorded": true
}
```

### `search_knowledge`

Retrieve traces, observations, and relations for an AI client to analyze.

Request:

```json
{
  "space_id": "space_123",
  "query": "what keeps returning around advice, slop, and stewardship?",
  "limit": 20,
  "include_latent": false
}
```

Response:

```json
{
  "traces": [
    {
      "id": "trace_123",
      "snippet": "Advice, action, or self-analysis is not where the value lives.",
      "created_at": "2026-05-05T12:00:00Z"
    }
  ],
  "observations": [
    {
      "id": "obs_123",
      "kind": "recurrence",
      "statement": "Several traces contrast useful stewardship with unsolicited advice.",
      "trace_ids": ["trace_123", "trace_456"],
      "maturity": "repeated"
    }
  ],
  "relations": []
}
```

The client may use this response to explore or compile analysis. The service does not generate the analysis.

### `inspect_observations`

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
      "statement": "Anti-slop and no-advice appear to be the same restraint principle at different layers.",
      "trace_ids": ["trace_123", "trace_456"],
      "maturity": "weak",
      "visibility": "latent"
    }
  ]
}
```

### `link_records`

Record a relation between graph records.

Request:

```json
{
  "space_id": "space_123",
  "from": {
    "type": "trace",
    "id": "trace_123"
  },
  "to": {
    "type": "observation",
    "id": "obs_456"
  },
  "relation": "supports",
  "author": "user"
}
```

Response:

```json
{
  "relation_id": "rel_123"
}
```

### `branch_trace`

Create or attach to a branch without forcing resolution.

Request:

```json
{
  "space_id": "space_123",
  "trace_id": "trace_123",
  "label": "changing how change happens"
}
```

Response:

```json
{
  "branch_id": "branch_123",
  "trace_id": "trace_123"
}
```

### `register_mode`

Set an attentional contract for clients to respect.

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

### `get_active_modes`

Return active attentional contracts for a client.

Request:

```json
{
  "space_id": "space_123",
  "conversation_id": "conv_123"
}
```

Response:

```json
{
  "modes": [
    {
      "id": "mode_123",
      "name": "no_advice",
      "instruction": "Do not provide advice unless explicitly released.",
      "scope": "conversation",
      "release_phrase": "YOU ARE FREE"
    }
  ]
}
```

### `update_record`

Edit a trace, observation, relation, or mode.

Request:

```json
{
  "space_id": "space_123",
  "record": {
    "type": "observation",
    "id": "obs_123"
  },
  "patch": {
    "maturity": "retired",
    "visibility": "dismissed"
  }
}
```

Response:

```json
{
  "updated": true
}
```

### `hide_or_delete_record`

Remove a record from normal use.

Request:

```json
{
  "space_id": "space_123",
  "record": {
    "type": "trace",
    "id": "trace_123"
  },
  "state": "hidden"
}
```

Response:

```json
{
  "state": "hidden"
}
```

## Surface Policy

The service returns knowledge records. It does not surface advice.

It may return latent observations only when:

- The client explicitly requests them.
- The user explicitly asks to inspect stewardship.
- The caller has permission to view latent knowledge.

The service should not return:

- Generated advice
- Action items
- Therapy or mental health interpretation
- Reflective essays
- Psychological interpretation as fact
- Product framing unless requested as retrieved knowledge

## Minimal ChatGPT Flow

1. User muses in ChatGPT.
2. ChatGPT calls `capture_trace`.
3. The service stores the trace.
4. The service or client records latent observations through `record_observation`.
5. ChatGPT calls `search_knowledge` when it needs context.
6. ChatGPT uses the retrieved knowledge to explore or compile analysis.
7. If the user asks to see the silent graph, ChatGPT calls `inspect_observations`.
8. The user can edit, hide, delete, or reshape records.

## Non-Goals

- Not a therapy API.
- Not a productivity task system.
- Not a generic chatbot memory store.
- Not an advice engine.
- Not a reflection generator.
- Not an autonomous agent that decides what matters.

