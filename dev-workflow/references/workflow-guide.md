# Workflow Guide

Detailed stage-by-stage instructions for executing the dev-workflow.

## Table of Contents

1. [Stage 1: Requirements Understanding](#stage-1-requirements-understanding)
2. [Stage 2: Proposal Creation](#stage-2-proposal-creation)
3. [Stage 3: Code Development](#stage-3-code-development)
4. [Stage 4: Testing](#stage-4-testing)
5. [Stage 5: Code Review](#stage-5-code-review)
6. [Troubleshooting](#troubleshooting)

## Stage 1: Requirements Understanding

### Objective
Transform user's initial request into structured, unambiguous requirements.

### Process

1. **Invoke openspec skill:**
   ```
   Use Skill tool with:
   - skill: "openspec"
   - args: "analyze requirements: [user's request]"
   ```

2. **What openspec will do:**
   - Ask clarifying questions about ambiguous aspects
   - Identify functional requirements (what the system must do)
   - Identify non-functional requirements (performance, security, scalability)
   - Document constraints (technology, timeline, resources)
   - Define success criteria

3. **Expected output:**
   - Structured requirements document
   - Clear scope boundaries
   - Identified dependencies
   - Risk assessment

### Success Criteria
- All ambiguities resolved
- Requirements are testable
- Scope is clearly defined
- User has confirmed understanding

## Stage 2: Proposal Creation

### Objective
Create a detailed technical proposal that guides implementation.

### Process

1. **Invoke openspec skill for proposal:**
   ```
   Use Skill tool with:
   - skill: "openspec"
   - args: "create technical proposal based on requirements: [requirements doc]"
   ```

2. **What openspec will do:**
   - Design system architecture
   - Select appropriate technology stack
   - Define API contracts and interfaces
   - Plan data models and schemas
   - Identify implementation phases
   - **CRITICAL:** Mark frontend requirement flag

3. **Frontend detection checklist:**
   The proposal MUST explicitly state if ANY of these apply:
   - [ ] User interface components needed
   - [ ] Web pages or views required
   - [ ] Client-side interactions (forms, buttons, navigation)
   - [ ] Frontend framework usage (React, Vue, Angular, etc.)
   - [ ] Styling or visual design needed

4. **Expected output:**
   - Architecture diagram or description
   - Technology stack with justification
   - API specifications
   - Data models
   - Implementation plan
   - **Frontend flag: YES/NO**

### Success Criteria
- Architecture is sound and scalable
- Technology choices are justified
- All interfaces are documented
- Frontend requirement is explicitly marked
- Proposal is implementable

## Stage 3: Code Development

### Objective
Implement the technical proposal into working code.

### Decision Point: Frontend Required?

**Check the proposal's frontend flag:**

#### Path A: Frontend Required (frontend=YES)

1. **Frontend implementation:**
   ```
   Use Skill tool with:
   - skill: "ui-ux-pro-max"
   - args: "implement frontend based on proposal: [frontend specs from proposal]"
   ```

2. **Backend implementation (if applicable):**
   ```
   Use mcp__codex-cli__codex with:
   - prompt: "Implement backend API based on proposal: [backend specs from proposal]"
   - fullAuto: true
   - sandbox: "workspace-write"
   - workingDirectory: [project root]
   ```

3. **Integration:**
   ```
   Use mcp__codex-cli__codex with:
   - prompt: "Integrate frontend with backend API endpoints"
   - fullAuto: true
   - sandbox: "workspace-write"
   - sessionId: [backend session ID]
   ```

#### Path B: No Frontend (frontend=NO)

```
Use mcp__codex-cli__codex with:
- prompt: "Implement the following technical proposal: [full proposal]"
- fullAuto: true
- sandbox: "workspace-write"
- workingDirectory: [project root]
```

### Important Notes

- **Save the sessionId** from codex response for Stage 4
- **fullAuto mode** enables automatic command execution without approval prompts
- **workspace-write sandbox** allows file writes only within the workspace
- **Working directory** should be set to the project root

### Expected Output
- Implemented code matching proposal specifications
- Proper project structure
- Dependencies installed
- Configuration files created

## Stage 4: Testing

### Objective
Validate the implementation through automated tests.

### Process

1. **Run tests using codex:**
   ```
   Use mcp__codex-cli__codex with:
   - prompt: "Run all tests for the implemented code. If tests fail, analyze failures and fix the code. Re-run tests until all pass."
   - fullAuto: true
   - sandbox: "workspace-write"
   - sessionId: [session ID from Stage 3]
   - workingDirectory: [project root]
   ```

2. **What codex will do:**
   - Discover and run existing tests
   - If no tests exist, create appropriate tests
   - Analyze test failures
   - Fix code issues
   - Re-run tests iteratively
   - Report final test results

### Success Criteria
- All tests pass
- Code coverage is adequate
- Edge cases are tested
- No critical bugs remain

### If Tests Fail Repeatedly

If codex cannot fix failing tests after multiple attempts:
1. Report the issue to the user
2. Provide test failure details
3. Ask if they want to:
   - Continue with manual intervention
   - Revise the proposal
   - Skip testing (not recommended)

## Stage 5: Code Review

### Objective
Perform quality and security review of the implemented code.

### Process

1. **Run codex review:**
   ```
   Use mcp__codex-cli__review with:
   - uncommitted: true
   - prompt: "Review the implemented code for: (1) Code quality and best practices, (2) Security vulnerabilities, (3) Performance issues, (4) Maintainability concerns, (5) Documentation completeness"
   - workingDirectory: [project root]
   ```

2. **What codex review will do:**
   - Analyze code changes
   - Check for security vulnerabilities (SQL injection, XSS, etc.)
   - Identify code smells and anti-patterns
   - Verify best practices adherence
   - Assess documentation quality
   - Provide actionable recommendations

### Expected Output
- Comprehensive review report
- Security findings (if any)
- Code quality issues
- Improvement recommendations
- Overall assessment

### Post-Review Actions

Based on review findings:
- **Critical issues found:** Fix immediately and re-review
- **Minor issues found:** Report to user, ask if they want fixes
- **No issues found:** Workflow complete, present final deliverables

## Troubleshooting

### Common Issues

#### Issue: openspec skill not found
**Solution:** Verify openspec skill is installed and available

#### Issue: codex session lost between stages
**Solution:** Ensure sessionId is properly captured and passed between Stage 3 and 4

#### Issue: Frontend detection unclear
**Solution:** Manually review proposal and ask user if frontend is needed

#### Issue: Tests fail in Stage 4
**Solution:** Let codex iterate on fixes. If it fails after 3 attempts, escalate to user

#### Issue: Sandbox restrictions blocking operations
**Solution:** Verify sandbox level is appropriate:
- Use "workspace-write" for development and testing
- Use "read-only" for review
- Only use "danger-full-access" if explicitly approved by user

### Error Recovery

If any stage fails catastrophically:
1. Capture error details
2. Report to user with context
3. Offer options:
   - Retry the failed stage
   - Skip to next stage (if safe)
   - Restart from Stage 1
   - Manual intervention

### Session Management Best Practices

- **Stages 3 & 4:** Use same sessionId for context continuity
- **Stage 5:** Use fresh session for objective review
- **Error recovery:** Start new session after failures
- **Long workflows:** Consider session timeout limits

## Performance Optimization

### For Large Projects

- Break implementation into smaller phases
- Run tests incrementally during development
- Use targeted reviews for specific components
- Consider parallel execution where possible

### For Simple Projects

- Combine stages where appropriate
- Use minimal testing for trivial changes
- Skip review for low-risk code

## Quality Gates

Before proceeding to next stage, verify:

**Stage 1 → 2:**
- [ ] Requirements are complete and clear
- [ ] User has confirmed understanding

**Stage 2 → 3:**
- [ ] Proposal is technically sound
- [ ] Frontend requirement is explicitly marked
- [ ] All interfaces are documented

**Stage 3 → 4:**
- [ ] Code compiles/runs without errors
- [ ] Basic functionality works
- [ ] Dependencies are installed

**Stage 4 → 5:**
- [ ] All tests pass
- [ ] No critical bugs remain
- [ ] Code is ready for review

**Stage 5 → Complete:**
- [ ] No critical security issues
- [ ] Code quality is acceptable
- [ ] Documentation is adequate
