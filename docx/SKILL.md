---
name: docx
description: Comprehensive Word document creation, editing, and analysis with tracked changes, comments, OOXML manipulation, and professional redlining workflows for .docx files
origin: tfriedel/claude-office-skills/public/docx
tools: Read, Write, Edit, Bash
---

# DOCX creation, editing, and analysis

## Overview

A user may ask you to create, edit, or analyze the contents of a .docx file. A .docx file is essentially a ZIP archive containing XML files and other resources that you can read or edit. You have different tools and workflows available for different tasks.

## Workflow Decision Tree

### Reading/Analyzing Content
Use "Text extraction" or "Raw XML access" sections below

### Creating New Document
Use "Creating a new Word document" workflow

### Editing Existing Document
- **Your own document + simple changes**
  Use "Basic OOXML editing" workflow

- **Someone else's document**
  Use **"Redlining workflow"** (recommended default)

- **Legal, academic, business, or government docs**
  Use **"Redlining workflow"** (required)

## Text Extraction

```bash
# Extract text from docx
python -m markitdown document.docx
```

## Raw XML Access

```bash
# Unpack .docx to inspect XML structure
unzip document.docx -d document_unpacked/
# Main content is in word/document.xml
```

## Creating a New Word Document

Use python-docx for creating new documents:

```python
from docx import Document
from docx.shared import Inches, Pt, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH

doc = Document()

# Title
title = doc.add_heading('Document Title', 0)
title.alignment = WD_ALIGN_PARAGRAPH.CENTER

# Body paragraph
p = doc.add_paragraph('This is a body paragraph.')

# Add table
table = doc.add_table(rows=2, cols=3)
table.style = 'Table Grid'
table.cell(0, 0).text = 'Header 1'
table.cell(0, 1).text = 'Header 2'
table.cell(0, 2).text = 'Header 3'

doc.save('output/document.docx')
```

## Redlining Workflow (Tracked Changes)

Use the document.py script for professional redlining:

```bash
# The scripts/document.py handles tracked changes via OOXML
python scripts/document.py --input original.docx --output redlined.docx
```

Key tracked change XML elements:
- `<w:ins>` — inserted text (shown in green by default)
- `<w:del>` — deleted text (shown in red strikethrough)
- `<w:rPrChange>` — formatting change
- Author and date attributes required: `w:author="Name" w:date="2024-01-01T00:00:00Z"`

## Basic OOXML Editing

```python
import zipfile
import shutil
from lxml import etree

# Unpack
shutil.unpack_archive('document.docx', 'temp_docx', 'zip')

# Parse and edit XML
tree = etree.parse('temp_docx/word/document.xml')
root = tree.getroot()

# Make changes...

# Repack
tree.write('temp_docx/word/document.xml', xml_declaration=True, encoding='UTF-8', standalone=True)
shutil.make_archive('output_document', 'zip', 'temp_docx')
import os
os.rename('output_document.zip', 'output_document.docx')
```

## Adding Comments

```python
# Comments are stored in word/comments.xml
# Each comment needs:
# - Unique ID
# - Author name
# - Date
# - Comment text
# Reference in document.xml with <w:commentReference w:id="1"/>
```

## Dependencies

```
pip install python-docx lxml markitdown
```

## Output Structure

All generated documents go to `outputs/<document-name>/` directory.

## Example Prompts

- "Create a professional report with this content"
- "Add tracked changes to this document showing my edits"
- "Extract all text from this Word file"
- "Add comments to paragraphs 3 and 7"
- "Redline this contract with the following changes"
