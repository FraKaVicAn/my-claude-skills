---
name: business-automation
description: Auto-activating business automation skill — email templates, invoice generation, PDF processing, Slack/Discord bots, Google Sheets, Make/n8n/Zapier workflows, approval flows, and report generation
origin: jeremylongshore/claude-code-plugins-plus-skills/19-business-automation
tools: Read, Write, Edit, Bash, Grep
---

# Business Automation Skill

Automated assistance for business process automation: document processing, communication bots, spreadsheet automation, workflow platforms, and report generation.

## When to Activate

Activates automatically when you mention: email template, invoice, PDF, Slack bot, Discord bot, Google Sheets, Excel, Make, n8n, Zapier, approval workflow, report generator, notification, reminder, calendar, meeting scheduler, form builder, survey, batch processing, CSV.

## Covered Skills (28 total)

| Skill | What it does |
|-------|-------------|
| `email-template-generator` | HTML/text email templates with variables |
| `email-parser` | Extract structured data from email content |
| `invoice-generator` | PDF invoice generation with line items |
| `pdf-generator` | PDF creation from templates |
| `pdf-parser` | Extract text and data from PDFs |
| `document-merger` | Combine multiple documents |
| `csv-processor` | CSV transformation and validation |
| `batch-file-processor` | Process thousands of files in parallel |
| `slack-bot-creator` | Slack app with slash commands and events |
| `discord-bot-generator` | Discord.js bot with command handlers |
| `teams-webhook-sender` | Microsoft Teams adaptive card sender |
| `notification-dispatcher` | Multi-channel notification routing |
| `approval-workflow-generator` | Multi-step approval workflow |
| `calendar-event-creator` | Google Calendar / Outlook event creation |
| `meeting-scheduler-helper` | Availability-based meeting scheduling |
| `reminder-system-creator` | Scheduled reminders with escalation |
| `report-generator` | Automated report generation and delivery |
| `survey-creator` | Survey form generation and response collection |
| `google-sheets-automation` | Google Sheets API automation |
| `excel-formula-generator` | Complex Excel formulas and macros |
| `excel-macro-creator` | VBA macro generation |
| `make-scenario-creator` | Make (Integromat) scenario blueprint |
| `n8n-workflow-generator` | n8n workflow JSON export |
| `zapier-integration-helper` | Zapier Zap configuration |
| `form-builder-helper` | Dynamic form schema and validation |

## Slack Bot (Bolt)

```javascript
const { App } = require('@slack/bolt')

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET,
  socketMode: true,
  appToken: process.env.SLACK_APP_TOKEN,
})

// Slash command
app.command('/report', async ({ command, ack, respond }) => {
  await ack()
  const report = await generateReport(command.text)
  await respond({
    text: `Report for: ${command.text}`,
    blocks: [
      { type: 'section', text: { type: 'mrkdwn', text: `*Report: ${command.text}*` } },
      { type: 'section', fields: report.metrics.map(m => ({
        type: 'mrkdwn', text: `*${m.name}:*\n${m.value}`
      }))},
      { type: 'actions', elements: [{
        type: 'button', text: { type: 'plain_text', text: 'Download PDF' },
        action_id: 'download_report', value: report.id
      }]}
    ]
  })
})

// Action handler
app.action('download_report', async ({ action, ack, client, body }) => {
  await ack()
  await client.files.uploadV2({
    channel_id: body.channel.id,
    filename: `report-${action.value}.pdf`,
    file: await generatePDF(action.value),
  })
})

;(async () => await app.start())()
```

## Invoice Generation (Python)

```python
from reportlab.lib.pagesizes import A4
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.lib import colors
from datetime import datetime

def generate_invoice(invoice_data: dict, output_path: str):
    doc = SimpleDocTemplate(output_path, pagesize=A4)
    styles = getSampleStyleSheet()
    elements = []

    # Header
    elements.append(Paragraph(f"INVOICE #{invoice_data['number']}", styles['Title']))
    elements.append(Paragraph(f"Date: {datetime.now().strftime('%Y-%m-%d')}", styles['Normal']))

    # Line items table
    table_data = [["Description", "Qty", "Unit Price", "Total"]]
    for item in invoice_data["items"]:
        table_data.append([item["description"], item["qty"],
                           f"${item['price']:.2f}", f"${item['qty'] * item['price']:.2f}"])
    table_data.append(["", "", "TOTAL", f"${invoice_data['total']:.2f}"])

    table = Table(table_data, colWidths=[250, 50, 100, 100])
    table.setStyle(TableStyle([
        ('BACKGROUND', (0, 0), (-1, 0), colors.grey),
        ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
        ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
        ('ROWBACKGROUNDS', (0, 1), (-1, -2), [colors.white, colors.lightgrey]),
        ('FONTNAME', (0, -1), (-1, -1), 'Helvetica-Bold'),
        ('GRID', (0, 0), (-1, -1), 0.5, colors.grey),
    ]))
    elements.append(table)
    doc.build(elements)
```

## n8n Workflow (JSON)

```json
{
  "name": "Daily Report Email",
  "nodes": [
    {"id": "trigger", "type": "n8n-nodes-base.scheduleTrigger",
     "parameters": {"rule": {"interval": [{"field": "cronExpression", "expression": "0 9 * * 1-5"}]}}},
    {"id": "fetch_data", "type": "n8n-nodes-base.httpRequest",
     "parameters": {"url": "https://api.example.com/daily-stats", "authentication": "headerAuth"}},
    {"id": "send_email", "type": "n8n-nodes-base.emailSend",
     "parameters": {"to": "team@company.com", "subject": "Daily Report {{$now.format('YYYY-MM-DD')}}"}}
  ],
  "connections": {"trigger": {"main": [["fetch_data"]]}, "fetch_data": {"main": [["send_email"]]}}
}
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `19-business-automation` (28 skills)
