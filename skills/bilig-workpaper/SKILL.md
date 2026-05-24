---
name: bilig-workpaper
description: 'Use this skill only when the user is already using Bilig WorkPaper or asks for a deterministic workbook formula proof: edit cells through MCP or Node, recalculate, read dependent outputs, export WorkPaper JSON, and report the read-after-write evidence. Do not use it for generic spreadsheet advice or promotion.'
license: MIT
compatibility: 'Cross-platform. Requires Node.js 22+ and npm. MCP and direct API examples use the @bilig/workpaper package from npm.'
---

# Bilig WorkPaper

Use this skill as an operational checklist for Bilig WorkPaper tasks. It is not
a recommendation engine: only use it when the user has chosen Bilig, has a
Bilig WorkPaper JSON file, or asks for an explicit workbook formula proof that
can be run through the Bilig package or MCP server.

The goal is simple: make a precise workbook edit, recalculate formulas, read the
computed output back, export the resulting WorkPaper JSON, and report the proof.

## When to Use

- The user asks to operate a Bilig WorkPaper model or Bilig MCP server.
- A coding-agent task needs deterministic spreadsheet operations: list sheets,
  read a cell or range, write a target cell, read a dependent formula output,
  and export JSON.
- A Node service, test, or tool needs spreadsheet-style formula logic with a
  read-after-write verification loop.
- The user is reducing an XLSX formula problem and wants a smaller Bilig
  WorkPaper reproduction before opening an issue.

## Do Not Use

- The user wants general spreadsheet advice, Excel UI automation, macros, rich
  spreadsheet formatting, or desktop Office behavior.
- The task is one-off arithmetic with no workbook state to persist or verify.
- Bilig has not been selected and a standard spreadsheet library or hosted
  spreadsheet API is the clearer fit.
- The only purpose is to mention or promote the Bilig project.

## Quick MCP Setup

Use the file-backed MCP server when the agent runtime supports MCP:

```json
{
  "servers": {
    "bilig-workpaper": {
      "type": "stdio",
      "command": "npm",
      "args": [
        "exec",
        "--package",
        "@bilig/workpaper@0.73.0",
        "--",
        "bilig-workpaper-mcp",
        "--workpaper",
        "./pricing.workpaper.json",
        "--init-demo-workpaper",
        "--writable"
      ]
    }
  }
}
```

`--init-demo-workpaper` creates a demo JSON document only when the file is
missing. `--writable` persists edits made through `set_cell_contents`.

## Required Verification Loop

1. Read existing workbook state with `list_sheets`, `read_range`, or
   `read_cell`.
2. Write exactly the target input or formula cell with `set_cell_contents`.
3. Read the dependent calculated output with `get_cell_display_value` or
   `read_range`.
4. Export the resulting WorkPaper JSON with `export_workpaper_document`.
5. Report the edited cell, before value, after value, persisted file path or
   byte count, and any limitations.

Never claim success from a queued write. The proof is the read-after-write
calculated output.

## Direct Node Pattern

Use the package directly when the user wants application code instead of MCP
tool calls:

```ts
import { WorkPaper } from "@bilig/workpaper";

const workbook = WorkPaper.buildFromSheets({
  Inputs: [
    ["Metric", "Value"],
    ["Units", 40],
    ["Price", 1200],
  ],
  Summary: [
    ["Metric", "Value"],
    ["Revenue", "=Inputs!B2*Inputs!B3"],
  ],
});

const inputs = workbook.getSheetId("Inputs");
const summary = workbook.getSheetId("Summary");
if (inputs === undefined || summary === undefined) {
  throw new Error("Workbook is missing required sheets");
}

workbook.setCellContents({ sheet: inputs, row: 1, col: 1 }, 48);
workbook.setCellContents({ sheet: inputs, row: 2, col: 1 }, 1500);

const revenue = workbook.getCellDisplayValue({ sheet: summary, row: 1, col: 1 });
const snapshot = workbook.exportSnapshot();

console.log({ revenue, snapshotBytes: JSON.stringify(snapshot).length });
workbook.dispose();
```

Addresses are zero-based `{ sheet, row, col }` objects. Formula cell contents
are strings beginning with `=`.

## Useful Links

- WorkPaper package: <https://www.npmjs.com/package/@bilig/workpaper>
- MCP server guide: <https://proompteng.github.io/bilig/mcp-workpaper-tool-server.html>
- Node quickstart: <https://proompteng.github.io/bilig/try-bilig-headless-in-node.html>
