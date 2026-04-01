---
name: md2pdf
description: >
  Convert Markdown documents to professionally typeset PDF files with reportlab.
  Handles CJK/Latin mixed text, fenced code blocks, tables, blockquotes, cover pages,
  clickable TOC, PDF bookmarks, watermarks, and page numbers. Supports multiple
  color themes (Warm Academic, Nord, GitHub Light, Solarized, etc.) and is
  battle-tested for Chinese technical reports. Use this skill whenever the user
  wants to turn a .md file into a styled PDF, generate a report PDF from markdown,
  or create a print-ready document from markdown content — especially if CJK
  characters, code blocks, or tables are involved. Also trigger when the user
  mentions "markdown to PDF", "md转pdf", "报告生成", or asks for a "typeset" or
  "professionally formatted" PDF from markdown source.
license: MIT
compatibility: >
  Requires Python 3.8+ and reportlab (`pip install reportlab`).
  macOS: uses Palatino, Songti SC, Menlo (pre-installed).
  Linux: uses Carlito, Liberation Serif, Droid Sans Fallback, DejaVu Sans Mono.
metadata:
  author: lovstudio
  version: "1.0.0"
  tags: markdown pdf cjk reportlab typesetting
---

# md2pdf — Markdown to Professional PDF

This skill converts any Markdown file into a publication-quality PDF using Python's
reportlab library. It was developed through extensive iteration on real Chinese
technical reports and solves several hard problems that naive MD→PDF converters
get wrong.

## When to Use

- User wants to convert `.md` → `.pdf`
- User has a markdown report/document and wants professional typesetting
- Document contains CJK characters (Chinese/Japanese/Korean) mixed with Latin text
- Document has fenced code blocks, markdown tables, or nested lists
- User wants a cover page, table of contents, or watermark in their PDF

## Quick Start

```bash
python md2pdf/scripts/md2pdf.py \
  --input report.md \
  --output report.pdf \
  --title "My Report" \
  --author "Author Name" \
  --theme warm-academic
```

All parameters except `--input` are optional — sensible defaults are applied.

## Pre-Conversion Options (MANDATORY)

**IMPORTANT: You MUST use the `AskUserQuestion` tool to ask these questions BEFORE
running the conversion. Do NOT list options as plain text — use the tool so the user
gets a proper interactive prompt. Ask all three in a SINGLE `AskUserQuestion` call.**

Use `AskUserQuestion` with this exact format:

```
转 PDF 前需要确认几个选项：

1. 扉页图片（封面后的全页插图）
   a) 跳过
   b) 我提供本地图片路径
   c) AI 根据内容自动生成

2. 水印（每页淡色对角线文字）
   a) 不加水印
   b) 自定义水印文字（如 "DRAFT"、"内部资料"、"仅供学习"）

3. 封底宣传物料（名片/二维码/品牌信息）
   a) 跳过
   b) 我提供图片（名片/二维码/logo 等）
   c) 纯文字（网站/公众号/版权声明等）

请回复你的选择，如 "1a 2b:仅供学习参考 3b:/path/to/qr.png"
```

### Handling Responses

- **Frontispiece "AI generate"**: Read the document title + first paragraphs, use an
  image generation tool to create a themed illustration, show for approval, then
  pass via `--frontispiece /path/to/image.png`
- **Frontispiece "local"**: Use path directly via `--frontispiece <path>`
- **Watermark**: Pass via `--watermark "文字内容"`
- **Back cover image**: Pass via `--banner <path>` (recommend 1200px+ wide)
- **Back cover text**: Use `--disclaimer "声明文字"` and `--copyright "© 版权信息"`

## Architecture

```
Markdown → Preprocess (split merged headings) → Parse (code-fence-aware) → Story (reportlab flowables) → PDF build
```

Key components:
1. **Font system**: Palatino (Latin body), Songti SC (CJK body), Menlo (code) on macOS; auto-fallback on Linux
2. **CJK wrapper**: `_font_wrap()` wraps CJK character runs in `<font>` tags for automatic font switching
3. **Mixed text renderer**: `_draw_mixed()` handles CJK/Latin mixed text on canvas (cover, headers, footers)
4. **Code block handler**: `esc_code()` preserves indentation and line breaks in reportlab Paragraphs
5. **Smart table widths**: Proportional column widths based on content length, with 18mm minimum
6. **Bookmark system**: `ChapterMark` flowable creates PDF sidebar bookmarks + named anchors
7. **Heading preprocessor**: `_preprocess_md()` splits merged headings like `# Part## Chapter` into separate lines

## Hard-Won Lessons

### CJK Characters Rendering as □

reportlab's `Paragraph` only uses the font in ParagraphStyle. If `fontName="Mono"` but
text contains Chinese, they render as □. **Fix**: Always apply `_font_wrap()` to ALL text
that might contain CJK, including code blocks.

### Code Blocks Losing Line Breaks

reportlab treats `\n` as whitespace. **Fix**: `esc_code()` converts `\n` → `<br/>` and
leading spaces → `&nbsp;`, applied BEFORE `_font_wrap()`.

### CJK/Latin Word Wrapping

Default reportlab breaks lines only at spaces, causing ugly splits like "Claude\nCode".
**Fix**: Set `wordWrap='CJK'` on body/bullet styles to allow breaks at CJK character boundaries.

### Canvas Text with CJK (Cover/Footer)

`drawString()` / `drawCentredString()` with a Latin font can't render 年/月/日 etc.
**Fix**: Use `_draw_mixed()` for ALL user-content canvas text (dates, stats, disclaimers).

## Configuration Reference

| Argument | Default | Description |
|----------|---------|-------------|
| `--input` | (required) | Path to markdown file |
| `--output` | `output.pdf` | Output PDF path |
| `--title` | From first H1 | Document title for cover page |
| `--subtitle` | `""` | Subtitle text |
| `--author` | `""` | Author name |
| `--date` | Today | Date string |
| `--version` | `""` | Version string for cover |
| `--watermark` | `""` | Watermark text (empty = none) |
| `--theme` | `warm-academic` | Color theme name |
| `--theme-file` | `""` | Custom theme JSON file path |
| `--cover` | `true` | Generate cover page |
| `--toc` | `true` | Generate table of contents |
| `--page-size` | `A4` | Page size (A4 or Letter) |
| `--frontispiece` | `""` | Full-page image after cover |
| `--banner` | `""` | Back cover banner image |
| `--header-title` | `""` | Report title in page header |
| `--footer-left` | author | Brand/author in footer |
| `--stats-line` | `""` | Stats on cover |
| `--stats-line2` | `""` | Second stats line |
| `--edition-line` | `""` | Edition line at cover bottom |
| `--disclaimer` | `""` | Back cover disclaimer |
| `--copyright` | `""` | Back cover copyright |
| `--code-max-lines` | `30` | Max lines per code block |

## Themes

Available: `warm-academic`, `nord-frost`, `github-light`, `solarized-light`,
`paper-classic`, `ocean-breeze`.

Each theme defines: page background, ink color, accent color, faded text, border, code background, watermark tint.

## Dependencies

```bash
pip install reportlab --break-system-packages
```
