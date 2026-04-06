# DocuBot Model Card

## System Overview
DocuBot is a retrieval-augmented assistant that answers developer questions
using local documentation files. It supports three modes: naive LLM generation,
retrieval only, and full RAG (retrieval + generation).

## Mode Comparisons

### Question: "Where is the auth token generated?"

| Mode | Answer Quality | Notes |
|------|---------------|-------|
| Naive LLM | Poor | Hallucinated — mentioned Auth0, Okta, AWS with no basis in docs |
| Retrieval Only | Good | Returned accurate AUTH.md snippets, but raw and hard to interpret |
| RAG | Best | Concise, grounded, cited AUTH.md correctly |

### Question: "Which endpoint returns all users?"

| Mode | Answer Quality | Notes |
|------|---------------|-------|
| Naive LLM | Accidentally correct | Got lucky — this info exists in general web knowledge |
| Retrieval Only | Good | Found right snippets from API_REFERENCE.md and DATABASE.md |
| RAG | Good | Clean answer but missed the explicit endpoint name GET /api/users |

## Key Observations

**Where naive generation fails:**
The model answered the auth token question with confident, detailed information
about Auth0, Okta, and AWS — none of which appear anywhere in the docs.
Fluent output is not the same as accurate output.

**Where retrieval only falls short:**
Retrieval returns raw text chunks that are accurate but require the user to
interpret them. It also has no ability to synthesize across multiple snippets.

**Where RAG works best:**
RAG combines the accuracy of retrieval with the readability of generation.
Answers are grounded in real snippets and cite their sources.

**Where RAG still fails:**
RAG can miss specific details if the retrieved snippet contains the context
but not the exact answer. Example: retrieved "returns a list of all users,
admin only" but missed stating the actual endpoint GET /api/users.

## Guardrail Behavior
DocuBot refuses to answer when no snippets score above the minimum threshold.
Example: "What's the weather like?" correctly returns "I do not know based
on these docs." This prevents confident but unsupported answers.

## Design Decisions
- Chunking by paragraph (double newline) kept snippets focused
- min_score=3 threshold balanced precision without blocking valid queries
- RAG prompt instructs model to cite source files and avoid invention