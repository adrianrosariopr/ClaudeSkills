<overview>
Documents optimized for LLM (Large Language Model) readability improve AI-assisted workflows: RAG retrieval, automated analysis, and AI-powered search. The same structural principles that help LLMs also improve human readability.
</overview>

<why_structure_matters>
## Why Structure Matters for LLMs

**How LLMs process documents:**
1. Documents are chunked into segments
2. Chunks are embedded as vectors
3. Retrieval finds relevant chunks by similarity
4. LLM generates responses using retrieved chunks

**The problem with poor structure:**
- Flat text makes chunking arbitrary
- Mixed topics in one section cause irrelevant retrieval
- Missing headers lose semantic boundaries
- Inconsistent formatting confuses parsing

"Hallucinations often start at ingestion—lose layout and you lose truth."
</why_structure_matters>

<hierarchy>
## Hierarchical Headers

Use strict header progression:

```markdown
# Document Title (H1)          ← Only one per document

## Major Section (H2)          ← Top-level categories

### Subsection (H3)            ← Specific topics

#### Detail (H4)               ← Rarely needed
```

**Rules:**
- Never skip levels (H1 → H3)
- Each header describes its content
- Headers create chunk boundaries
- Consistent hierarchy throughout

**Bad:**
```markdown
# API Guide
### Authentication      ← Skipped H2
## Endpoints
#### GET /users         ← Skipped H3
```

**Good:**
```markdown
# API Guide
## Authentication
### OAuth 2.0
### API Keys
## Endpoints
### GET /users
### POST /users
```
</hierarchy>

<paragraphs>
## Short Paragraphs

Limit paragraphs to 3-5 sentences. Long paragraphs:
- Dilute key information
- Make chunking less precise
- Reduce scanability

**Bad (wall of text):**
```
The authentication system supports multiple methods including OAuth 2.0 for
third-party integrations, API keys for server-to-server communication, and
session tokens for web applications. OAuth 2.0 requires registering your
application to obtain client credentials. API keys are generated in the
dashboard and should be kept secret. Session tokens are automatically
managed by our SDK. All methods require HTTPS. Rate limits apply to all
authentication methods equally.
```

**Good (chunked):**
```
The authentication system supports three methods:
- **OAuth 2.0**: For third-party integrations
- **API Keys**: For server-to-server communication
- **Session Tokens**: For web applications

All methods require HTTPS. Rate limits apply equally across methods.
```
</paragraphs>

<lists_and_tables>
## Lists and Tables

**Use lists when:**
- Enumerating 3+ related items
- Showing steps in a process
- Highlighting key points

**Use tables when:**
- Comparing options
- Showing structured data
- Mapping relationships

**Bad:**
```
The free plan includes 1GB storage and 100 API calls per day. The pro plan
includes 10GB storage and 1000 API calls per day for $29/month. The enterprise
plan includes unlimited storage and API calls with custom pricing.
```

**Good:**
```
| Plan | Storage | API Calls | Price |
|------|---------|-----------|-------|
| Free | 1GB | 100/day | $0 |
| Pro | 10GB | 1000/day | $29/mo |
| Enterprise | Unlimited | Unlimited | Custom |
```
</lists_and_tables>

<topic_separation>
## Topic Separation

**Critical rule: One topic per section.**

LLMs chunk by section. Mixed topics cause retrieval problems.

**Bad (mixed topics):**
```markdown
## Configuration

Set DATABASE_URL and REDIS_URL environment variables. Rate limits are
100 requests per minute for free tier. Make sure to set API_KEY as well.
Paid users get 1000 requests per minute.
```

If a user asks about rate limits, they might get database configuration too.

**Good (separated):**
```markdown
## Environment Variables

Set these variables before starting:
- `DATABASE_URL`: Database connection string
- `REDIS_URL`: Redis connection string
- `API_KEY`: Your API key

## Rate Limits

| Tier | Requests/Minute |
|------|-----------------|
| Free | 100 |
| Paid | 1000 |
```
</topic_separation>

<semantic_markers>
## Semantic Markers

Help LLMs understand content type:

**Callouts and emphasis:**
```markdown
> **Note:** This applies to v2.0+ only

> **Warning:** This operation cannot be undone

> **Important:** Read security guidelines first
```

**Code blocks with language:**
```python
# Always specify language for code
def example():
    return "LLM knows this is Python"
```

**Explicit section summaries:**
```markdown
## Authentication

**Summary:** This section covers three authentication methods: OAuth, API keys, and sessions.

[Detailed content follows...]
```
</semantic_markers>

<format_selection>
## When to Use Which Format

<decision_tree>
**Markdown** (default for most content)
- General documentation
- Guides and tutorials
- READMEs and wikis
- Technical documentation

**JSON/YAML** (structured data)
- Configuration specifications
- API response examples
- Data schemas
- Machine-readable configs

**XML** (complex nesting)
- Highly structured prompts
- Deep hierarchical content
- Strict sectioning requirements
- Content with cross-references

**Tables** (comparisons)
- Feature comparisons
- Configuration options
- Status matrices
- Structured reference data
</decision_tree>
</format_selection>

<metadata>
## Document Metadata

Add context at the top:

```markdown
---
title: API Documentation
type: Technical Reference
version: 2.1.0
last_updated: 2025-01-23
audience: Engineers
status: Approved
---
```

Metadata helps:
- LLMs understand document context
- Filtering during retrieval
- Version tracking
- Audience targeting
</metadata>

<checklist>
## LLM Optimization Checklist

**Structure**
- [ ] Single H1, logical H2/H3 hierarchy
- [ ] No skipped header levels
- [ ] One topic per section

**Content**
- [ ] Paragraphs ≤5 sentences
- [ ] Lists for 3+ items
- [ ] Tables for comparisons
- [ ] Code blocks with language tags

**Semantic**
- [ ] Descriptive headers
- [ ] Section summaries where helpful
- [ ] Callouts for important notes
- [ ] Document metadata

**Consistency**
- [ ] Consistent formatting throughout
- [ ] Consistent terminology
- [ ] Consistent header capitalization
</checklist>

<key_principles>
## Key Principles

1. **Hierarchy creates chunks**: Headers define semantic boundaries
2. **Short paragraphs aid precision**: Less dilution of key info
3. **Separate topics completely**: Mixed sections cause bad retrieval
4. **Structure aids both**: What helps LLMs helps humans too
5. **Metadata provides context**: Help LLMs understand document purpose
6. **Consistency matters**: Uniform formatting improves parsing
</key_principles>
