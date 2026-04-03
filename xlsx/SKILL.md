---
name: xlsx
description: Comprehensive spreadsheet creation, editing, and analysis with support for formulas, data validation, and conditional formatting for .xlsx, .xlsm, .csv, .tsv files
origin: tfriedel/claude-office-skills/public/xlsx
tools: Read, Write, Edit, Bash
---

# XLSX Requirements and Workflow Documentation

## Core Requirements

**Zero Formula Errors**: All deliverables must contain zero Excel errors (#REF!, #DIV/0!, #VALUE!, #N/A, #NAME?).

**Template Preservation**: When modifying existing files, match established formatting and conventions rather than imposing standardized styles.

## Financial Model Standards

### Color Coding
- Blue text: Hardcoded inputs and user-changeable values
- Black text: All formulas and calculations
- Green text: Internal worksheet links
- Red text: External file references
- Yellow background: Key assumptions requiring attention

### Number Formatting
- Years display as text ("2024" not "2,024")
- Currency: `$#,##0` format with unit headers
- Zeros appear as "-"
- Percentages: `0.0%` format
- Multiples: `0.0x`
- Negative numbers: parentheses rather than minus signs

### Formula Construction
Place all assumptions in separate cells and reference them rather than hardcoding values. Verify cell references for accuracy, check for off-by-one errors, ensure consistent formulas across periods, and test edge cases.

## Implementation Tools

### pandas — Data Analysis
```python
import pandas as pd

# Read Excel
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')

# Basic analysis
print(df.describe())
print(df.head())

# Write back
df.to_excel('output.xlsx', index=False)
```

### openpyxl — Formulas and Formatting
```python
from openpyxl import load_workbook, Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.styles import numbers

wb = load_workbook('template.xlsx')
ws = wb.active

# Write formula (not calculated value)
ws['B2'] = '=A2*1.1'
ws['C2'] = '=SUM(A2:B2)'

# Blue text for hardcoded inputs
ws['A2'] = 100
ws['A2'].font = Font(color='0000FF')

# Format as currency
ws['B2'].number_format = '$#,##0'

# Yellow background for key assumptions
yellow = PatternFill(start_color='FFFF00', end_color='FFFF00', fill_type='solid')
ws['A2'].fill = yellow

wb.save('output.xlsx')
```

**Critical rule**: Always use Excel formulas instead of calculating values in Python and hardcoding them. This maintains spreadsheet dynamism.

**Warning**: Never open files with `data_only=True` and save — this permanently replaces formulas with values.

## Formula Recalculation

After creating or modifying files with openpyxl, run recalc.py to calculate formula values:

```bash
python scripts/recalc.py output.xlsx
# Returns JSON: {"success": true} or {"errors": [{"cell": "B2", "error": "#REF!"}]}
```

## Data Validation

```python
from openpyxl.worksheet.datavalidation import DataValidation

# Dropdown list
dv = DataValidation(type="list", formula1='"Option1,Option2,Option3"')
ws.add_data_validation(dv)
dv.add(ws['A1'])

# Number range
dv2 = DataValidation(type="whole", operator="between", formula1=0, formula2=100)
ws.add_data_validation(dv2)
dv2.add(ws['B1'])
```

## Conditional Formatting

```python
from openpyxl.formatting.rule import ColorScaleRule, DataBarRule

# Color scale (green to red)
color_scale = ColorScaleRule(
    start_type='min', start_color='00FF00',
    end_type='max', end_color='FF0000'
)
ws.conditional_formatting.add('A1:A100', color_scale)
```

## Charts

```python
from openpyxl.chart import BarChart, Reference

chart = BarChart()
chart.type = "col"
chart.title = "Sales by Quarter"
chart.y_axis.title = "Revenue ($)"
chart.x_axis.title = "Quarter"

data = Reference(ws, min_col=2, min_row=1, max_row=5, max_col=4)
categories = Reference(ws, min_col=1, min_row=2, max_row=5)
chart.add_data(data, titles_from_data=True)
chart.set_categories(categories)

ws.add_chart(chart, "F1")
```

## Output Structure

All generated spreadsheets go to `outputs/<spreadsheet-name>/` directory.

## Dependencies

```
pip install openpyxl pandas xlsxwriter
```

## Example Prompts

- "Create a financial model for this business with DCF analysis"
- "Build a budget tracker with monthly columns and YTD totals"
- "Extract data from this Excel file and create a summary sheet"
- "Add conditional formatting to highlight values above/below targets"
- "Create a pivot table analysis of this sales data"
