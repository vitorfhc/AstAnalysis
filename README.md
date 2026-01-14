# AstAnalysis

A Claude Code skill for querying JavaScript/TypeScript code using AST parsing. Instead of regex-based searching, this skill dynamically generates and runs custom Babel-based parsers for any code pattern you describe.

## Installation

1. Clone this repository into your Claude Code skills directory:
   ```bash
   git clone https://github.com/vitorfhc/AstAnalysis.git ~/.claude/skills/AstAnalysis
   ```

2. Install dependencies:
   ```bash
   cd ~/.claude/skills/AstAnalysis/Tools
   bun install
   ```

3. The skill will be automatically available in Claude Code.

## How It Works

1. **Clarify**: AI asks what exactly you're looking for (you can skip this)
2. **Generate**: AI creates custom TypeScript AST traversal code for your query
3. **Execute**: Runs the generated parser against your target files
4. **Report**: Returns matches in `file:line - code` format

The AI generates fresh parsing code for each request, so you can query for **any pattern** - not limited to predefined queries.

## Usage Examples

**Find postMessage listeners:**
```
/ast-analysis find all postMessage listeners in src/
```

**Get function calls:**
```
/ast-analysis list all fetch() calls in ./api
```

**Extract sensitive values:**
```
/ast-analysis find strings starting with 'sk-' in config/
```

**Find React hooks:**
```
/ast-analysis find all useState calls in components/
```

**Locate event handlers:**
```
/ast-analysis get onClick handlers in src/
```

## Requirements

- [Bun](https://bun.sh) runtime
- Claude Code CLI

## Dependencies

- `@babel/parser` - JavaScript/TypeScript parsing
- `@babel/traverse` - AST traversal
