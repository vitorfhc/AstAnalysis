---
name: AstAnalysis
description: AST-based code querying for JS/TS. USE WHEN find code patterns OR get function calls OR list event listeners OR extract imports OR query AST.
---

# AstAnalysis

Query JavaScript/TypeScript code using AST parsing. Dynamically generates and runs custom parser code for any pattern you describe.

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **Analyze** | "find", "get", "list", "extract" code patterns | `Workflows/Analyze.md` |

## How It Works

1. **Clarify** (skippable): AI asks clarifying questions to understand exactly what you need
2. **Generate**: AI creates custom TypeScript AST traversal code for your specific query
3. **Execute**: Runs the generated code against your files/directories
4. **Report**: Returns findings with `file:line - code` format

The AI generates fresh AST parsing code for each request, so you can query for any pattern - not limited to predefined queries.

## Examples

**Example 1: Find postMessage listeners**
```
User: "Find all postMessage listeners in src/"
→ AI clarifies: event types? include iframes? (user can skip)
→ Returns file:line for each match
```

**Example 2: Get function calls**
```
User: "List all fetch() calls in ./api"
→ AI clarifies: include axios/got? async only? (user can skip)
→ Returns locations and call details
```

**Example 3: Extract values**
```
User: "Find strings starting with 'sk-' in config/"
→ AI clarifies: variable assignments only? include template literals? (user can skip)
→ Returns matches with locations
```
