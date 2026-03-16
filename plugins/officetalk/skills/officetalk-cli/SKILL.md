---
name: officetalk-cli
description: >
  Apply deterministic operations to Microsoft Office documents using the OfficeTalk CLI tool.
  Use this skill when the user wants to modify Word (.docx), Excel (.xlsx), or PowerPoint (.pptx)
  documents programmatically — fixing typos, updating headings, reformatting tables, inserting
  sections, or applying bulk changes. Activate when the user mentions OfficeTalk, .otk files,
  document editing via CLI, or wants to script changes to Office documents. Also useful when an
  LLM needs to generate structured document modifications.
license: MIT
compatibility: Requires .NET 9.0 or later SDK. Works on Windows, macOS, and Linux.
metadata:
  author: spec-works
  version: "0.2"
  repository: https://github.com/spec-works/OfficeTalkEngine
---

# OfficeTalk CLI — Scripted Office Document Editing

Apply deterministic, repeatable operations to Microsoft Office documents (Word, Excel, PowerPoint) using a human-readable, LLM-friendly grammar.

OfficeTalk is a structured document format (`application/officetalk`, file extension `.otk`) that describes precise modifications to Office documents. The CLI tool parses `.otk` files, validates them, resolves addresses against target documents, and applies operations using the Office Open XML SDK.

## Prerequisites — Installing .NET

The OfficeTalk CLI requires .NET 9.0 or later. Install the SDK for your platform:

### Windows

```powershell
# Using winget (recommended)
winget install Microsoft.DotNet.SDK.9

# Or download from https://dotnet.microsoft.com/download/dotnet/9.0
```

### macOS

```bash
# Using Homebrew (recommended)
brew install dotnet-sdk

# Or download from https://dotnet.microsoft.com/download/dotnet/9.0
```

### Linux (Ubuntu/Debian)

```bash
wget https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install -y dotnet-sdk-9.0
```

### Linux (Fedora/RHEL)

```bash
sudo dnf install dotnet-sdk-9.0
```

### Verify Installation

```bash
dotnet --version
# Should show 9.x.x or later
```

## Installing the CLI Tool

```bash
dotnet tool install --global SpecWorks.OfficeTalk.CLI
```

Verify it works:

```bash
officetalk version
```

## OfficeTalk Document Format

An OfficeTalk document (`.otk` file) is a UTF-8 text file with a line-oriented grammar. Every document starts with a version header and document type, followed by operation blocks.

### Document Structure

```
OFFICETALK/1.0
DOCTYPE word|excel|powerpoint

# Comments start with #

AT <address>
<OPERATION> [arguments]
<OPERATION> [arguments]

AT <address>
<OPERATION> [arguments]
```

### Document Types

| DOCTYPE | Target | File Extension |
|---------|--------|----------------|
| `word` | Word documents | .docx |
| `excel` | Excel workbooks | .xlsx |
| `powerpoint` | PowerPoint presentations | .pptx |

## Addressing

Addresses are `/`-separated paths that locate content within an Office document. They use a simplified syntax — no XML namespaces, no Open XML internals.

### Path Segments

#### Word

| Segment | Description |
|---------|-------------|
| `body` | Document body |
| `paragraph` | A paragraph |
| `heading` | A paragraph with a heading style |
| `run` | A text run within a paragraph |
| `table` | A table |
| `row` | A table row |
| `cell` | A table cell |
| `list` | A list |
| `item` | A list item |
| `image` | An inline or anchored image |
| `section` | A document section |
| `header` | Page header |
| `footer` | Page footer |
| `bookmark` | A named bookmark |
| `content-control` | A structured document tag |

#### Excel

| Segment | Description |
|---------|-------------|
| `sheet` | A worksheet — `sheet["Revenue"]` or `sheet[1]` |
| `row` | A row within a sheet — `sheet[1]/row[3]` |
| *cell ref* | Direct cell reference — `sheet["Revenue"]/B7`, `sheet[1]/A1` |

Cell references (e.g., `A1`, `B7`, `D2`) are used directly as address segments. If the cell doesn't exist, it is created automatically.

#### PowerPoint

| Segment | Description |
|---------|-------------|
| `slide` | A slide — `slide[1]` or `slide[text*="Revenue"]` |
| `title` | Title placeholder shape |
| `subtitle` | Subtitle placeholder shape |
| `body` | Body/content placeholder shape |
| `notes` | Speaker notes |
| `shape` | A named shape — `shape[name="Logo"]` |
| `table` | A table on the slide |
| `image` | An image on the slide |

### Predicates

Predicates filter elements and go in square brackets after a segment.

#### Positional (1-based)

```
paragraph[3]              # third paragraph
table[1]/row[2]           # second row of first table
slide[5]                  # fifth slide
```

#### Key-Value

```
heading[level=2]                      # H2 heading
heading[level=2, text="Background"]   # specific H2 by text
table[caption="Results"]              # table by caption
sheet["Revenue"]                      # sheet by name (shorthand)
shape[name="Logo"]                    # shape by name
header[type=default]                  # default page header
content-control[tag="author-name"]    # content control by tag
```

#### Text Matching

| Operator | Meaning | Example |
|----------|---------|---------|
| `text="..."` | Exact match | `paragraph[text="Hello World"]` |
| `text~="..."` | Regex match (I-Regexp) | `paragraph[text~="^Chapter \d+"]` |
| `text^="..."` | Starts with | `paragraph[text^="In conclusion"]` |
| `text$="..."` | Ends with | `paragraph[text$="respectively."]` |
| `text*="..."` | Contains | `paragraph[text*="important"]` |

#### Disambiguating Multiple Matches

```
paragraph[text*="revenue"][2]    # second paragraph containing "revenue"
```

### Address Examples

```
# Word
body/paragraph[1]
body/heading[level=1]
body/heading[level=2, text="Methods"]
body/paragraph[text*="conclusion"]
body/table[1]/row[3]/cell[2]
body/table[caption="Results"]/row[1]
body/bookmark["references"]
header[type=default]

# Excel
sheet["Revenue"]/B7
sheet["Q1 Budget"]/D2
sheet[1]/row[5]
sheet["Headcount"]/A7

# PowerPoint
slide[1]/title
slide[1]/subtitle
slide[3]/body
slide[3]/shape[name="Chart1"]
slide[2]/notes
```

## Operations

### Content Operations

#### SET — Replace element content

```
AT body/paragraph[3]
SET "New paragraph text."

AT body/paragraph[3]
SET <<<
This paragraph now contains
multiple lines of text.
>>>
```

#### REPLACE — Find and replace text

```
AT body/paragraph[1]
REPLACE "FY2024" WITH "FY2025"

# Replace all occurrences
AT body
REPLACE ALL "colour" WITH "color"
```

#### INSERT BEFORE / INSERT AFTER — Add sibling content

```
AT body/heading[text="Conclusion"]
INSERT BEFORE <<<
Recommendations

Based on the findings presented in this report, we recommend:

1. Increase investment in renewable energy.
2. Expand the remote work program.
>>>

AT body/paragraph[1]
INSERT AFTER "A new paragraph after the first one."
```

#### DELETE — Remove an element

```
AT body/paragraph[text*="DRAFT"]
DELETE
```

#### APPEND / PREPEND — Add to existing content

```
AT body/paragraph[1]
APPEND " (see Appendix A)"

AT sheet["Notes"]/A1
PREPEND "UPDATED: "
```

### Formatting Operations

#### FORMAT — Apply formatting properties

```
AT body/paragraph[1]
FORMAT bold=true, font-size=14pt, color=#2B579A

AT body/heading[level=1]
FORMAT font-name="Aptos Display", font-size=28pt
FORMAT color=#1F3864, spacing-after=12pt
```

When FORMAT follows a content operation in the same block, it formats the produced content:

```
AT body/heading[text="Summary"]
INSERT AFTER "Key Findings"
FORMAT bold=true, font-size=16pt
```

#### STYLE — Apply a named style

```
AT body/paragraph[5]
STYLE "Heading 2"

AT body/table[1]
STYLE "Grid Table 4 - Accent 1"
```

### Structural Operations

#### Table Operations

```
AT body/table[1]/row[3]
INSERT ROW AFTER
SET CELLS "Product D", "450", "12%", "Active"

AT body/table[1]/row[5]
DELETE ROW

AT body/table[1]/row[2]/cell[1]
MERGE CELLS TO row[2]/cell[3]
```

#### Slide Operations (PowerPoint)

```
AT slide[3]
INSERT SLIDE AFTER
SET title "New Section"
SET subtitle "Overview of Changes"

AT slide[5]
DELETE SLIDE

AT slide[3]
DUPLICATE SLIDE
```

#### Sheet Operations (Excel)

```
ADD SHEET "Summary"

AT sheet["Old Name"]
RENAME SHEET "New Name"
```

### Annotation Operations

#### COMMENT — Add a review comment

```
AT body/paragraph[text*="revenue"]
COMMENT "Please verify this figure against the Q1 actuals."

AT sheet["Q1 Budget"]/D2
COMMENT "Engineering is $12K over budget. Verify contractor spend."

AT slide[3]
COMMENT "These risks need mitigation plans before the board meeting."

# Multi-line comment using content block
AT body/heading[text="Conclusion"]
COMMENT <<<
This section needs work:
- Add supporting data
- Reference the appendix
>>>
```

Comments are supported across all three document types:
- **Word**: Creates range-based comments visible in the Review pane
- **Excel**: Adds cell comments (visible on hover)
- **PowerPoint**: Adds slide-level comments

### Metadata Operations

```
PROPERTY title="Quarterly Report Q1 2026"
PROPERTY author="Finance Team"
```

## Bulk Operations with AT EACH

Use `AT EACH` to apply operations to all matching elements:

```
AT EACH body/paragraph[text*="TODO"]
FORMAT highlight=#FFFF00, bold=true

AT EACH body/heading[level=2]
FORMAT color=#2B579A
```

**Important:** `AT EACH` requires at least one match — zero matches is an error. Use it for bulk formatting, styling, and replacement. Exercise caution with destructive operations like DELETE.

## Data Types

| Type | Examples |
|------|---------|
| Strings | `"Hello"`, `"She said \"hi\""`, `"Line 1\nLine 2"` |
| Numbers | `42`, `3.14`, `-7` |
| Booleans | `true`, `false` |
| Colors | `#2B579A`, `#FF0000`, `red`, `cornflowerblue` |
| Lengths | `12pt`, `1.5in`, `2.54cm`, `50%`, `914400emu` |
| Content blocks | `<<<` ... `>>>` (multi-line text, each on its own line) |

## Formatting Properties Reference

### Text Properties

`font-name`, `font-size`, `bold`, `italic`, `underline` (single/double/dotted/dashed/wavy/none), `strikethrough`, `color`, `highlight`, `superscript`, `subscript`, `small-caps`, `all-caps`

### Paragraph Properties

`alignment` (left/center/right/justify), `spacing-before`, `spacing-after`, `line-spacing`, `indent-left`, `indent-right`, `indent-first-line`, `indent-hanging`, `keep-with-next`, `page-break-before`

### Table/Cell Properties

`width`, `alignment`, `border`, `border-color`, `border-width`, `cell-padding`, `fill-color`, `vertical-align`, `wrap-text`, `number-format`

### Image Properties

`width`, `height`, `alt`, `position` (inline/anchor)

## Processing Model — Snapshot Semantics

**Critical concept:** All addresses are resolved against the original document BEFORE any operations are applied. This means:

- `paragraph[5]` always refers to the fifth paragraph in the original document
- Operations that insert or delete elements do NOT shift indices for later blocks
- Every block operates on the document as it existed before any modifications

This makes OfficeTalk documents safe to produce — the author reasons about the current state of the document, not a partially-modified intermediate state.

## Live Editing with COM (Windows)

On Windows, if the target application (Word, Excel, or PowerPoint) is running with the target document open, the CLI **automatically uses COM automation** instead of OpenXML. This means:

- **Changes appear instantly** in the open document — no need to close and reopen
- **No file lock conflicts** — COM edits the live document through the application
- **Automatic detection** — no flags needed; the CLI checks if the app has the file open
- Falls back to OpenXML if the application isn't running or the file isn't open

Supported COM operations by application:

| Operation | Word | Excel | PowerPoint |
|-----------|------|-------|------------|
| SET | ✓ | ✓ | ✓ |
| DELETE | ✓ | ✓ | ✓ |
| COMMENT | ✓ | ✓ | ✓ |
| FORMAT | ✓ | ✓ (bold, borders, fill, alignment, number-format) | ✓ (background, font) |
| REPLACE | ✓ | — | — |
| INSERT | ✓ | — | — |
| STYLE | ✓ | — | — |

When COM is active, verbose mode (`-v`) will report:
```
Excel is open with target workbook — using COM executor.
```

## CLI Commands

### `officetalk apply` — Execute .otk against a document

```bash
# Apply changes in-place
officetalk apply -i changes.otk -t report.docx

# Write to a new file
officetalk apply -i changes.otk -t report.docx -o report-updated.docx

# Dry run — validate and resolve without modifying
officetalk apply -i changes.otk -t report.docx --dry-run

# Verbose output
officetalk apply -i changes.otk -t report.docx -v

# Pipe OfficeTalk from stdin
echo "OFFICETALK/1.0
DOCTYPE word
AT body/heading[1]
SET \"Updated Title\"" | officetalk apply -t report.docx
```

| Option | Alias | Description |
|--------|-------|-------------|
| `--input` | `-i` | Path to .otk file (or pipe via stdin) |
| `--target` | `-t` | Target Office document (required) |
| `--output` | `-o` | Output path (default: modify in-place) |
| `--force` | — | Overwrite output if exists |
| `--dry-run` | — | Validate without applying |
| `--verbose` | `-v` | Show detailed operation log |

### `officetalk validate` — Check .otk for errors

```bash
# Syntax-only validation (no target doc needed)
officetalk validate -i changes.otk --syntax-only

# Full validation with semantic checks
officetalk validate -i changes.otk -t report.docx

# Machine-readable JSON output
officetalk validate -i changes.otk -f json
```

| Option | Alias | Description |
|--------|-------|-------------|
| `--input` | `-i` | Path to .otk file (required) |
| `--target` | `-t` | Target document for semantic validation |
| `--syntax-only` | — | Skip semantic validation |
| `--format` | `-f` | Output format: `text` (default) or `json` |

### `officetalk parse` — Dump AST as JSON

```bash
officetalk parse -i changes.otk
officetalk parse -i changes.otk --pretty
```

### `officetalk inspect` — Explore addresses

```bash
# See what an address resolves to
officetalk inspect -t report.docx -a "body/heading[level=1]"

# With surrounding context
officetalk inspect -t report.docx -a "body/paragraph[text*='revenue']" -c 2
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Runtime/IO error |
| 2 | Parse or validation error |
| 3 | Address resolution failure |

## Writing OfficeTalk Documents — Best Practices

### 1. Always Start with Header

Every `.otk` file must begin with exactly:
```
OFFICETALK/1.0
DOCTYPE word
```
(Replace `word` with `excel` or `powerpoint` as needed.)

### 2. Use Comments to Document Intent

```
# Fix the typo reported in issue #42
AT body/paragraph[text*="teh company"]
REPLACE "teh" WITH "the"
```

### 3. Prefer Text Predicates Over Positional

Text-based addresses are more robust — they survive document edits:
```
# Good — survives if paragraphs are added/removed above
AT body/heading[text="Conclusion"]

# Fragile — breaks if document structure changes
AT body/heading[5]
```

### 4. Use Specific Addresses

Be as specific as possible to avoid ambiguity errors:
```
# Good — specific
AT body/heading[level=2, text="Methods"]

# Risky — may match multiple H2 headings
AT body/heading[level=2]
```

### 5. Group Related Changes

Put related formatting in the same block:
```
AT body/heading[level=1]
SET "Annual Report — FY2026"
FORMAT font-size=28pt, color=#1F3864
STYLE "Title"
```

### 6. Use Content Blocks for Multi-line Text

```
AT body/paragraph[text="placeholder"]
SET <<<
First paragraph of the new content.

Second paragraph, separated by a blank line.
This continues the second paragraph.
>>>
```

### 7. Use AT EACH for Bulk Operations

```
# Highlight all TODO markers
AT EACH body/paragraph[text*="TODO"]
FORMAT highlight=#FFFF00

# Restyle all H2 headings
AT EACH body/heading[level=2]
FORMAT color=#2B579A, font-size=18pt
```

### 8. Validate Before Applying

```bash
# Always validate first, especially for LLM-generated .otk files
officetalk validate -i changes.otk -t report.docx
# If exit code 0, safe to apply
officetalk apply -i changes.otk -t report.docx
```

## Complete Examples

### Fix Typos and Update Metadata

```
OFFICETALK/1.0
DOCTYPE word

# Fix typo in introduction
AT body/paragraph[text*="teh company"]
REPLACE "teh" WITH "the"

# Update the document title
AT body/heading[level=1]
SET "Annual Report — FY2026"
FORMAT font-size=28pt, color=#1F3864

# Remove draft notice
AT body/paragraph[text="DRAFT — DO NOT DISTRIBUTE"]
DELETE

# Update metadata
PROPERTY title="Annual Report — FY2026"
PROPERTY author="Finance Team"
```

### Insert a New Section

```
OFFICETALK/1.0
DOCTYPE word

AT body/heading[text="Conclusion"]
INSERT BEFORE <<<
Recommendations

Based on the findings presented in this report, the committee
recommends the following actions:

1. Increase investment in renewable energy infrastructure.
2. Expand the remote work pilot program.
3. Commission an independent audit of supply chain practices.
>>>

AT body/paragraph[text="Recommendations"]
STYLE "Heading 2"
```

### Update a Table

```
OFFICETALK/1.0
DOCTYPE word

# Add a new data row
AT body/table[caption="Quarterly Results"]/row[4]
INSERT ROW AFTER
SET CELLS "Q4", "$2.1M", "$1.8M", "16.7%"

# Highlight the header row
AT body/table[caption="Quarterly Results"]/row[1]
FORMAT bold=true, fill-color=#2B579A, color=#FFFFFF
```

### Update an Excel Spreadsheet

```
OFFICETALK/1.0
DOCTYPE excel

# Add totals row with formatting
AT sheet["Q1 Budget"]/A7
  SET "Total"
  FORMAT bold=true, border-bottom=medium

AT sheet["Q1 Budget"]/B7
  SET "490000"
  FORMAT bold=true, border-bottom=medium

AT sheet["Q1 Budget"]/D2
  COMMENT "Engineering is $12K over budget. Verify contractor spend."
```

### Update a PowerPoint Presentation

```
OFFICETALK/1.0
DOCTYPE powerpoint

AT slide[1]/title
SET "Q1 2026 Business Review"

AT slide[1]/subtitle
SET "Prepared by the Strategy Team — March 2026"

# Change slide background color
AT slide[3]
FORMAT background=green

# Add a reviewer comment
AT slide[3]
COMMENT "These risks need mitigation plans before the board meeting."

AT slide[4]/body
SET "1. Finalize Q2 budget allocations\n2. Launch customer retention program\n3. Hire 5 senior engineers"
```

## Limitations — What OfficeTalk Cannot Do

1. **No code execution** — OfficeTalk is purely declarative; no formulas, macros, or scripts
2. **No external references** — content blocks are literal text only; no URL fetching or file includes
3. **No image insertion** — cannot embed new images (can modify existing image properties)
4. **No chart creation** — charts must exist in the document already
5. **No template application** — cannot apply .dotx/.potx templates
6. **No shape creation** — cannot add textboxes or shapes to slides/documents (see [issue #2](https://github.com/spec-works/OfficeTalk/issues/2))
7. **FORMAT operation** — not all formatting properties are implemented yet (COM executors have broader support)
8. **Structural operations** — INSERT ROW/COLUMN, MERGE CELLS are not yet implemented for Excel/PowerPoint
9. **COM is Windows-only** — live editing requires Windows with the Office application installed

## Composing with Other Tools

```bash
# Convert markdown to Word, then apply OfficeTalk edits
markmyword convert -i doc.md -o doc.docx && officetalk apply -i polish.otk -t doc.docx

# Generate an .otk file with an LLM, validate, then apply
llm "Fix all typos" --format officetalk > fixes.otk
officetalk validate -i fixes.otk -t report.docx && officetalk apply -i fixes.otk -t report.docx

# Validate OfficeTalk scripts in CI
for f in scripts/*.otk; do officetalk validate -i "$f" --syntax-only; done
```

## Installing This Skill

### GitHub Copilot CLI (personal)

```bash
git clone https://github.com/spec-works/OfficeTalkEngine.git /tmp/OfficeTalkEngine
mkdir -p ~/.copilot/skills/officetalk-cli
cp -r /tmp/OfficeTalkEngine/skills/officetalk-cli/* ~/.copilot/skills/officetalk-cli/
```

### GitHub Copilot CLI (project)

```bash
mkdir -p .github/skills/officetalk-cli
cp -r /path/to/OfficeTalkEngine/skills/officetalk-cli/* .github/skills/officetalk-cli/
git add .github/skills/
git commit -m "Add OfficeTalk CLI agent skill"
```

### Claude Code (personal)

```bash
mkdir -p ~/.claude/skills/officetalk-cli
cp -r /path/to/OfficeTalkEngine/skills/officetalk-cli/* ~/.claude/skills/officetalk-cli/
```

### Claude Code (project)

```bash
mkdir -p .claude/skills/officetalk-cli
cp -r /path/to/OfficeTalkEngine/skills/officetalk-cli/* .claude/skills/officetalk-cli/
```

### VS Code / Cursor (project)

```bash
mkdir -p .github/skills/officetalk-cli
cp -r /path/to/OfficeTalkEngine/skills/officetalk-cli/* .github/skills/officetalk-cli/
```

After installing, restart your agent session to pick up the new skill.

## Further Reference

- [OfficeTalk Specification](https://github.com/spec-works/OfficeTalk) — Full specification with formal grammar
- [OfficeTalkEngine](https://github.com/spec-works/OfficeTalkEngine) — Execution engine source code
