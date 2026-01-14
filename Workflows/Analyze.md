# Analyze Workflow

Dynamically generate and run AST parsing code to find any code pattern in JS/TS files.

**CRITICAL: NEVER read JS/TS files directly using Read tool. This wastes tokens. Only use the generated AST parser to analyze files.**

## Step 1: Clarify Requirements (MANDATORY)

You MUST use **AskUserQuestion** before proceeding. This step cannot be skipped by the AI - only the user can skip by selecting the "Skip" option.

```
Question: "What exactly should I look for?"
Options:
1. [Specific interpretation of user's request]
2. [Alternative interpretation]
3. [Related pattern they might also want]
4. "Skip - just work with what I said"
```

If user selects "Skip", proceed with your best interpretation. Otherwise, incorporate their clarification.

## Step 2: Create Temp Directory

First, create a temporary directory using:

```bash
mktemp -d
```

Save the returned path - you'll use it in the next step.

## Step 3: Generate AST Parser Code

Use the **Write** tool (NOT Bash) to create a TypeScript file at `<temp-dir>/query.ts`.

The file should:
1. Use `@babel/parser` and `@babel/traverse` to parse files
2. Traverse the AST looking for the specific pattern
3. Output matches as `file:line - code`

**Template structure:**

```typescript
#!/usr/bin/env bun
import { parse } from "@babel/parser";
import traverse from "@babel/traverse";
import * as fs from "fs";
import * as path from "path";

// Get all files from path (file or directory)
// No extension filtering - we try to parse everything
function getFiles(inputPath: string): string[] {
  const stats = fs.statSync(inputPath);
  if (stats.isFile()) return [inputPath];

  const files: string[] = [];

  function walk(dir: string) {
    for (const entry of fs.readdirSync(dir, { withFileTypes: true })) {
      const fullPath = path.join(dir, entry.name);
      if (entry.isDirectory() && !entry.name.startsWith(".") && entry.name !== "node_modules") {
        walk(fullPath);
      } else if (entry.isFile()) {
        files.push(fullPath);
      }
    }
  }
  walk(inputPath);
  return files;
}

// Try to parse a file - returns null if not parseable as JS/TS
function tryParse(filePath: string) {
  try {
    const code = fs.readFileSync(filePath, "utf-8");
    const ast = parse(code, {
      sourceType: "module",
      plugins: ["typescript", "jsx", "decorators-legacy"],
      errorRecovery: true,
    });
    return { ast, code };
  } catch {
    return null;
  }
}

// Get line of code
function getLine(code: string, line: number): string {
  return code.split("\n")[line - 1]?.trim() || "";
}

const inputPath = path.resolve(process.argv[2] || ".");
const files = getFiles(inputPath);
console.log(`Found ${files.length} file(s), attempting to parse...\n`);

const matches: Array<{file: string, line: number, code: string}> = [];
let parsedCount = 0;
let failedFiles: string[] = [];

for (const file of files) {
  const result = tryParse(file);
  if (!result) {
    failedFiles.push(file);
    continue;
  }
  parsedCount++;
  const { ast, code } = result;

  traverse(ast, {
    // === CUSTOM VISITOR LOGIC HERE ===
    // The AI fills this in based on what the user wants to find
  });
}

console.log(`Successfully parsed ${parsedCount} file(s)`);
if (failedFiles.length > 0) {
  console.log(`Could not parse ${failedFiles.length} file(s) as JS/TS`);
}
console.log("");

if (matches.length === 0) {
  console.log("No matches found.");
} else {
  console.log(`Found ${matches.length} match(es):\n`);
  for (const m of matches) {
    console.log(`${path.relative(process.cwd(), m.file)}:${m.line} - ${m.code}`);
  }
}
```

**Fill in the visitor based on the user's request. Examples:**

For "postMessage listeners":
```typescript
CallExpression(p) {
  const callee = p.node.callee;
  if (callee.type === "MemberExpression" &&
      callee.property.type === "Identifier" &&
      callee.property.name === "addEventListener") {
    const firstArg = p.node.arguments[0];
    if (firstArg?.type === "StringLiteral" && firstArg.value === "message") {
      matches.push({ file, line: p.node.loc!.start.line, code: getLine(code, p.node.loc!.start.line) });
    }
  }
}
```

For "fetch calls":
```typescript
CallExpression(p) {
  const callee = p.node.callee;
  if ((callee.type === "Identifier" && callee.name === "fetch") ||
      (callee.type === "MemberExpression" && callee.property.type === "Identifier" && callee.property.name === "fetch")) {
    matches.push({ file, line: p.node.loc!.start.line, code: getLine(code, p.node.loc!.start.line) });
  }
}
```

## Step 4: Run the Code

Run the generated code from the Tools directory (where dependencies are installed), saving output to the temp directory:

```bash
cd ~/.claude/skills/AstAnalysis/Tools && bun run <temp-dir>/query.ts <target-path> > <temp-dir>/output.txt 2>&1
```

## Step 5: Report Results

Do NOT read the output file. Simply tell the user where the output is saved:

```
Results saved to: <temp-dir>/output.txt
```
