# Question authoring MCP reference

## Host setup

MCP endpoint: `https://staging.zyxacademy.com/api/mcp/authoring`

- Claude Desktop or Claude Code: install the `zyx-authoring-mcp` plugin, which carries `.mcp.json` and the skills, or add a remote connector with the URL above. Zyx admin OAuth opens automatically on hosts that support it.
- ChatGPT: enable Developer mode in Settings, add an MCP-type connector with the URL above, then authenticate via OAuth. If the host does not support interactive OAuth, ask a Zyx admin to create a short-lived connection token via `POST /api/mcp/authoring/connection` while an admin session is active, and use it as a Bearer token. Paste the SKILL.md body into custom GPT or project instructions so the tool flow is followed; drop the YAML frontmatter because only the Claude skill loader needs it, and replace the `references/workflow.md` link in the first step with this file when you paste it too.
- Never store a connection token in configuration files or repositories.

## Tool sequence

```text
workflow.list
catalog.list_courses
catalog.list_chapters { courseKey }
workflow.start { workflow: "quiz_bank", courseKey, chapterKey }
workflow.get_contract { runToken }
assessment.list_ideas { runToken, query?, knowledgeKinds?, instructionalRoles?, difficultyLevels?, limit?, offset? }
assessment.get_idea { runToken, ideaId }
assessment.validate_quiz_draft { runToken, draftJson }   // repeat until valid
assessment.submit_quiz_draft { runToken, draftJson }     // requires authoring:stage scope
```

`workflow.start` without `sourcePackToken` is valid for quiz_bank. With an optional Source Pack, run `source.ingest` first and then include `sourcePackToken`.

## Idea catalog

The `assessment.list_ideas` response contains:

- `scope`: labels of the locked course and chapter.
- `totalIdeaCount` and `ideaCount`: total published ideas in scope and the number of filtered results.
- `ideas`: one page of items with id, code, statement, explanation, latex, knowledgeKind, instructionalRole, difficultyLevel, version, semanticHash.
- `facets`: counts per knowledgeKind, instructionalRole, and difficultyLevel to refine the next iteration.
- `paging.nextOffset`: send it as the next `offset` when it is not null.

Filters combine with AND. Use `query` for free text search over code, statement, explanation, and latex.

`assessment.get_idea` adds active relations (outgoing and incoming directions plus the partner Idea code) and a provenance summary. Use prerequisite relations to make sure a question does not require Ideas that have not been taught yet.

## Draft contract

Schema `question-bank-draft.v2`, maximum 500 questions:

```json
{
  "schemaVersion": "question-bank-draft.v2",
  "draftId": "draft-2026-08-21-kalkulus-bab1",
  "questions": [
    {
      "id": "q-limit-01",
      "questionType": "multiple_choice",
      "difficulty": "medium",
      "cognitiveLevel": "apply",
      "reasoningPattern": "direct_application",
      "tags": ["limit-barisan"],
      "prompt": "Question text.",
      "options": ["Option A", "Option B", "Option C", "Option D"],
      "correctIndices": [1],
      "acceptableAnswers": [],
      "explanation": "Full worked explanation.",
      "ideaLinks": [
        { "ideaId": "<from catalog>", "weight": 0.7, "role": "primary" },
        { "ideaId": "<from catalog>", "weight": 0.3, "role": "supporting" }
      ]
    }
  ]
}
```

Allowed values:

| Field | Values |
|---|---|
| questionType | multiple_choice, multiple_choices, short_answer, essay |
| difficulty | easy, medium, hard |
| cognitiveLevel | remember, understand, apply, analyze, evaluate, create |
| ideaLinks[].role | primary, supporting, required |
| tags | short lowercase strings, consistent across questions |

Linking rules:

1. Links are optional but strongly recommended; a question without links remains valid.
2. Positive weights across all ideaLinks of one question must sum to exactly 1.
3. Ideas must be published and belong to the same course and chapter as the run.
4. An Idea must not appear twice in the same question.

## Validation and submit

`assessment.validate_quiz_draft` returns all findings in a single call, not one at a time:

- `issues`: blocking, must be zero before submit.
- `warnings`: for example QUESTION_COVERAGE_GAP for Ideas not yet covered and QUESTION_EXPLANATION_EMPTY.
- `stats`: distribution of difficulty, cognitive level, question type, tag frequency, and coverage per Idea.

Fix the draft, validate again, then submit. The submit response contains `createdCount`, `existingCount`, `questionIds`, remaining warnings, and `next: "admin_review"`. No MCP tool can publish; publication is performed by admins only.

## Error codes and actions

| Code | Meaning | Action |
|---|---|---|
| AUTHORING_MCP_WORKFLOW_MISMATCH | Run is not quiz_bank | Start a new run with the quiz_bank workflow |
| AUTHORING_MCP_IDEA_NOT_IN_SCOPE | Idea not published or out of scope | Pick again from assessment.list_ideas |
| QUESTION_BANK_DRAFT_DUPLICATE_ID | Duplicate question IDs in the draft | Give each question a unique ID |
| QUESTION_AUTHORING_INVALID_QUESTION | Incomplete question structure | Follow the message, for example at least two options |
| QUESTION_AUTHORING_DUPLICATE_ANSWER | Duplicate answer key | One unique index for multiple_choice |
| QUESTION_AUTHORING_DUPLICATE_IDEA | Same Idea twice in one question | Remove the duplicate |
| QUESTION_AUTHORING_IDEA_WEIGHT_INVALID | Total weight is not 1 | Normalize positive weights to 1 |
| QUESTION_AUTHORING_IDEA_NOT_FOUND | ideaId does not exist | Copy again from the latest catalog |
| QUESTION_AUTHORING_IDEA_NOT_PUBLISHED | Idea not published yet | Replace with another published Idea |
| QUESTION_AUTHORING_IDEA_COURSE_MISMATCH | Idea from another course | Pick from the scoped catalog |
| QUESTION_AUTHORING_IDEA_CHAPTER_MISMATCH | Idea from another chapter | Pick from the scoped catalog |
| AUTHORING_MCP_TOKEN_EXPIRED | Run token expired (7 days) | Run workflow.start again |
| AUTHORING_MCP_SCOPE_REQUIRED | Missing scope token | Reconnect with authoring:stage to submit |

A staged draft gets status generated and is reviewed by admins in the question bank review flow.
