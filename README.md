<img width="4000" height="1700" alt="sceneformtools-sceneformfountain" src="https://github.com/user-attachments/assets/87980288-43c0-48f6-9c68-9b83c5acc589" />

---

A lightweight, high-performance, non-destructive Fountain screenplay parser written in pure JavaScript with TypeScript definitions. Designed for screenwriting tools, code editors, and structural analysis applications.

## Key Features

- Every character in the input string maps directly to a byte offset. Re-serializing a document (`serialize(doc)`) returns the exact source text character-for-character.

- Operates in a single pass with one line of lookahead. `parse()` will never throw an error regardless of input.

- Parses scene headings, section hierarchies (`#`, `##`), notes (`[[...]]`), boneyards (`/*...*/`), and synopses (`=`).

- Line-by-line and block-level breakdown covering action, character cues, dialogue, parentheticals, dual dialogue, transitions, lyrics, and page breaks.

- Pure vanilla JavaScript bundled with TypeScript declarations. Works out of the box in Node.js and browsers.

## Repository Structure
```
.
├── fountain-parser.js      # Core parser (UMD / CommonJS / Browser global)
├── fountain-parser.d.ts    # TypeScript definitions
├── LICENSE                 # License file
└── README.md               # Documentation
```


## Installation

### Option 1: Direct GitHub Dependency (Recommended via npm)

You can install this repository directly as a package dependency via npm without needing an npm registry release:
```
npm install github:sceneform/sceneformfountain
```

Or using yarn:
```
yarn add sceneform/sceneformfountain
```

### Option 2: Direct File Import / Vendor Copy

Since the library consists of a standalone JS file and TypeScript declaration file, you can copy fountain-parser.js and fountain-parser.d.ts into your project vendor folder.

Node.js / CommonJS
```js
const SceneformFountain = require('./fountain-parser.js');
```

Browser Script Tag
```html
<script src="path/to/fountain-parser.js"></script>
<script>
  // Global variable SceneformFountain is automatically available
  const doc = SceneformFountain.parse(scriptText);
</script>
```
---
## Quick Start Example
```html
const SceneformFountain = require('sceneformfountain');

const script = `
Title: THE GREAT ESCAPE
Author: Jane Doe

# Act I

EXT. COFFEE SHOP - DAY

JANE
(smiling)
Is this parser working?

JOHN ^
It sure is!
`;

// 1. Parse the script
const doc = SceneformFountain.parse(script);

// 2. Access title page information
console.log(doc.titlePage.title);  // "THE GREAT ESCAPE"
console.log(doc.titlePage.author); // "Jane Doe"

// 3. Inspect detected scenes and outline
console.log(doc.scenes[0].title);  // "EXT. COFFEE SHOP - DAY"
console.log(doc.scenes[0].time);   // "DAY"
console.log(doc.scenes[0].characters); // ["JANE", "JOHN"]

// 4. Verify dual dialogue detection
console.log(doc.blocks.filter(b => b.dual)); 

// 5. Non-destructive guarantee check
const reconstructed = SceneformFountain.serialize(doc);
console.log(reconstructed === script); // true
```

# API Documentation
```js
SceneformFountain.parse(input, options?)
```
Parses a raw Fountain screenplay string into a structured document object.

## Parameters:

- `input` (`string`): The Fountain screenplay text.

- `options` (`FountainParseOptions`, optional):
- - `caretLine (number)`: The line index of the user's cursor. Used to parse provisional character cues while the user is actively typing.
- - `markers` (`RegExp | RegExp[]`): Regex pattern(s) to classify custom marker lines (e.g., /^@@(END)?ID\b/).
- - `maxCueLength` (`numbe`r, default: `50`): Maximum line character length permitted for auto-detecting character names.

Returns: `FountainParserDocument`

## `SceneformFountain.parseInline(input)`

Parses inline formatting (`**bold**`, `*italic*`, `_underline_`, `***bold italic***`) within a snippet of text. Escaped symbols (`\*`, `\_`) and `snake_case` words are handled automatically.
```js
const runs = SceneformFountain.parseInline("A **very** _important_ test.");
// Returns array of objects:
// [
//   { text: "A ", bold: false, italic: false, underline: false },
//   { text: "very", bold: true, italic: false, underline: false },
//   { text: " ", bold: false, italic: false, underline: false },
//   { text: "important", bold: false, italic: false, underline: true },
//   { text: " test.", bold: false, italic: false, underline: false }
// ]
```
## `SceneformFountain.serialize(doc)`

Reconstructs the original raw source string from a parsed FountainParserDocument object.
```js
const originalText = SceneformFountain.serialize(doc);
```

## Key Data Interfaces

### `FountainParserDocument`
```js
interface FountainParserDocument {
  text: string;                        // Full raw source string
  bom: boolean;                        // UTF-8 BOM indicator
  lines: FountainLine[];               // Line-by-line parser tokens
  blocks: FountainBlock[];             // Grouped screenplay element blocks
  scenes: FountainScene[];             // Structured scene data
  outline: FountainOutlineNode[];      // Ordered scene & section tree
  notes: FountainNote[];               // [[ Notes ]] extracted from text
  omits: FountainOmit[];               // /* Boneyard */ omitted sections
  diagnostics: FountainDiagnostic[];   // Warnings (e.g., unclosed omits, invalid dual dialogue)
  titlePage: Record<string, string>;   // Extracted title page properties
  titleEntries: FountainTitleEntry[];  // Raw title page lines with line indices
  characters: FountainCharacter[];     // Character stats (count, first line, scene appearances)
}
```
`FountainLine`
```js
interface FountainLine {
  index: number;            // Line number (0-based)
  start: number;            // Start character offset in document
  end: number;              // End character offset in document
  eol: string;              // Detected line ending ('\n', '\r\n', or '')
  raw: string;              // Original unparsed text
  type?: FountainLineType;  // E.g., 'scene_heading', 'character', 'action', etc.
  text?: string;            // Cleaned text content
  forced?: boolean;         // True if overridden with Fountain syntax rules (.SCENE, @NAME)
}
```

# License

This project is open source and available under the terms defined in the LICENSE file.
