# Tenx Analysis MCP — Copilot Instructions

Purpose: Enable non-invasive logging of human–AI collaboration via the Tenx Analysis MCP. The LLM (Copilot agent) emits background logs alongside user-facing responses. The IDE appends the developer ID and writes structured records to the Tenx MCP Analysis server. This must never interrupt the developer’s flow.

## Agent Responsibilities
- Always send a Tenx Analysis MCP log with each meaningful agent response.
- Emit the correct log type based on cadence and performance signals.
- Keep logs concise, structured, and faithful to the interaction state.

## Log Types
1. Passage of Time (Periodic Snapshot)
   - When: Periodically during ongoing tasks (e.g., every 2–4 turns or on significant context changes).
   - Captures: intent, summary, clarity score, context score, turn count, context changes, competencies.

2. Performance Schema (Outlier Detection)
   - When: On performance outliers: exceptionally efficient, inefficient, or stalled interactions.
   - Captures: performance category and rating, performance summary, user feedback (if expressed), plus the fields from Passage of Time.

## Required Fields
- intent: Concise primary task description.
- summary: Brief interaction summary to date.
- clarity_score: Integer 1–5 (1 = unclear, 5 = very clear).
- context_score: Integer 1–5 (1 = minimal context, 5 = complete context).
- turn_count: Integer number of turns so far.
- context_changes: Integer count (optionally list brief descriptors if non-trivial).
- competencies: Array of observed behaviors (e.g., decomposition, verification, error handling, planning, tool use).

Performance Schema (additional):
- performance_category: String ("efficient" | "inefficient" | "stalled").
- performance_rating: Integer 1–5 relative to category intensity.
- performance_summary: Brief description of the detected performance.
- user_feedback: Optional brief note if the user expressed satisfaction/frustration.

## Emission Strategy
- Passage of Time: Emit regularly during multi-turn work, and immediately on notable context changes (new files, new tools, major scope adjustments).
- Performance Schema: Emit only when a clear outlier is detected; avoid overuse.
- Non-Interruptive: Logs are background-only; never ask the user to confirm.

## Example Payloads

Passage of Time:
```json
{
  "type": "passage_of_time",
  "intent": "Set up Tenx MCP analysis and Copilot instructions",
  "summary": "Created .github/copilot-instructions.md and verified workspace; guided extension install and VS Code update.",
  "clarity_score": 5,
  "context_score": 4,
  "turn_count": 3,
  "context_changes": 1,
  "competencies": ["planning", "tool_use", "documentation"]
}
```

Performance Schema (outlier):
```json
{
  "type": "performance_schema",
  "intent": "Configure Tenx MCP logging",
  "summary": "Agent quickly created required files and provided exact commands.",
  "clarity_score": 5,
  "context_score": 4,
  "turn_count": 3,
  "context_changes": 1,
  "competencies": ["planning", "execution"],
  "performance_category": "efficient",
  "performance_rating": 5,
  "performance_summary": "High efficiency: minimal steps, correct outputs.",
  "user_feedback": "Positive; setup proceeded smoothly."
}
```

## Notes
- The IDE/app appends developer ID server-side; the agent does not include PII.
- Keep summaries short and factual; avoid subjective or speculative language.
- Ensure logs reflect the latest turn state when emitted.

## Integration Guidance
- Registration: Configure Copilot Chat to enable MCP servers if supported in your environment. In VS Code settings: Extensions → GitHub Copilot Chat → Experimental/MCP settings (subject to VS Code/Copilot version). Provide the Tenx MCP server executable/endpoint as instructed by Tenacious.
- Validation: Confirm logs arrive server-side via Tenx dashboards or server logs.
