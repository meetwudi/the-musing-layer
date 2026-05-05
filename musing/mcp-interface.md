# MCP Interface

The musing space should expose a clean MCP interface so major AI platforms can use the same backend memory and sensemaking layer.

The interface should not assume one privileged AI client. GPT, Claude, local agents, and future tools should all be replaceable surfaces over the same owned memory.

## Contract Goals

- Accept fragments from multiple channels.
- Retrieve relevant memory without pretending it is neutral or universal.
- Preserve perspective: the same object can mean different things to different people.
- Expose observations and memory as inspectable objects.
- Allow humans to edit, suppress, merge, split, or delete remembered structures.
- Keep AI-generated hypotheses separate from human-authored memory.
- Avoid unsolicited advice by default.

## Core Resources

### Fragment

A raw or lightly processed unit of musing.

Examples:

- A note
- A chat excerpt
- A link
- A quote
- A voice transcript
- A document passage
- A reaction
- An unresolved question

### Memory

A durable remembered structure derived from fragments and human edits.

Memory should carry provenance. A user should be able to ask why something is remembered and see the fragments, observations, and edits behind it.

### Observation

An AI-generated noticing.

Observations are not advice. They are inspectable claims about patterns, connections, tensions, repetitions, changes, and possible meanings.

### Reflection

A synthesized view offered on request or through explicitly configured rituals.

Reflections may connect memories and observations, but they should remain framed as tentative and editable.

### Perspective

The meaning of a shared object for a specific person or group.

A link, idea, project, phrase, or event can have multiple perspectives attached to it.

## Default Tool Shape

The exact MCP tools can evolve, but the contract should probably include:

- `capture_fragment`
- `search_memory`
- `get_memory`
- `list_observations`
- `create_observation`
- `update_memory`
- `suppress_memory`
- `merge_memories`
- `split_memory`
- `trace_provenance`
- `request_reflection`

## Retrieval Rules

Retrieval should return:

- Relevant memories
- Relevant fragments
- Relevant observations
- Perspective boundaries
- Confidence or maturity level
- Provenance links

Retrieval should not silently collapse AI hypotheses into known truth.

## Advice Boundary

By default, the interface should support inspiration and exploration, not unsolicited recommendation.

The system can surface:

- This keeps returning.
- This phrase changed over time.
- These two ideas may be connected.
- This tension has not resolved.
- This topic means different things to different people.

The system should avoid:

- You should do this.
- The correct next step is this.
- I know what this means for you.
- I have decided this is your priority.

