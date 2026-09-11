---
id: nodez-code-editor-implementation
title: Code Editor Implementation
type: architecture
status: active
created: 2026-08-21
updated: 2026-09-11
tags:
  - editor
  - codemirror
  - frontend
  - roadmap
---

# Code Editor Implementation

Related: [[Next Steps]], [[Frontend]], [[Product Roadmap]].

Current Markdown note editor in the app repo: `src/features/notes/MarkdownEditor.tsx` (CodeMirror 6 + Markdown language + format toolbar via `App.tsx`). **MVP code mode landed** in `src/features/editor/components/` (lazy-loaded from the topbar Code button): languages TS/JS/JSON/HTML/CSS/MD, Prettier format, structural lint gutter. This note remains the deeper plan (richer autocomplete, ESLint, multi-file). Work shipped past the MVP is recorded below.

## Shipped past the MVP

### Right-click context menu and Ask AI (2026-09-11)

The editor had no context menu at all, so right-clicking fell through to the
native WebKit menu. It now carries its own, acting on the current selection:
Cut, Copy, Paste, Select All, Find, Go to Line, Toggle Comment, and Ask AI.

- `src/features/editor/components/contextMenuItems.ts` — the item model and
  placement, kept pure so it is testable without a DOM. `buildEditorContextMenu`
  decides labels, platform shortcut hints and enabled state;
  `clampMenuPosition` flips the menu back across the cursor at a viewport edge.
- `src/features/editor/components/EditorContextMenu.tsx` — renders the items,
  following the `.treeContextMenu` pattern already used by the note tree.
- `src/features/editor/components/editorCommands.ts` — gained `select-all`,
  `cut`, `copy` and `paste`. The command map is a `Record<EditorCommandId, …>`,
  so TypeScript, not a test, enforces that every menu id resolves to a command.

Right-clicking outside the selection moves the caret there first; inside, the
selection is kept. Dismissing returns focus to the editor.

**Ask AI reuses the chat panel's existing context plumbing** rather than adding a
second path. `CodexChatPanel` already received `editorContext` (path, content,
`from`/`to`) and already had `includeCode` and `selectionOnly` toggles, so the
menu item threads a callback up to `App.tsx`, which opens the chat sidebar and
bumps an `includeCodeRequest` nonce — the same shape as the `includeNoteRequest`
behind the note's Ask button. The panel pins the context, sets `selectionOnly`
only when the range is non-empty, and focuses the composer **without sending**.
See [[Codex Chat Implementation]].

### Two things worth remembering

**A menu must not arm its own dismissal during the gesture that opened it.** The
first build looked completely inert — no custom menu, and no native menu either.
The `preventDefault()` had suppressed the native one, and the opening
`contextmenu` event then carried on bubbling to the `window` listener that closes
the menu, destroying it during the frame it spends hidden while measuring itself
for placement. Nothing was ever painted, so the symptom read as "right-click does
nothing" rather than as a flicker. `armMenuDismiss` now defers attaching those
listeners by one animation frame, and holds `onDismiss` in a ref so a
re-rendering parent cannot disarm and re-arm them.

**Paste depends on a clipboard read the webview may withhold.** Cut and Copy use
`navigator.clipboard.writeText`, which the app already relies on elsewhere.
Paste needs `readText`, which WKWebView can refuse. The menu gates the item on a
capability check up front, and if the read is refused at call time the editor
latches the failure so every later open shows Paste disabled with a "Use ⌘V"
hint instead of a dead item.

## Draft implementation guide

The detailed agent brief below is preserved as plain text (draft formatting).

```text
Goal
Implement a lightweight, VS Code-like code editor in the Nodez app with:
- syntax highlighting
- line numbers
- bracket matching
- basic autocomplete
- format code action
- lightweight lint diagnostics
- dark/light theme support
- lazy-loaded editor dependencies
Use CodeMirror 6 as the editor engine, Prettier standalone for formatting, and CodeMirror lint diagnostics for lightweight linting.
CodeMirror is the preferred editor because it is modular and lightweight. Monaco is the editor that powers VS Code, but it is heavier and more complex for this requirement. Microsoft GitHub
1. Technical Decision
Use this stack
CodeMirror 6
Prettier standalone
@codemirror/lint
Optional: ESLint browser integration later
Do not use Monaco for MVP
Monaco gives the closest VS Code feel, but it adds more bundle weight and worker setup complexity. It should only be considered later if Nodez needs full IDE-like behavior such as deep TypeScript IntelliSense, multi-file project awareness, or advanced language services.
2. Install Dependencies
For a React or Next.js app:
npm install codemirror \
  @codemirror/state \
  @codemirror/view \
  @codemirror/commands \
  @codemirror/language \
  @codemirror/autocomplete \
  @codemirror/search \
  @codemirror/lint \
  @codemirror/lang-javascript \
  @codemirror/lang-json \
  @codemirror/lang-html \
  @codemirror/lang-css \
  @codemirror/theme-one-dark \
  prettier
CodeMirror’s lint package is designed to show errors and warnings inside the editor through diagnostics. CodeMirror Prettier supports browser usage through its standalone build and explicit plugins. Prettier
3. Create File Structure
Ask the agent to create this structure:
src/
  components/
    code-editor/
      CodeEditor.tsx
      codeEditorExtensions.ts
      formatCode.ts
      lintCode.ts
      languageExtensions.ts
      types.ts
Expected responsibilities:
CodeEditor.tsx
- Main reusable editor component.

codeEditorExtensions.ts
- Base CodeMirror setup: line numbers, keymaps, bracket matching, search, theme.

formatCode.ts
- Prettier formatting logic.

lintCode.ts
- Lightweight diagnostics.

languageExtensions.ts
- Language-specific CodeMirror extensions.

types.ts
- Shared TypeScript types.
4. Define Editor Types
Create:
// src/features/editor/components/types.ts

export type SupportedCodeLanguage =
  | 'javascript'
  | 'typescript'
  | 'json'
  | 'html'
  | 'css';

export type CodeEditorTheme = 'light' | 'dark';

export interface CodeEditorProps {
  value: string;
  language: SupportedCodeLanguage;
  theme?: CodeEditorTheme;
  readOnly?: boolean;
  height?: string;
  onChange?: (value: string) => void;
  onFormat?: (value: string) => void;
  onLintChange?: (errorCount: number) => void;
}
5. Add Language Extensions
Create:
// src/features/editor/components/languageExtensions.ts

import { javascript } from '@codemirror/lang-javascript';
import { json } from '@codemirror/lang-json';
import { html } from '@codemirror/lang-html';
import { css } from '@codemirror/lang-css';
import type { Extension } from '@codemirror/state';
import type { SupportedCodeLanguage } from './types';

export function getLanguageExtension(language: SupportedCodeLanguage): Extension {
  switch (language) {
    case 'javascript':
      return javascript({ jsx: true });
    case 'typescript':
      return javascript({ typescript: true, jsx: true });
    case 'json':
      return json();
    case 'html':
      return html();
    case 'css':
      return css();
    default:
      return [];
  }
}
6. Add Basic Editor Extensions
Create:
// src/features/editor/components/codeEditorExtensions.ts

import { lineNumbers, highlightActiveLineGutter } from '@codemirror/view';
import { highlightActiveLine, keymap } from '@codemirror/view';
import { bracketMatching, indentOnInput, syntaxHighlighting, defaultHighlightStyle } from '@codemirror/language';
import { defaultKeymap, history, historyKeymap } from '@codemirror/commands';
import { searchKeymap, highlightSelectionMatches } from '@codemirror/search';
import { autocompletion, completionKeymap } from '@codemirror/autocomplete';
import { lintGutter, lintKeymap } from '@codemirror/lint';
import type { Extension } from '@codemirror/state';

export function getBaseEditorExtensions(readOnly = false): Extension[] {
  return [
    lineNumbers(),
    highlightActiveLineGutter(),
    history(),
    bracketMatching(),
    indentOnInput(),
    autocompletion(),
    highlightActiveLine(),
    highlightSelectionMatches(),
    syntaxHighlighting(defaultHighlightStyle, { fallback: true }),
    lintGutter(),
    keymap.of([
      ...defaultKeymap,
      ...historyKeymap,
      ...searchKeymap,
      ...completionKeymap,
      ...lintKeymap,
    ]),
  ];
}
7. Add Lightweight Linting
Start simple. Do not add full ESLint in the MVP unless required.
Create:
// src/features/editor/components/lintCode.ts

import { linter, type Diagnostic } from '@codemirror/lint';
import type { SupportedCodeLanguage } from './types';

export function createLightweightLinter(language: SupportedCodeLanguage) {
  return linter((view) => {
    const code = view.state.doc.toString();
    const diagnostics: Diagnostic[] = [];

    if (!code.trim()) return diagnostics;

    if (language === 'json') {
      try {
        JSON.parse(code);
      } catch (error) {
        diagnostics.push({
          from: 0,
          to: Math.min(code.length, 1),
          severity: 'error',
          message: error instanceof Error ? error.message : 'Invalid JSON',
        });
      }
    }

    if (language === 'javascript' || language === 'typescript') {
      try {
        // Lightweight syntax check only.
        // Note: this does not fully validate TypeScript syntax.
        if (language === 'javascript') {
          new Function(code);
        }
      } catch (error) {
        diagnostics.push({
          from: 0,
          to: Math.min(code.length, 1),
          severity: 'error',
          message: error instanceof Error ? error.message : 'Syntax error',
        });
      }
    }

    return diagnostics;
  });
}
Important note for the agent: this MVP linter is intentionally basic. CodeMirror supports proper linter integration via diagnostics, and lintGutter can show errors in the gutter. CodeMirror
8. Add Prettier Formatting
Create:
// src/features/editor/components/formatCode.ts

import * as prettier from 'prettier/standalone';
import * as babelPlugin from 'prettier/plugins/babel';
import * as estreePlugin from 'prettier/plugins/estree';
import * as htmlPlugin from 'prettier/plugins/html';
import * as postcssPlugin from 'prettier/plugins/postcss';
import type { SupportedCodeLanguage } from './types';

function getParser(language: SupportedCodeLanguage) {
  switch (language) {
    case 'javascript':
      return 'babel';
    case 'typescript':
      return 'typescript';
    case 'json':
      return 'json';
    case 'html':
      return 'html';
    case 'css':
      return 'css';
    default:
      return 'babel';
  }
}

function getPlugins(language: SupportedCodeLanguage) {
  switch (language) {
    case 'javascript':
    case 'typescript':
    case 'json':
      return [babelPlugin, estreePlugin];
    case 'html':
      return [htmlPlugin];
    case 'css':
      return [postcssPlugin];
    default:
      return [babelPlugin, estreePlugin];
  }
}

export async function formatCode(
  code: string,
  language: SupportedCodeLanguage
): Promise<string> {
  return prettier.format(code, {
    parser: getParser(language),
    plugins: getPlugins(language),
    semi: true,
    singleQuote: true,
    trailingComma: 'es5',
  });
}
Prettier’s browser usage requires the standalone build and explicit plugin loading. For JavaScript, TypeScript, Flow, and JSON printing, the ESTree plugin is required. Prettier
9. Build the Editor Component
Create:
// src/features/editor/components/CodeEditor.tsx

'use client';

import { useEffect, useMemo, useRef } from 'react';
import { EditorState, Compartment } from '@codemirror/state';
import { EditorView } from '@codemirror/view';
import { oneDark } from '@codemirror/theme-one-dark';
import { getBaseEditorExtensions } from './codeEditorExtensions';
import { getLanguageExtension } from './languageExtensions';
import { createLightweightLinter } from './lintCode';
import { formatCode } from './formatCode';
import type { CodeEditorProps } from './types';

export function CodeEditor({
  value,
  language,
  theme = 'light',
  readOnly = false,
  height = '320px',
  onChange,
  onFormat,
}: CodeEditorProps) {
  const parentRef = useRef<HTMLDivElement | null>(null);
  const viewRef = useRef<EditorView | null>(null);

  const editableCompartment = useMemo(() => new Compartment(), []);
  const languageCompartment = useMemo(() => new Compartment(), []);
  const themeCompartment = useMemo(() => new Compartment(), []);

  useEffect(() => {
    if (!parentRef.current || viewRef.current) return;

    const updateListener = EditorView.updateListener.of((update) => {
      if (update.docChanged) {
        onChange?.(update.state.doc.toString());
      }
    });

    const state = EditorState.create({
      doc: value,
      extensions: [
        ...getBaseEditorExtensions(readOnly),
        languageCompartment.of(getLanguageExtension(language)),
        createLightweightLinter(language),
        editableCompartment.of(EditorView.editable.of(!readOnly)),
        themeCompartment.of(theme === 'dark' ? oneDark : []),
        updateListener,
        EditorView.theme({
          '&': {
            height,
          },
          '.cm-scroller': {
            overflow: 'auto',
          },
        }),
      ],
    });

    viewRef.current = new EditorView({
      state,
      parent: parentRef.current,
    });

    return () => {
      viewRef.current?.destroy();
      viewRef.current = null;
    };
  }, []);

  useEffect(() => {
    const view = viewRef.current;
    if (!view) return;

    const currentValue = view.state.doc.toString();
    if (value !== currentValue) {
      view.dispatch({
        changes: {
          from: 0,
          to: currentValue.length,
          insert: value,
        },
      });
    }
  }, [value]);

  useEffect(() => {
    const view = viewRef.current;
    if (!view) return;

    view.dispatch({
      effects: languageCompartment.reconfigure(getLanguageExtension(language)),
    });
  }, [language, languageCompartment]);

  useEffect(() => {
    const view = viewRef.current;
    if (!view) return;

    view.dispatch({
      effects: editableCompartment.reconfigure(EditorView.editable.of(!readOnly)),
    });
  }, [readOnly, editableCompartment]);

  useEffect(() => {
    const view = viewRef.current;
    if (!view) return;

    view.dispatch({
      effects: themeCompartment.reconfigure(theme === 'dark' ? oneDark : []),
    });
  }, [theme, themeCompartment]);

  async function handleFormat() {
    const view = viewRef.current;
    if (!view) return;

    const currentCode = view.state.doc.toString();
    const formatted = await formatCode(currentCode, language);

    view.dispatch({
      changes: {
        from: 0,
        to: currentCode.length,
        insert: formatted,
      },
    });

    onFormat?.(formatted);
    onChange?.(formatted);
  }

  return (
    <div className="nodez-code-editor">
      <div className="mb-2 flex items-center justify-between">
        <span className="text-sm opacity-70">{language}</span>
        <button
          type="button"
          onClick={handleFormat}
          className="rounded-md border px-3 py-1 text-sm"
        >
          Format
        </button>
      </div>

      <div ref={parentRef} className="overflow-hidden rounded-lg border" />
    </div>
  );
}
10. Add Example Usage
'use client';

import { useState } from 'react';
import { CodeEditor } from '@/components/code-editor/CodeEditor';

export default function CodeEditorDemo() {
  const [code, setCode] = useState(`function hello(name){console.log("Hello "+name)}`);

  return (
    <CodeEditor
      value={code}
      language="javascript"
      theme="dark"
      height="400px"
      onChange={setCode}
      onFormat={(formatted) => console.log('Formatted:', formatted)}
    />
  );
}
11. Add Lazy Loading
For Next.js, import the editor dynamically so it does not load on every page.
import dynamic from 'next/dynamic';

const CodeEditor = dynamic(
  () => import('@/components/code-editor/CodeEditor').then((m) => m.CodeEditor),
  {
    ssr: false,
    loading: () => <div className="h-80 rounded-lg border p-4">Loading editor...</div>,
  }
);
This is important because the requirement is lightweight.
12. Performance Requirements
The agent should follow these rules:
1. Do not load Monaco.
2. Do not load all CodeMirror language packages.
3. Only include languages Nodez actually supports.
4. Lazy-load the editor component.
5. Run Prettier only when the user clicks Format or saves.
6. Debounce linting if custom linting becomes expensive.
7. Avoid full ESLint in MVP.
8. Keep editor state local unless app-level persistence is required.
13. Optional Phase 2: Real ESLint Support
Only add this if users need real JavaScript/TypeScript lint rules.
Possible packages:
npm install eslint-linter-browserify
CodeMirror’s JavaScript language package can connect ESLint’s Linter class to CodeMirror linting, and browserified ESLint can help in browser environments. npm
Add this only after measuring bundle size.
14. Acceptance Criteria
The AI agent should verify:
Editor
- User can type code.
- User can select supported language.
- Syntax highlighting works.
- Line numbers are visible.
- Bracket matching works.
- Undo and redo work.
- Search shortcut works.
- Dark mode works.

Formatting
- Format button formats JavaScript.
- Format button formats TypeScript.
- Format button formats JSON.
- Format button formats HTML.
- Format button formats CSS.
- Formatting errors are shown gracefully.

Linting
- Invalid JSON shows an error.
- Basic JavaScript syntax error shows an error.
- Lint gutter appears.
- Editor does not freeze on invalid code.

Performance
- Editor is lazy-loaded.
- Prettier is not run on every keystroke.
- Monaco is not included in bundle.
- No unnecessary language packages are loaded.
15. Final Instruction for the Other AI Agent
Use CodeMirror 6 for the editor, Prettier standalone for formatting, and CodeMirror diagnostics for lightweight linting. Keep the first version intentionally small. Do not implement Monaco unless the product later requires full VS Code-level language services.ote 13
```
