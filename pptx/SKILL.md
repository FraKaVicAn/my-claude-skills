---
name: pptx
description: Comprehensive PowerPoint presentation creation, editing, and analysis with HTML-to-PPTX conversion, template-based workflows, slide inventory, animations, and speaker notes for .pptx files
origin: tfriedel/claude-office-skills/public/pptx
tools: Read, Write, Edit, Bash
---

# PPTX Creation, Editing, and Analysis

## Overview

Comprehensive guidance for working with PowerPoint presentations (.pptx files). Three main workflows: reading/analyzing content, creating new presentations, and editing existing ones.

## Key Capabilities

- **Text Extraction**: `python -m markitdown presentation.pptx`
- **Raw XML Access**: Unpack to inspect animations, speaker notes, comments
- **HTML-to-PPTX**: Create from HTML templates (primary creation method)
- **Template-based**: Reuse existing presentation layouts
- **Visual Validation**: Generate thumbnail grids to check layout

## Creating New Presentations (html2pptx workflow)

```bash
# 1. Create HTML slides file
# 2. Convert to PPTX
node scripts/html2pptx.js slides.html output.pptx
# 3. Validate visually
python scripts/thumbnail.py output.pptx
```

### Slide HTML Structure
```html
<!-- Slide dimensions: 720pt × 405pt for 16:9 -->
<section class="slide" style="width:720pt; height:405pt;">
  <h1 style="font-family: Arial; font-size: 36pt; color: #1a1a2e;">Title</h1>
  <p style="font-size: 18pt;">Content goes here</p>
</section>
```

### Design Requirements
- Web-safe fonts only: Arial, Helvetica, Times New Roman, Georgia, Courier New, Verdana
- Strong text contrast (minimum 4.5:1 ratio)
- Consistent visual hierarchy across slides
- Maximum 6 bullets per content slide
- No gradients or 3D effects for data charts

### Color Palette Examples
```
Classic Blue:    #1a1a2e / #16213e / #0f3460 / #e94560
Corporate Gray:  #2d3436 / #636e72 / #b2bec3 / #dfe6e9
Forest Green:    #1e3932 / #00704a / #d4e9e2 / #cba258
Warm Amber:      #8b1a1a / #d4380d / #fa8c16 / #ffd591
```

## Template-Based Workflow

```bash
# 1. Generate thumbnail grid to see available layouts
python scripts/thumbnail.py template.pptx

# 2. Analyze slide inventory
python scripts/inventory.py template.pptx > inventory.json

# 3. Rearrange/duplicate slides
python scripts/rearrange.py template.pptx --plan "1,1,2,3,3,4" output.pptx

# 4. Extract text for replacement
python scripts/inventory.py output.pptx --text > text_inventory.json

# 5. Apply replacements
python scripts/replace.py output.pptx replacements.json final.pptx
```

### Replacement JSON Format
```json
{
  "slide_1": {
    "title": "Q4 2024 Results",
    "subtitle": "Record-breaking performance"
  },
  "slide_2": {
    "heading": "Revenue Growth",
    "body": "• $2.4M total revenue\n• 34% YoY growth\n• 89% retention rate"
  }
}
```

## Editing Existing Presentations (OOXML)

```bash
# Unpack
python -c "import shutil; shutil.unpack_archive('pres.pptx', 'pres_xml', 'zip')"

# Main slide files are at ppt/slides/slide1.xml, slide2.xml, etc.
# Edit XML directly

# Repack
python -c "import shutil; shutil.make_archive('output', 'zip', 'pres_xml'); import os; os.rename('output.zip', 'output.pptx')"
```

### Key XML Namespaces
```xml
xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main"
xmlns:p="http://schemas.openxmlformats.org/presentationml/2006/main"
xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships"
```

### Text Replacement in XML
```python
from lxml import etree

tree = etree.parse('pres_xml/ppt/slides/slide1.xml')
ns = {'a': 'http://schemas.openxmlformats.org/drawingml/2006/main'}

for t in tree.findall('.//a:t', ns):
    if 'OLD_TEXT' in t.text:
        t.text = t.text.replace('OLD_TEXT', 'NEW_TEXT')

tree.write('pres_xml/ppt/slides/slide1.xml', xml_declaration=True, encoding='UTF-8')
```

## Visual Validation

```bash
# Generate thumbnail grid (requires LibreOffice + Poppler)
python scripts/thumbnail.py presentation.pptx
# Creates: presentation_thumbnails.png

# Convert single slide to image
libreoffice --headless --convert-to png --outdir ./slides presentation.pptx
```

## python-pptx Alternative

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor

prs = Presentation()
slide_layout = prs.slide_layouts[1]  # Title and Content
slide = prs.slides.add_slide(slide_layout)

title = slide.shapes.title
title.text = "My Presentation"
title.text_frame.paragraphs[0].font.color.rgb = RGBColor(0x1a, 0x1a, 0x2e)

content = slide.placeholders[1]
tf = content.text_frame
tf.text = "First bullet point"
tf.add_paragraph().text = "Second bullet point"

prs.save('output.pptx')
```

## Output Structure

All generated presentations go to `outputs/<presentation-name>/` directory.

## Dependencies

```bash
pip install python-pptx markitdown lxml
npm install html2pptx  # for HTML-to-PPTX conversion
# System: LibreOffice (for PDF/image conversion), poppler-utils (for thumbnails)
```

## Example Prompts

- "Create a 10-slide pitch deck for this startup"
- "Convert this HTML report into a PowerPoint presentation"
- "Replace all company names in this template presentation"
- "Generate thumbnail previews of each slide"
- "Add speaker notes to each slide based on the content"
