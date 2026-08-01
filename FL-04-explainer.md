# FL-04: Workflows, Agents, and MCP — Explainer

## Workflow vs. agent

Anthropic draws the line at who's holding the map. In a **workflow**, a developer writes the route ahead of time — call model A, pass its output to model B, check a condition, branch left or right — and the LLM just executes each step when it's handed the wheel. Control flow lives in code, not in the model's judgment. In an **agent**, the model decides the route as it goes: it looks at the result of its last action, picks what to do next, calls a tool, observes the result, and keeps looping until it judges the task done or hits a stopping condition. The defining feature isn't "does it use an LLM more than once" — a workflow can chain ten LLM calls and still be a workflow. It's whether a human pre-wired the sequence of steps, or the model is choosing that sequence itself based on what it observes.

The "right" choice depends on whether the number and order of steps is predictable ahead of time. If you can say "step 2 always follows step 1, and there are always five steps," a workflow gives you predictability and is cheaper and easier to debug. If the number of steps genuinely can't be known in advance — a coding agent has no idea how many files a bug touches until it starts reading code — hardcoding a fixed path isn't possible, and you need an agent.

## FL-04 classification

My FL-04 build (project notes → fact-checked LinkedIn post, in n8n) is a **workflow**, specifically a mix of two named patterns: **prompt chaining** with a **gate**. The five steps are fixed and always run in the same order: gather → synthesize → draft → critique → revise/format. The critique step behaves as a gate — checking the draft against a `pass`/`issues` structure — but the branch it triggers (revise-once, then stop) is hardcoded in the n8n graph, not decided by the model. Nothing in the pipeline chooses a different tool, loops an unbounded number of times, or changes its own plan based on what it discovers. I designed the path; the model just fills in each step along it.

## What MCP is

MCP (Model Context Protocol) is the standard that lets an AI application connect to external tools and data without a bespoke integration for every pairing of app and service. Before MCP, connecting an LLM app to N services meant custom glue code for each one; MCP replaces that with one client-server protocol, so any MCP-compatible client can talk to any MCP-compatible server. A **host** (like Claude) runs an MCP **client**, which connects to one or more **servers**, each exposing some mix of three primitives:

- **Tools** — model-controlled functions the LLM decides to call to take an action (e.g., search Gmail, create a draft), similar to function calling.
- **Resources** — application-controlled data pulled into context, roughly a GET endpoint: read-only, no side effects.
- **Prompts** — user-controlled templates that pre-package a good way to use a tool or resource, chosen before inference starts.

The distinction that matters: tools are model-initiated, resources are application-supplied, prompts are user-chosen upfront.

## MCP demonstration

I connected the Gmail MCP connector already available to my Claude account and ran three tasks chat alone has no way to do, since each requires live access to a real account:

1. **list_labels** — pulled real label counts (e.g., 945 unread across 1,071 inbox threads at the time of the run). No conversation with a model can know this; it's live account state.
2. **search_threads** with a Gmail-syntax query (`is:unread newer_than:3d`) — returned actual thread IDs, senders, subjects, timestamps from my inbox.
3. **create_draft** — wrote a real draft into my Gmail Drafts folder and returned its draft ID, an action with an external side effect chat alone cannot produce.

All three appear in the conversation as explicit tool-call blocks carrying real returned data, not generated text — the visible evidence of tool use versus plain chat.

## What FL-04 would need to become an agent

Right now the critique step can only do one thing when it fails: trigger a single hardcoded revise call, then stop, pass or fail. To make it an agent, I'd remove that fixed branch and give the model a small toolset — `revise_draft`, `pull_more_context`, `flag_for_human`, `mark_done` — and let it decide, based on what the critique actually says, which tool to call and how many times. A missing-metric flag might trigger a re-pull of the original project notes rather than a blind revision; voice drift might trigger one `revise_draft` and a re-check; something unresolved after a couple of attempts calls `flag_for_human` instead of looping forever. The model needs to see each tool's actual output as "ground truth" before deciding its next move — that feedback loop is what's missing now, since every node's output currently goes to a predetermined next node regardless of content.

The tradeoff is real: an agent version costs more (potentially many calls per run instead of a fixed four), is harder to predict, and needs a stopping condition. For a low-stakes, well-defined task like this — five fixed steps, clear success criteria, cheap to re-run — the workflow is plausibly still the right choice. The honest upgrade isn't "make everything agentic," it's recognizing only the critique-to-revision loop has genuinely unpredictable structure, and scoping agent behavior to that piece rather than rebuilding the whole pipeline.

*(Body word count, excluding headers: ~850)*
