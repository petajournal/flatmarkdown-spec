# Examples of comrak Extensions

This file explains the comrak extensions listed under "Available Syntax" in SPEC.md.
For each extension, it shows how to write it and how Flat Markdown splits it into items.

In Flat Markdown, each top-level block becomes one item.
Inline syntax (used inside a sentence) does not affect item boundaries.
Block syntax (alert, subtext, multiline_block_quotes, display math) becomes one item by itself.

## footnote

Write `[^name]` in the text. Define the footnote body with `[^name]: text`.

```markdown
Flat Markdown uses block boundaries[^1].

[^1]: A blank line is not required to separate items.
```

- The footnote reference is inline.
- The footnote definition is treated as a line that produces no block. It is appended to the end of the previous item's body.
- The example above becomes one item. The footnote still resolves when the body is parsed alone.
- The reference and the definition may also be in different items. Both are kept when a body is parsed alone, so a reference in one item links to a definition in a later item, such as a list of definitions at the end of the page.

## alert

Put `[!TYPE]` on the first line of a block quote. The five types are NOTE, TIP, IMPORTANT, WARNING, and CAUTION.

```markdown
> [!NOTE]
> A metadata line must fit on one line.

> [!WARNING]
> A metadata line must not start with whitespace.
```

- An alert is one block. The example above becomes two items.
- Several paragraphs inside one alert still form one item.
- Combined with multiline block quotes (`>>>`, see below), you can omit the leading `>` on each line.

```markdown
>>> [!TIP]
A long alert is easier to read as a multiline block quote.

- Lists work here too.
>>>
```

## subtext

Start a line with `-#` followed by a space. The whole paragraph becomes small supporting text.

```markdown
Paragraph 1

-# Information as of October 2026.
```

- Subtext is block syntax. The example above becomes two items.
- It looks like a heading `#`, but it starts with `-`.

## highlight

Wrap the text in `==`.

```markdown
A tag starts with ==a # that follows a space==.
```

- Inline syntax. It does not affect item boundaries.

## superscript

Wrap the text in `^`.

```markdown
The area is x^2^.
```

- Inline syntax.
- It looks like a footnote `[^1]`, but only the text between `^` marks becomes superscript.

## subscript

Wrap the text in `~`.

```markdown
The formula for water is H~2~O.
```

- Inline syntax.
- It is distinct from GFM strikethrough `~~text~~`. One `~` is subscript. Two are strikethrough.

## spoilered_text

Wrap the text in `||`. The text stays hidden until clicked.

```markdown
The culprit was ||the butler||.
```

- Inline syntax.
- It is easy to confuse with the table cell separator `|`. Avoid it inside tables.

## shortcode

Write `:name:`. It is replaced with the emoji of the same name as on GitHub.

```markdown
Task completed :tada:
```

- Inline syntax.
- If the name does not exist, the text is left as is.

## math

Wrap inline math in `$`. Wrap display math in `$$`.

```markdown
The quadratic formula is $x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$.

$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$
```

- Inline math does not affect item boundaries.
- Display math wrapped in `$$` is one block. The example above becomes two items.
- `^` and `~` inside math are not treated as superscript or subscript. They are part of the math.

## hashtag

Write `#` followed by the tag name. Only a `#` at the start of a line, or right after a space, starts a tag.

```markdown
#diary Wrote the spec today. #flat-markdown #spec/v0.2
```

- Inline syntax.
- Allowed characters are Unicode letters and digits, plus `-`, `_`, and `/`. Punctuation or whitespace ends the tag.
- A `#` without a preceding space is not a tag. For example, `C#` is not a tag.
- Write `\#` to prevent it from being parsed as a tag.
- A tag has the same meaning as a wikilink to a page. See "Tags" in SPEC.md for details.

## underline

Wrap the text in `__`.

```markdown
The only __required__ element is `@`.
```

- Inline syntax.
- In standard Markdown, `__text__` is bold. With this extension it becomes underline. Use `**text**` for bold.

## wikilink

Write `[[full path of the page]]`. To add a label, put it after `|`.

```markdown
See [[spec/flatmarkdown]] for details.

With a label: [[spec/flatmarkdown|Specification]]

Link to a block: [[spec/flatmarkdown#2026-09-26-18-14-47-32W15WT|Metadata example]]
```

- Inline syntax.
- Write the page name as a full path.
- An item ID after `#` links to that item.
- When written to a Markdown file, it is converted to a Markdown link. See "Wikilink Syntax" in SPEC.md for details.

## multiline_block_quotes

Surround the text with lines that contain only `>>>`. The lines inside become a block quote without a leading `>`.

```markdown
>>>
First paragraph of the quote.

Second paragraph of the quote.

- A list inside the quote
>>>
```

- A multiline block quote is one block. The example above becomes one item.
- It stays one item no matter how many paragraphs or lists it contains.
- The result is the same as a normal block quote with `>` on every line. It is handy for long quotes.

## Summary

| Extension | Syntax | Kind | Item boundary |
| --- | --- | --- | --- |
| footnote | `[^1]` / `[^1]: text` | inline + definition line | definition joins the previous item |
| alert | `> [!NOTE]` | block | one item |
| subtext | `-# text` | block | one item |
| highlight | `==text==` | inline | no effect |
| superscript | `^text^` | inline | no effect |
| subscript | `~text~` | inline | no effect |
| spoilered_text | `\|\|text\|\|` | inline | no effect |
| shortcode | `:tada:` | inline | no effect |
| math | `$x$` / `$$x$$` | inline / block | `$$` is one item |
| hashtag | `#tag` | inline | no effect |
| underline | `__text__` | inline | no effect |
| wikilink | `[[path\|label]]` | inline | no effect |
| multiline_block_quotes | `>>>` ... `>>>` | block | one item |
