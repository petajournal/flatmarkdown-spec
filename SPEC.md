# Flat Markdown Specification v0.2

# Overview

- Flat Markdown is a note format for outliners. It uses Markdown delimited by blocks.
- The smallest unit in an outliner is a single list item.
- An outline is a nested document. A list item may contain another list.
- Markdown is designed for ordinary documents. It uses indentation to express an outline. Deep indentation makes items hard to read and write.
- Flat Markdown expresses an outline without indentation, so items are easy to read and write. Item boundaries match Markdown block boundaries, so items are easy to tell apart visually.

# Document (Page) Structure

```text
page
├── page-metadata?
└── item[]
    ├── item-metadata?
    │   ├── level
    │   ├── id?
    │   └── property[]
    └── body
```

`?` marks an optional element. `[]` marks a list of zero or more elements.

- **Page**
  - Consists of page metadata (page-metadata) and a list of items (item).
  - Page metadata
    - Optional.
    - The region enclosed by `---` at the top of the document is interpreted as YAML front matter and treated as page metadata.
    - Front matter is not an item.
  - Items
    - Outliner items. Written in Markdown.

- **Item**
  - Consists of item metadata (item-metadata) and a body.
  - Item metadata
    - Optional.
    - Written as a single-line HTML comment directly above the body.
    - Holds an indent level (level), an ID (id), and a list of properties (property).
    - The level is required. The ID is optional. The list of properties may be empty.
  - Body
    - Written in Markdown.

# Parsing Rules

## Available Syntax

- GitHub Flavored Markdown and the following comrak extensions are available.
  - footnote
  - alert
  - subtext
  - highlight
  - superscript
  - subscript
  - spoilered_text
  - shortcode
  - math
  - hashtag
  - underline
  - wikilink
  - multiline_block_quotes
- See [example.md](example.md) for examples of each extension.

## Item Delimiters via Blocks

- Each block element at the shallowest level is treated as an outline item.
  - Blocks nested inside a block are simply part of the body of the outer block.
  - For example, a list parsed as a single `<ul>` or `<ol>` is one item, including any nested lists.
  - Multiple paragraphs inside a block quote are also one item.
- Block boundaries are decided by the Markdown parser.
  - A blank line often becomes a block boundary. However, a blank line is not a condition for an item boundary.
  - For example, a paragraph written directly after a heading with no blank line is still a separate block. So it is a separate item.
  - For example, two consecutive list items form a single list block, with or without a blank line between them. So they are one item.
- All items in a document keep their order in the outline.

## Lines That Produce No Block

- Markdown has lines that the parser reads but that produce no block.
  - Link reference definitions (`[ref]: /url`). The parser only records them in its reference table and leaves no block.
  - Footnote definitions (`[^1]: text`). A referenced definition is a block for the Markdown parser. Flat Markdown still treats it as a line without a block, so that every footnote definition is handled the same way.
- These lines belong to no block. However, they must not be lost after parsing.
- These lines are appended to the body of the previous item.
  - Reference definitions and footnote definitions are normally written after the paragraph that uses them. Keeping them in the previous item lets the reference resolve even when the item body is parsed alone.
  - A footnote reference and its definition may also be in different items. A reference without a definition and a definition without a reference are both kept when the body is parsed alone, so they can be linked by label across items.
  - When the lines sit between a metadata line and its block, they are prepended to the body of that metadata's item. Metadata belongs to the block after it, not to the item before it.
  - When the lines are at the start of the document and there is no previous item, they are prepended to the body of the next block. When the next line is a metadata line, the lines form one item with omitted metadata.
- Example

```markdown
See [site][s].

[s]: https://example.com

Paragraph 2

[^1]: unreferenced
```

- The document above becomes two items.
  - The body of the first item is three lines: `See [site][s].`, a blank line, and `[s]: https://example.com`. `[site][s]` becomes a link even when the body is parsed alone.
  - The body of the second item is three lines: `Paragraph 2`, a blank line, and `[^1]: unreferenced`.

# Metadata

## Position

- Metadata is written as a single-line comment directly above a block at the shallowest level (an outline item).

```markdown
<!-- @ 2026-09-26-18-14-47-32W15WT -->

# Heading 1

Paragraph 1

<!-- @@ 2026-09-26-18-15-30-6VZSYB4 -->


<!-- @@ 2026-09-27-15-00-16-W3N8ASY plan="2026-09-28" -->
- [ ] Task1
```

- Blank lines between a metadata line and its body are ignored (the blank line above `Heading 1` in the example).
- A block without a metadata line is treated as an item whose metadata is omitted (`Paragraph 1` in the example).
- If only blank lines follow a metadata line until the next metadata line or the end of the document, that metadata line is treated as one item with an empty string as its body (the blank lines between `2026-09-26-18-15-30-6VZSYB4` and `2026-09-27-15-00-16-W3N8ASY` in the example).

## Syntax

```markdown
<!-- @@ ID key1="value1" key2="value2" -->
```

- A metadata line starts with `<!-- @` at the beginning of the line and ends with `-->`. No leading whitespace is allowed.
- A metadata line is complete on a single line. A comment spanning multiple lines is not metadata.
- Elements are separated by ASCII spaces.
- The only required element is one or more consecutive `@` characters.
- The number of `@` characters indicates the indent level. The shallowest level is `@`. An item indented one step to the right is `@@`.
- An item whose metadata is omitted is treated the same as `@`. In other words, when ordinary Markdown is interpreted under the Flat Markdown rules, every item is at the shallowest level.
- ID
  - Optional element. Used for cross-references and synchronization.
  - Placed directly after the `@` characters.
  - The format is HMTID. https://github.com/sosuisen/hmtid
    - A 27-character ID separated by hyphens, including the date and time.
- Properties
  - Optional elements. Zero or more may be listed.
  - Written in `key="value"` form. Placed after the ID, or after the `@` characters when there is no ID.
  - A key must not contain whitespace, `=`, or `"`.
  - A value is enclosed in double quotes. This allows a value to contain spaces. Metadata is not meant to be written by hand, so a few extra symbols are acceptable.
  - A `"` inside a value is escaped as `\"`. A `\` is escaped as `\\`.
  - A value must not contain `-->`.
  - The meaning of each key is defined by the application. Examples include the collapsed state of an item, task details, or any other key-value pair.
- Metadata can be written by hand, but it is designed for automatic input or input through a GUI.

## Invalid Metadata

- The following comments are not metadata. They are treated as body text.
  - A comment that does not start with `<!-- @`.
  - A comment that starts with `<!-- @` but does not follow the syntax above.
  - A comment spanning multiple lines.

# Characteristics of Flat Markdown

- Visual clarity
  - Item boundaries are the same as Markdown block boundaries. Anyone who can read Markdown can tell items apart naturally.
  - When a metadata line sits at the top of an item, the start of the item is even clearer.
- Compatibility
  - Ordinary Markdown documents, such as blog posts, can be handled by an outliner.
  - Because it is standard Markdown, it can be edited and displayed by any Markdown tool. Metadata is an HTML comment, so it does not get in the way of display.
- Ease of data conversion
  - Compatible with the many tools that publish Markdown data to web pages and other formats.
  - Splitting items can be left to the Markdown parser. There is no need to write a custom parser.
  - If you prefer to express outline items as a Markdown list tree, a tool that converts Flat Markdown into a Markdown list tree is easy to build.
- No-indentation philosophy
  - The first idea that comes to mind for expressing an outliner tree in Markdown is a Markdown list tree. It was not adopted.
  - When editing items by hand, it is easy to miscount the leading spaces in a list tree, and every line needs the same number of spaces. Moreover, deep indentation pushes items off the screen on narrow displays such as smartphones. This is inconvenient.
  - Markdown is a general-purpose markup language. It is not necessarily suited to expressing outliner items. However, Markdown has value as an intermediate language in the modern era.
  - Removing indentation solves all of these problems.
  - Removing indentation also brought new advantages.
    - First, the document diff is small when the indentation changes. Only the number of `@` characters in the metadata line changes.
    - When adding metadata to items, a Markdown list tree looks quite confusing. With block delimiters, item boundaries remain clearly distinguishable even after metadata lines are added.
  - The background assumes outliners with namespaces and page units, such as Roam Research and Logseq.
    - Because pages have hierarchy, indentation does not become as deep as in outliners with a single large tree, such as Workflowy.
    - With shallow indentation, the problems of having no visual indentation in Markdown are mitigated.

# Flat Markdown in PetaJournal

## Metadata

- journal-server writes a metadata line for every item. An ID is always assigned.
- The order of properties is not defined by the specification, but journal-server serializes them in a fixed order.

## Task List Interpretation

- For items starting with a GFM task list `- [ ]`, PetaJournal treats the entire item as a task and replaces the bullet with a checkbox.

## Embedding

Embedding format for media, including images:

```markdown
![alt](img.png#w=300&h=240)
![alt](video.mp4#w=640&autoplay)
![alt](https://youtube.com/watch?v=xxx#w=640&start=30)
![alt](doc.pdf#page=3&w=800)
```

The server-side parser adds an embed node to the AST and returns it. Rendering is the responsibility of the frontend.

## Tags

- Can be placed anywhere in the item body. `#tagname`
- Page tags are set as properties in the Front Matter.
- Only the following characters are valid:
  - Unicode alphanumeric characters
    - Based on Rust's char::is_alphanumeric()
    - Kanji: タグ, 日記 — Letter (Lo) → included
    - Hiragana: たぐ — Letter (Lo) → included
    - Katakana: タグ — Letter (Lo) → included
    - Full-width digits: １２３ — Digit (Nd) → included
    - Half-width alphanumerics: abc, 123 — included
    - Punctuation is not included.
  - Three symbols: `-` `_` `/`
  - `/` is not allowed at the beginning or end of a tag name.
- Tag start conditions
  - `#` at the beginning of a line or following a half-width space.
    - ==Note: Logseq does not require a half-width space, so this is not compatible.==
  - Can be escaped with a backslash. `\#`
- Tag end conditions
  - The rule is simply: the tag ends at any character that is not an allowed character.
    - In `#mytag。next`, the tag is mytag.
    - Japanese punctuation, (), [], etc. naturally terminate the tag.
    - Whitespace characters and the end of the string also terminate the tag.
- Tag semantics
  - Behavior is the same as Logseq. That is, a tag name is equivalent to a Wikilink, and the link target is a page.

## Handling in Markdown Files

Conversion rule: `Tag <---> Markdown link`

- When link resolution is ON
  - `#tagname <---> [#tagname](relative path to the tag page)`
  - The link target is the page whose full path is the tag name. The relative path is built the same way as in "Handling in Markdown Files" under "Wikilink Syntax".
  - Example: `#test` written in `journals/2026-10-12.md` becomes `[#test](../docs/test.md)`.
  - When reading, a Markdown link whose label is `#` followed by the full path of the page is turned back into a tag.
- When link resolution is OFF
  - No conversion

# Wikilink Syntax

- Link to a page
  - Wikilink syntax `[[full path of page name]]`
  - With a label `[[full path of page name|label]]`
- Link to a block
  - `[[full path of page name#id]]`
  - With a label `[[full path of page name#id|label]]`

## Handling in Markdown Files

- Conversion rule: `Wikilink <---> Markdown link`

- When link resolution is ON
  - When exporting to Markdown, convert Wikilinks to Markdown links before saving.
  - This is because general Markdown tools cannot handle Wikilinks.
  - In other words, write Wikilinks in the app, but serialize data as Markdown links.
  - The target of the Markdown link is the path of the target file relative to the file being written.
    - This lets the exported file work on its own in a general Markdown tool.
    - Files live in several folders such as `docs/` and `journals/`, so a fixed form like `../full path.md` cannot be resolved.
  - `[[full path of page name]] <---> [full path of page name](relative path)`
  - `[[full path of page name|label]] <---> [label](relative path)`
  - `[[full path of page name#id|label]] <---> [label](relative path#id)`
  - When the relative path contains a character that is not allowed in a Markdown link target, such as a space, wrap it in `<` and `>`. Example: `[My Page](<../My Page.md>)`
  - Example: with files placed as `docs/<full path>.md` and `journals/<date>.md`, `[[Projects/oecha/prd]]` written in `docs/Projects/oecha/index.md` becomes `[Projects/oecha/prd](prd.md)`. `[[2026-10-12]]` in the same file becomes `[2026-10-12](../../../journals/2026-10-12.md)`.
  - When reading, the relative path is resolved from the location of the file and turned back into the full path of the page.
    - When the label equals the full path, the result is a Wikilink without a label.
    - A link that does not resolve to a page file (an external URL, a file that is not a page, and so on) is kept as a Markdown link.
  - Text inside code blocks and inline code is never converted in either direction.
- When link resolution is OFF
  - No conversion

## Path Handling

- Obsidian allows omitting the folder path when there is no other file with the same name.
- Using full paths like Logseq seems simpler and better.
  - Therefore, when a folder changes, all links are rewritten.
  - This also makes conversion between Wikilinks and Markdown links easier.
