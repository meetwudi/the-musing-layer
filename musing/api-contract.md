# API Contract Draft

This is a draft contract for a musing knowledge service.

The service is not the conversational agent. ChatGPT, Claude, or another AI surface talks with the user, explores, and compiles analysis. This service is the knowledge system underneath: a persistent, inspectable graph about becoming.

The system's main work is observation. Preservation is necessary, but comparatively easy. A rough split:

- 90% observation
- 10% preservation

Observation does not need to be a first-class product object. In this draft, observation is expressed through ontology-governed assertions over traces.

## Core Idea

Use an RDF-like mindset:

- **Terminology** describes what the system currently cares about.
- **Assertions** say things using that terminology.
- **Traces** provide source material and provenance.
- **Named graphs** or assertion groups hold context, authorship, confidence, visibility, and evolution.

The ontology should stay dynamic. Over-specifying the core ontology too early creates bias. The system should begin with a small vocabulary and let new terms emerge, stabilize, merge, or retire as the musing space changes what it cares about.

## Design Principles

- The service is a pure knowledge system about becoming.
- GPT uses the service to retrieve knowledge and compile analysis.
- The service itself does not give advice, action items, therapy, or reflective essays.
- The ontology is the living set of things the system cares about.
- Assertions are the main knowledge unit.
- Relations are just predicates in the ontology.
- Raw traces are evidence, not the center of meaning.
- Most inferred assertions remain latent.
- Users can inspect, edit, hide, delete, and reshape traces, terms, and assertions.
- The API should be transport-agnostic and easy to expose as MCP tools.

## Recommendation

Use RDF as the mental model, but expose a practical JSON contract first.

Raw RDF triples are elegant, but product code will need provenance, confidence, maturity, visibility, source traces, authorship, and edit state. A simple assertion wrapper around RDF-style statements is probably more useful than exposing raw triples everywhere.

JSON-LD can be a good serialization later. SPARQL can be useful for advanced querying later. The early contract should stay readable to AI clients and humans.

## Core Objects

### Trace

A trace is raw preserved material.

It is evidence, source material, or residue. It is close to what we previously called a fragment, but less conceptually loaded.

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

### Term

A term is part of the system's current terminology.

Terms describe what the system is prepared to notice. They should be self-descriptive enough that a client can understand what the system wants to capture or assert.

```ts
type Term = {
  id: string;
  space_id: string;
  iri: string;
  label: string;
  kind: "class" | "predicate" | "property";
  description: string;
  examples?: string[];
  anti_examples?: string[];
  capture_guidance?: string;
  status: "candidate" | "stable" | "retired";
  introduced_by: "user" | "system";
  evidence_trace_ids?: string[];
  created_at: string;
  updated_at: string;
};
```

Examples of terms:

- `musing:RecurringQuestion`
- `musing:UnresolvedTension`
- `musing:FramingShift`
- `musing:AttentionPattern`
- `musing:returnsTo`
- `musing:reframes`
- `musing:complicates`

These are examples, not a fixed core ontology.

### Assertion

An assertion is a knowledge statement.

It uses ontology terms to say something about traces, people, projects, branches, other assertions, or the system's own ontology.

```ts
type Assertion = {
  id: string;
  space_id: string;
  subject: GraphRef;
  predicate: string;
  object: GraphValue;
  evidence_trace_ids: string[];
  graph_id?: string;
  author: "user" | "system" | "client";
  confidence?: number;
  maturity: "weak" | "repeated" | "stable" | "retired";
  visibility: "latent" | "surfaced" | "dismissed";
  created_at: string;
  updated_at: string;
};

type GraphRef =
  | { type: "trace"; id: string }
  | { type: "person"; id: string }
  | { type: "project"; id: string }
  | { type: "branch"; id: string }
  | { type: "term"; id: string }
  | { type: "assertion"; id: string }
  | { type: "space"; id: string };

type GraphValue =
  | GraphRef
  | { type: "literal"; value: string; datatype?: string };
```

This removes the separate `Relation` object. A relation is just an assertion whose predicate is relational.

It also removes the separate `Observation` object. An observation is one or more assertions with provenance.

### Graph

A graph groups assertions under a context.

```ts
type Graph = {
  id: string;
  space_id: string;
  label?: string;
  purpose: "latent_stewardship" | "user_visible" | "ontology" | "import";
  visibility: "latent" | "surfaced";
  created_at: string;
  updated_at: string;
};
```

## Public Tools

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

### `get_terms`

Return the current terminology so a client can understand what the system cares about.

Request:

```json
{
  "space_id": "space_123",
  "status": ["candidate", "stable"]
}
```

Response:

```json
{
  "terms": [
    {
      "id": "term_123",
      "iri": "musing:UnresolvedTension",
      "label": "Unresolved tension",
      "kind": "class",
      "description": "A live contradiction, tradeoff, or pressure that keeps returning without resolution.",
      "examples": ["I want this to be practical, but not productized too early."],
      "capture_guidance": "Look for repeated contrast, hesitation, or pressure between two live values.",
      "status": "candidate"
    }
  ]
}
```

### `upsert_term`

Create or update a terminology term.

Request:

```json
{
  "space_id": "space_123",
  "term": {
    "iri": "musing:AttentionCollapse",
    "label": "Attention collapse",
    "kind": "class",
    "description": "A moment where open musing collapses into advice, action, self-analysis, or product framing.",
    "examples": ["Turning an open thought into a task list too early."],
    "status": "candidate",
    "introduced_by": "system",
    "evidence_trace_ids": ["trace_123"]
  }
}
```

Response:

```json
{
  "term_id": "term_456",
  "status": "candidate"
}
```

### `assert_statements`

Write assertions into the graph.

Request:

```json
{
  "space_id": "space_123",
  "graph_id": "graph_latent",
  "assertions": [
    {
      "subject": { "type": "trace", "id": "trace_123" },
      "predicate": "musing:expresses",
      "object": { "type": "term", "id": "term_attention_collapse" },
      "evidence_trace_ids": ["trace_123"],
      "author": "system",
      "confidence": 0.68,
      "maturity": "weak",
      "visibility": "latent"
    }
  ]
}
```

Response:

```json
{
  "assertion_ids": ["assert_123"]
}
```

### `query_graph`

Retrieve traces, terms, and assertions for a client to analyze.

Request:

```json
{
  "space_id": "space_123",
  "query": "what keeps returning around advice, slop, and stewardship?",
  "include_latent": false,
  "limit": 30
}
```

Response:

```json
{
  "traces": [
    {
      "id": "trace_123",
      "snippet": "Advice, action, or self-analysis is not where the value lives."
    }
  ],
  "terms": [
    {
      "id": "term_456",
      "iri": "musing:AttentionCollapse",
      "label": "Attention collapse"
    }
  ],
  "assertions": [
    {
      "id": "assert_123",
      "subject": { "type": "trace", "id": "trace_123" },
      "predicate": "musing:expresses",
      "object": { "type": "term", "id": "term_456" },
      "maturity": "weak"
    }
  ]
}
```

The client compiles analysis from this knowledge. The service does not generate the analysis.

### `inspect_latent_graph`

Return latent assertions when the user explicitly asks to inspect stewardship.

Request:

```json
{
  "space_id": "space_123",
  "query": "show me latent assertions around anti-slop",
  "limit": 20
}
```

Response:

```json
{
  "assertions": [
    {
      "id": "assert_123",
      "predicate": "musing:resonatesWith",
      "subject": { "type": "term", "id": "term_anti_slop" },
      "object": { "type": "term", "id": "term_no_advice" },
      "confidence": 0.61,
      "maturity": "weak"
    }
  ]
}
```

### `update_record`

Edit a trace, term, assertion, or graph.

Request:

```json
{
  "space_id": "space_123",
  "record": {
    "type": "assertion",
    "id": "assert_123"
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

## How Clients Decide What Counts As A Trace

This remains an open question.

The ontology can help. Terms should include `capture_guidance`, examples, and anti-examples so clients know what the system is currently trying to notice.

Possible trace policies:

- Capture only explicit user-marked material.
- Capture every user message as a trace.
- Capture selected spans from messages when they match active terminology.
- Capture external artifacts such as links, notes, files, or quotes.
- Capture a coarse message first, then let assertions point to smaller spans later.

The least biased default may be:

1. Capture user-marked material exactly.
2. Capture full messages when the client is unsure.
3. Let assertions do the interpretive work.
4. Avoid over-splitting traces too early.

## Ontology Evolution

Ontology changes are themselves part of becoming.

The system should be able to assert things about its own terminology:

- A term was introduced.
- A term became useful.
- A term was merged.
- A term was retired.
- A term changed meaning.
- A term reflects a newly stable pattern of care.

This keeps the system's own evolution inspectable.

## Surface Policy

The service returns knowledge records. It does not surface advice.

It may return latent assertions only when:

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
2. ChatGPT calls `get_terms` to understand the current terminology.
3. ChatGPT calls `capture_trace` for source material.
4. ChatGPT or the service writes latent `assertions` using `assert_statements`.
5. ChatGPT calls `query_graph` when it needs context.
6. ChatGPT uses the retrieved knowledge to explore or compile analysis.
7. If the user asks to see the silent graph, ChatGPT calls `inspect_latent_graph`.
8. The user can edit, hide, delete, or reshape traces, terms, and assertions.

## Non-Goals

- Not a therapy API.
- Not a productivity task system.
- Not a generic chatbot memory store.
- Not an advice engine.
- Not a reflection generator.
- Not an autonomous agent that decides what matters.

