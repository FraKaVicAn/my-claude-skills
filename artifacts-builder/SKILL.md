---
name: artifacts-builder
description: Build Claude.ai HTML artifacts using React 18, TypeScript, Tailwind CSS, and shadcn/ui — produces self-contained bundle.html with inlined dependencies
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Edit, Bash
---

# Artifacts Builder Skill

Create polished, self-contained HTML artifacts for Claude.ai using a React 18 + TypeScript + Tailwind + shadcn/ui stack. Everything bundles into a single `bundle.html` with no external dependencies.

## When to Activate

- Building interactive dashboards, visualizations, or tools as Claude.ai artifacts
- Creating self-contained HTML apps that can be shared without a server
- Using shadcn/ui components for professional UI without setup overhead

## Workflow

1. **Initialize** — scaffold component structure
2. **Develop** — build with React, TypeScript, Tailwind
3. **Bundle** — inline all dependencies into bundle.html
4. **Display** — paste into Claude.ai artifact window
5. **Test** — validate interactivity and responsiveness

## Available Components (40+ pre-installed)

From shadcn/ui: Button, Card, Dialog, Table, Tabs, Input, Select, Badge, Alert, Progress, Tooltip, Dropdown, Calendar, Chart, and more.

## Example Structure

```tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { useState } from "react"

export default function MyArtifact() {
  const [count, setCount] = useState(0)
  return (
    <Card className="w-full max-w-md mx-auto mt-8">
      <CardHeader>
        <CardTitle>Counter</CardTitle>
      </CardHeader>
      <CardContent className="flex gap-4 items-center">
        <Button onClick={() => setCount(c => c - 1)}>-</Button>
        <span className="text-2xl font-mono">{count}</span>
        <Button onClick={() => setCount(c => c + 1)}>+</Button>
      </CardContent>
    </Card>
  )
}
```

## Bundle Output

The final `bundle.html` inlines React, ReactDOM, Tailwind CSS, and all component code — paste directly into any Claude.ai artifact slot.

## Links

- [shadcn/ui components](https://ui.shadcn.com)
- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
