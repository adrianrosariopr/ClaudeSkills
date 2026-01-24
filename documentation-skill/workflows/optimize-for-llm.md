# Workflow: Optimize Document for LLM Readability

<required_reading>
**Read this reference file NOW:**
1. references/llm-optimized-structure.md
</required_reading>

<process>

<step name="assess-current-state">
## Step 1: Assess Current State

Ask user for the document or path to it.

Evaluate current LLM-readability:

**Structure Check**
- Are headers hierarchical (H1 → H2 → H3)?
- Are topics separated into distinct sections?
- Is there a logical reading order?

**Content Check**
- Are paragraphs short (3-5 sentences)?
- Are lists used for multiple items?
- Are tables used for structured data?
- Is the formatting consistent?

**Semantic Check**
- Do headers describe their content?
- Are sections self-contained?
- Can sections be read independently?
</step>

<step name="identify-issues">
## Step 2: Identify LLM-Unfriendly Patterns

Look for these common problems:

**Flat Structure**
```
❌ Bad: Long paragraphs mixing multiple topics
✅ Good: Separate sections with clear headers
```

**Merged Topics**
```
❌ Bad: "Permissions and rate limits work as follows..."
✅ Good: Separate "Permissions" and "Rate Limits" sections
```

**Inconsistent Formatting**
```
❌ Bad: Mixing inline code and fenced blocks randomly
✅ Good: Consistent formatting throughout
```

**Wall of Text**
```
❌ Bad: 10+ sentence paragraphs
✅ Good: 3-5 sentence paragraphs with visual breaks
```

**Buried Information**
```
❌ Bad: Important details in the middle of paragraphs
✅ Good: Key info in headers, lists, or callouts
```
</step>

<step name="restructure">
## Step 3: Restructure for Hierarchy

Apply hierarchical organization:

**Level 1 (H1)**: Document title, major sections
**Level 2 (H2)**: Subsections, categories
**Level 3 (H3)**: Specific topics, items

```markdown
# API Documentation          ← H1: Document title

## Authentication           ← H2: Major section
### OAuth 2.0 Flow          ← H3: Specific topic
### API Keys                ← H3: Specific topic

## Endpoints                ← H2: Major section
### GET /users              ← H3: Specific topic
### POST /users             ← H3: Specific topic
```

Each header should describe what follows. LLMs use headers to navigate and chunk content.
</step>

<step name="optimize-content">
## Step 4: Optimize Content Blocks

**Convert long paragraphs to lists:**
```
❌ Before:
"The system supports PDF, DOC, DOCX, and TXT files.
Maximum file size is 10MB. Files are processed
asynchronously and results are available within 5 minutes."

✅ After:
**Supported Formats:**
- PDF
- DOC, DOCX
- TXT

**Limits:**
- Maximum file size: 10MB
- Processing time: < 5 minutes (async)
```

**Use tables for comparisons:**
```
| Plan | Storage | API Calls | Price |
|------|---------|-----------|-------|
| Free | 1GB | 100/day | $0 |
| Pro | 10GB | 1000/day | $29/mo |
```

**Add explicit section summaries:**
```markdown
## Authentication

**Summary:** This section covers OAuth 2.0 and API key authentication methods.

[Detailed content follows...]
```
</step>

<step name="separate-topics">
## Step 5: Separate Mixed Topics

LLMs chunk content by section. Mixed topics cause retrieval problems.

**Before (mixed):**
```markdown
## Configuration
To configure the app, set these environment variables:
DATABASE_URL, REDIS_URL, API_KEY. Note that rate limits
are 100 requests per minute for free tier and 1000 for paid.
```

**After (separated):**
```markdown
## Environment Configuration
Set these environment variables:
- `DATABASE_URL`: Database connection string
- `REDIS_URL`: Redis connection string
- `API_KEY`: Your API key

## Rate Limits
| Tier | Requests/Minute |
|------|-----------------|
| Free | 100 |
| Paid | 1000 |
```
</step>

<step name="add-metadata">
## Step 6: Add Structural Metadata

Help LLMs understand document context:

**Add document header:**
```markdown
---
title: API Documentation
type: Technical Reference
version: 2.1.0
last_updated: 2025-01-23
---
```

**Add section anchors:**
```markdown
## Authentication {#auth}
## Rate Limits {#limits}
```

**Add cross-references:**
```markdown
For authentication details, see [Authentication](#auth).
```
</step>

<step name="verify-optimization">
## Step 7: Verify Optimization

Run through this checklist:

- [ ] Every section has a descriptive header
- [ ] Headers follow logical hierarchy (H1 → H2 → H3)
- [ ] No topic is split across multiple sections
- [ ] No section mixes unrelated topics
- [ ] Paragraphs are ≤5 sentences
- [ ] Lists used for 3+ related items
- [ ] Tables used for structured comparisons
- [ ] Formatting is consistent throughout
- [ ] Key information is visually prominent
- [ ] Document has metadata (title, version, date)

Ask user: "Would you like me to apply these optimizations to your document?"
</step>

</process>

<format_decision_tree>
## When to Use Which Format

**Use Markdown when:**
- General documentation (guides, FAQs, READMEs)
- Blog posts and articles
- Most technical documentation

**Use tables when:**
- Comparing options or features
- Structured data with multiple attributes
- Configuration options with values

**Use lists when:**
- Steps in a process
- Multiple items of the same type
- Features or requirements

**Use code blocks when:**
- Code examples
- Configuration snippets
- Command-line instructions

**Consider XML when:**
- Highly structured, nested content
- Strict sectioning requirements
- Complex referencing between sections
</format_decision_tree>

<success_criteria>
An LLM-optimized document:
- [ ] Has clear hierarchical structure
- [ ] Uses descriptive section headers
- [ ] Separates topics into distinct sections
- [ ] Has short paragraphs (3-5 sentences)
- [ ] Uses lists and tables appropriately
- [ ] Has consistent formatting
- [ ] Includes document metadata
- [ ] Can be chunked without losing context
</success_criteria>
