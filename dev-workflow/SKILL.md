---
name: dev-workflow
description: Complete software development workflow orchestrating requirements analysis, technical proposals, code implementation, testing, and review. Use when the user invokes /dev-workflow command or requests a complete end-to-end development process from requirements to tested code. Automatically integrates openspec for requirements and proposals, codex for implementation and testing, and ui-ux-pro-max for frontend development when needed.
---

# Dev Workflow

Complete end-to-end software development workflow that orchestrates multiple specialized tools to deliver production-ready code from requirements to review.

## Workflow Overview

This skill executes a five-stage automated workflow:

1. **Requirements Understanding** - Analyze and clarify requirements using openspec
2. **Proposal Creation** - Generate technical proposal with openspec
3. **Code Development** - Implement solution using codex (with ui-ux-pro-max for frontend)
4. **Testing** - Execute tests using codex
5. **Code Review** - Perform review using codex

## Execution Process

### Stage 1: Requirements Understanding

Use openspec to analyze the user's requirements:

```
Call the openspec skill with the user's requirements to:
- Clarify ambiguous requirements
- Identify technical constraints
- Document functional and non-functional requirements
- Establish success criteria
```

**Output:** Structured requirements document

### Stage 2: Proposal Creation & Review

Use openspec to create a technical proposal:

```
Call the openspec skill to:
- Design system architecture
- Identify technology stack
- Plan implementation approach
- Document API contracts and data models
- CRITICAL: Explicitly mark whether frontend development is required
```

**Frontend Detection:** The proposal MUST clearly indicate if the implementation involves:
- User interfaces (UI/UX)
- Web pages or components
- Frontend frameworks (React, Vue, etc.)
- Client-side interactions

**Output:** Technical proposal with frontend flag

### Stage 3: Code Development

Route to appropriate implementation tool based on proposal:

**If frontend development is required:**
```
1. Call ui-ux-pro-max skill for frontend implementation
2. Call codex for backend implementation (if applicable)
```

**If no frontend development:**
```
Call codex with:
- Prompt: "Implement the following technical proposal: [proposal content]"
- fullAuto: true (enable sandboxed automatic execution)
- sandbox: "workspace-write" (allow writes in workspace only)
```

**Output:** Implemented code

### Stage 4: Testing

Use codex to execute tests:

```
Call codex with:
- Prompt: "Run all tests for the implemented code and fix any failures"
- fullAuto: true
- sandbox: "workspace-write"
- sessionId: [use same session from Stage 3]
```

**Output:** Test results and fixes

### Stage 5: Code Review

Use codex review functionality:

```
Call codex review with:
- uncommitted: true (review working tree changes)
- prompt: "Review the implemented code for quality, security, and best practices"
```

**Output:** Code review report

## Error Handling

If any stage fails:
1. Report the failure to the user with details
2. Ask if they want to:
   - Retry the failed stage
   - Modify requirements and restart
   - Continue manually from the failure point

## Session Management

- Maintain codex session continuity between development and testing stages
- Use the same sessionId for Stages 3 and 4 to preserve context
- Start fresh sessions for review to ensure objective analysis

## Best Practices

1. **Always validate proposal completeness** before proceeding to implementation
2. **Preserve session context** between related stages (dev → test)
3. **Use appropriate sandbox levels** - workspace-write for development, read-only for review
4. **Frontend detection is mandatory** - never skip the frontend check in proposals
5. **Full automation** - use fullAuto mode for seamless execution

## Example Usage

**User command:**
```
/dev-workflow Create a blog system with user authentication and post management
```

**Execution flow:**
1. openspec analyzes requirements → identifies need for auth, CRUD, database
2. openspec creates proposal → marks frontend=true (admin UI needed)
3. ui-ux-pro-max implements frontend → admin dashboard
4. codex implements backend → API endpoints, database, auth
5. codex runs tests → validates functionality
6. codex reviews code → provides quality report

## Additional Resources

For detailed guidance on each stage, see:
- [references/workflow-guide.md](references/workflow-guide.md) - Detailed stage-by-stage instructions
- [references/examples.md](references/examples.md) - Complete workflow examples for common scenarios
