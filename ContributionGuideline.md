# Contribution Guideline

### Overview

This page outlines the contribution process for the 'Standards and Guidelines' space. These guidelines help maintain consistency across all frontend projects. They supplement automated tools (ESLint, Prettier) with conventions that require human judgment.

Latest review date ΓÇô X

### What to contribute

1. Coding conventions that can't be automated.
2. Architecture patterns and best practices.
3. Lessons learned from production issues.
4. Team consensus on controversial topics.

### Contribution process

#### Phase 1: Before You Propose

1. Research existing guidelines.
2. Verify it's not automated in already.
3. Validate it's widely applicable.

Γ¥î **Don't propose:**

1. Personal style preferences.
2. Project-specific conventions.
3. Rules that contradict existing standards.
4. Rules already enforced by tooling.

#### Phase 2: Creating Your Proposal

1. **Choose the right page:** Add your rule to the appropriate guideline page:

- **JavaScript/TypeScript** - Language-level conventions.
- **ReactJS** ΓÇô Component patterns, hooks, JSX.
- **Assets** ΓÇô Images, SVGs, icons, media.
- **Testing** ΓÇô Test patterns, mocking.
- **Other** ΓÇô Create a new page if none fit.

  **2. Follow the standard format**

  Use this template for consistency:

```markdown
#### [Rule Title in Imperative Form]

Brief explanation of what the rule is and why it matters.

**Incorrect** [RED]
// Show the anti-pattern with comments explaining why it's wrong
const badExample = () => { ... };

**Correct** [GREEN]
// Show the recommended approach with comments if needed
const goodExample = () => { ... };

Rationale: [Explain WHY this rule exists - performance, maintainability, accessibility, etc.]

Exceptions: [List any valid exceptions to this rule]
```

**3. Mark it as a draft**  
Add `[DRAFT]` prefix or a draft label/badge to indicate it's pending review.

**4. Provide context in the edit summary**

1. What problem does this solve?
2. Link to Jira ticket or discussion if applicable.
3. Tag relevant team members.

#### Phase 3: Review & Approval

**1. Request review:** Tag the following in your page update or Slack message:

- **FE Lead Engineers**: @\[name\], @\[name\].
- **Optional**: Subject matter experts for specific topics.

**2. Gather feedback**

- Allow at least **5 business days** for review.
- Address comments and questions promptly.
- Be open to revising or withdrawing the proposal.
- Seek consensus, not just majority approval.

**3. Discussion channels**

- **Confluence comments** - For detailed technical discussion.
- **FE sync meetings** - For controversial or complex proposals.

**4. Approval criteria**  
A rule is approved when:

- Γ£à At least 2 FE Lead engineers approve.
- Γ£à No outstanding blocking concerns.
- Γ£à Practical examples are provided.
- Γ£à Rationale is clearly documented.

#### Phase 4: Publishing & Announcement

1. **Finalize the content**

- Remove `[DRAFT]` prefix/label.
- Add the approval date.
- Fix any formatting issues.
- Ensure all links work.

2. **Announce to the team**

Post in channel that New Frontend Guideline Approved!

```markdown
Topic: [Brief title]
Link: [Confluence link]
Summary: [1-2 sentence description]
Effective immediately / Effective [date]
```
