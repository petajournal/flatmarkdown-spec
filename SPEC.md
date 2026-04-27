Flat Markdown defines clear rules for treating ordinary Markdown documents as items in an outliner.

# Flat Markdown Policy

- Item boundaries should be visually easy to distinguish and easy to edit in a plain text editor.
- All documents written in standard GitHub Flavored Markdown are accepted.
- Extended Markdown syntax is supported.
- Line breaks are interpreted in a custom way.

# Document (Page) Structure

```
page
├── page-metadata[]
└── item[]
    ├── body
    └── item-metadata[]
```

- **Page**
  - Consists of a list of page metadata (page-metadata) and a list of items (item).
  - Page metadata
    - The region enclosed by `---` at the top of the document is interpreted as YAML front matter and treated as page metadata.
  - Items
    - Outliner items. Written in Markdown.

- **Item**
  - Consists of a body and a list of item metadata (item-metadata).
  - Body
    - Written in Markdown.
  - Item metadata
    - Written in `<!-- HTML comment -->` format.


## Item Delimiters via Blank Lines

- One or more blank lines in a Markdown document are interpreted as item delimiters.
  - Essentially, a paragraph in Markdown corresponds to an item.
- Multiple consecutive blank lines are treated as a single delimiter.
- Blank lines within an item body are not permitted, with the sole exception of blank lines inside code fences.
- One or more trailing blank lines at the end of a document are ignored.
- The first blank line after a front matter block marks the end of the front matter and is not treated as an item delimiter.

## Items

Each item may contain any GFM content as its body.

### Differences from GFM

#### Blank Line Handling

- A blank-line paragraph is represented by a line containing only `<br />`.
- Intentional blank lines within an item:
  - Blank lines inside a code fence are treated as blank lines. They are not treated as item delimiters.
  - All other intentional blank lines must be represented by a line containing only `<br />`, to remain visually distinguishable.
- The parser automatically interprets any line outside a code fence that consists entirely of characters with the Unicode White_Space property (empty lines, space-only lines, full-width-space-only lines, etc.) as a `<br />` line.
  - Whitespace means any character with the Unicode White_Space property, including the ideographic space (U+3000).

#### Line Break Handling

- All newline characters are treated as hard line breaks (`<br />`).
- GFM Hard line breaks syntax (two or more trailing spaces, or a trailing backslash) is ignored.

#### Extended Syntax
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

### Item Metadata

The lines from after the final newline of the body up to the next blank line or the end of the page constitute the metadata section.

#### Metadata Types

Metadata is broadly divided into two types: ID and properties.

- ID
  - Mandatory metadata. Used for cross-referencing and synchronisation of data.
- Properties
  - Key=value properties. Includes the item's collapsed state (collapsed), task details, and any other arbitrary key-value pairs.

The indentation level of an item in the outline is stored as an item property.
- The default depth is level 1. One indent to the right is written as `<!-- level=2 -->`.
- Level 1 is the default, so the level property may be omitted.
- In other words, when a standard Markdown document is interpreted under Flat Markdown rules, all items are at level 1 (no indentation) because no level property is present.

#### Metadata Syntax

- Each type of metadata is written on its own line.
- Metadata is written inside HTML comments `<!-- -->`. These will not be displayed when converted to HTML.
- A valid metadata HTML comment must be a single-line comment starting at the beginning of the line, and must match either the `<!-- key=value -->` property pattern or the `<!-- HMTID -->` pattern. HTML comments that do not match are treated as body content.
- HMTID
  - https://github.com/sosuisen/hmtid
  - A 27-character ID in hyphen-delimited format that also delimits date and time components.
- Metadata can be written by hand, but GUI-based input is the intended workflow.

### Notes
- To minimise exceptions related to blank lines, extended block constructs such as multiline blockquotes (`>>>`) are not supported.

# Characteristics of Flat Markdown

- Visual clarity
  - Items separated by blank lines can be distinguished at a glance.
- Compatibility
  - Markdown documents written as ordinary pages (e.g. blog posts) can be handled by the outliner.
  - Because it is standard Markdown, it can be edited and rendered by any Markdown tool.
- Ease of data transformation
  - Compatible with many tools that publish Markdown data to web pages and other formats.
  - Code to split items is straightforward to write.
  - If you prefer to represent outline items as a Markdown list tree, a tool that reads Flat Markdown and converts it to a Markdown list tree is trivially easy to build. An official tool is planned, but with today's AI coding assistance you could write it yourself in under three minutes.
- No-indentation philosophy
  - The obvious way to represent an outliner tree in Markdown is as a list tree. However, that approach was not adopted.
  - When editing items by hand, list trees make it easy to get the leading whitespace count wrong, and every line must carry the same number of leading spaces. Heavy indentation also pushes items off-screen on narrow displays such as smartphones.
  - Markdown is a general-purpose markup language and is not necessarily well-suited to expressing outliner items. Nevertheless, Markdown has value as an interchange format in the modern era.
  - Removing indentation solves all of these problems.
  - Removing indentation also introduced new advantages:
    - Diffs are small when indentation changes — only the level property line is affected.
    - Adding a metadata section to items in a Markdown list tree produces a visually cluttered result. With blank-line delimiters, item boundaries remain clearly visible even after metadata is added.
  - The design background is outliners with namespaces and page units, such as Roam Research and Logseq.
    - Because pages carry their own hierarchy, indentation does not become as deep as it would in a single-large-tree outliner like Workflowy.
    - Shallower indentation reduces the problems caused by the visual absence of indentation in plain Markdown.

# Flat Markdown in PetaJournal

## Metadata

The order of metadata is not defined by the specification, but journal-server serialises it in the following order:
  - ID
  - Indent level
  - Other properties

## Task List Interpretation

- For items beginning with the GFM task list syntax `- [ ] `, PetaJournal treats the entire item as a task and replaces the bullet with a checkbox.

## Embed

Media embed format for images and other media:
```
![alt](img.png#w=300&h=240)
![alt](video.mp4#w=640&autoplay)
![alt](https://youtube.com/watch?v=xxx#w=640&start=30)
![alt](doc.pdf#page=3&w=800)
```
The server-side parser adds an embed node to the AST and returns it. Rendering is the responsibility of the frontend.

## Tags

- Can be placed anywhere in the item body. `#tagname`
- Page tags are set as Front Matter properties.
- Valid characters:
  - Unicode alphanumerics
    - Based on Rust's `char::is_alphanumeric()`
    - Kanji: タグ, 日記 — Letter (Lo) → included
    - Hiragana: たぐ — Letter (Lo) → included
    - Katakana: タグ — Letter (Lo) → included
    - Full-width digits: １２３ — Digit (Nd) → included
    - ASCII alphanumerics: abc, 123 — included
    - Punctuation is not included.
  - Three symbols: `-` `_` `/`
  - `/` is not permitted as the first or last character of a tag name.
- Tag start conditions
  - `#` at the beginning of a string or immediately after a half-width space (U+0020)
    - ==Note: Logseq does not require a preceding space, so there is an intentional incompatibility here.==
  - Can be escaped with a backslash: `\#`
- Tag end conditions
  - The tag ends at the first character that is not a valid tag character.
    - For `#mytag。next`, the tag is `mytag`.
    - Japanese punctuation, `()`, `[]`, etc. naturally terminate the tag.
    - Whitespace and end of string also terminate the tag.
- Tag semantics
  - Behaviour is the same as Logseq.
  - Equivalent to a Wikilink; the link target is a page.

## Handling in Markdown Files

Conversion rule `tag <---> Markdown link`

- Link resolution ON:
  - `#tagname <---> [#tagname](../tagname.md)`
- Link resolution OFF:
  - No conversion.


# Wikilink Syntax

- Link to a page
  - Wikilink syntax: `[[full/path/to/page]]`
  - With a label: `[[full/path/to/page|label]]`
- Link to a block
  - `[[full/path/to/page#id]]`
  - With a label: `[[full/path/to/page#id|label]]`

## Handling in Markdown Files

Conversion rule `wikilink <---> Markdown link`

- Link resolution ON:
  - When writing to Markdown files, wikilinks are converted to Markdown links rather than being stored as wikilinks.
  - General Markdown tools cannot handle wikilink syntax.
  - In other words, write in wikilink syntax within the app; the serialised data uses Markdown links.
  - `[[full/path/to/page]] <---> [full/path/to/page](../full/path/to/page.md)`
  - `[[full/path/to/page|label]] <---> [label](../full/path/to/page.md)`
- Link resolution OFF:
  - No conversion.


## Path Handling
- Obsidian allows the folder path to be omitted when no other file has the same name.
- Full paths (as in Logseq) are simpler and preferred.
  - Therefore, if a folder changes, all links must be rewritten.
  - This also makes conversion between wikilinks and Markdown links straightforward.
