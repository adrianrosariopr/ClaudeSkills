<overview>
All Notion block types and how to express them in Notion-flavored Markdown when using the MCP `notion-create-pages` and `notion-update-page` tools.
</overview>

<text_blocks>

**Paragraph** - Standard text
```markdown
Just write plain text. Supports **bold**, *italic*, `inline code`, ~~strikethrough~~, and [links](url).
```

**Heading 1** - Major section headers
```markdown
# Section Title
```

**Heading 2** - Subsection headers
```markdown
## Subsection Title
```

**Heading 3** - Detail headers
```markdown
### Detail Title
```

**Toggleable Headings** - Collapsible heading sections
Headings can be made toggleable in Notion (expandable/collapsible). When creating via MCP, the tool may support toggle syntax or this may need manual adjustment.

</text_blocks>

<list_blocks>

**Bulleted List**
```markdown
- Item one
- Item two
  - Nested item
```

**Numbered List**
```markdown
1. First item
2. Second item
   1. Nested item
```

**To-Do List**
```markdown
- [ ] Unchecked task
- [x] Completed task
```

</list_blocks>

<rich_blocks>

**Quote**
```markdown
> This is a quote block. Use for emphasis, testimonials, or highlighted text.
```

**Callout** - Highlighted blocks with icons. In Notion-flavored Markdown:
```markdown
> [!NOTE]
> Important information in a callout block

> [!TIP]
> Helpful tip in a callout

> [!WARNING]
> Warning message in a callout
```

**Divider** - Horizontal separator
```markdown
---
```

**Code Block** - Syntax-highlighted code
````markdown
```javascript
const greeting = "Hello, Notion!";
console.log(greeting);
```
````
Supported languages include: javascript, typescript, python, java, go, rust, ruby, php, html, css, sql, bash, json, yaml, markdown, and many more.

**Table**
```markdown
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Cell 1   | Cell 2   | Cell 3   |
| Cell 4   | Cell 5   | Cell 6   |
```

**Toggle** - Collapsible content blocks
Toggles in Notion contain content that can be expanded/collapsed. The exact Markdown syntax depends on the MCP tool's implementation. Often represented as:
```markdown
<details>
<summary>Toggle title</summary>
Hidden content goes here
</details>
```

</rich_blocks>

<media_blocks>

**Image** (external URL only - uploads not supported via MCP)
```markdown
![Alt text](https://example.com/image.png)
```

**Bookmark** - Rich URL preview
```markdown
[Bookmark title](https://example.com)
```

**Embed** - Embedded external content
```markdown
[Embed](https://youtube.com/watch?v=...)
```

</media_blocks>

<advanced_blocks>

**Equation** - LaTeX math expressions
```markdown
$$E = mc^2$$
```
Inline: `$x^2 + y^2 = z^2$`

**Table of Contents** - Auto-generated from headings
Not directly expressible in Markdown. Mention in content as `/table_of_contents` or request it separately.

**Column Layout** - Multi-column layouts
Not directly expressible in standard Markdown. Notion's column_list and column blocks need to be created via the API's block structure or manually arranged after page creation.

**Synced Block** - Content shared across pages
Cannot be created via Markdown. Mention to the user as a manual step.

</advanced_blocks>

<colors>
Notion supports these colors for text and backgrounds:

**Text colors:** `default`, `gray`, `brown`, `orange`, `yellow`, `green`, `blue`, `purple`, `pink`, `red`

**Background colors:** `gray_background`, `brown_background`, `orange_background`, `yellow_background`, `green_background`, `blue_background`, `purple_background`, `pink_background`, `red_background`

Color application through MCP Markdown may be limited. If specific colors are critical, note them for manual adjustment.
</colors>

<rich_text_formatting>
Within any block, rich text supports:
- **Bold**: `**text**`
- *Italic*: `*text*`
- ~~Strikethrough~~: `~~text~~`
- `Inline code`: `` `code` ``
- [Links](url): `[text](url)`
- Underline: Not standard Markdown; may need manual application
- Text color: Not expressible in standard Markdown
</rich_text_formatting>
