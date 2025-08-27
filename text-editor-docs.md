# How to reuse our rich text editor

We now have a simple, flexible setup that you can drop into any component:

- Core hook: [useTiptapEditor()](cci:1://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/text-editor/useTiptapEditor.ts:35:0-168:2) in [components/text-editor/useTiptapEditor.ts](cci:7://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/text-editor/useTiptapEditor.ts:0:0-0:0)
- Ready-to-use wrapper: `ControlledTiptap` in [components/text-editor/controlled-tiptap.tsx](cci:7://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/text-editor/controlled-tiptap.tsx:0:0-0:0)
- Toolbars you can plug in:
  - Repost toolbar: [components/toolbar/editor-toolbar-repost.tsx](cci:7://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/toolbar/editor-toolbar-repost.tsx:0:0-0:0)
  - Comment toolbar: [components/toolbar/editor-toolbar-comment.tsx](cci:7://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/toolbar/editor-toolbar-comment.tsx:0:0-0:0)
  - Primitive button: [components/toolbar/primitives/toggle-iconbutton.tsx](cci:7://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/toolbar/primitives/toggle-iconbutton.tsx:0:0-0:0)

## 1) Easiest: ControlledTiptap (recommended)
Use when you just need an editor with HTML output and adjustable features.

- Import `ControlledTiptap`
- Provide `content` and `onUpdateHTML`
- Enable features you need

Example:
```tsx
import { ControlledTiptap } from '@/components/text-editor/controlled-tiptap';

function MyEditor() {
  const [html, setHtml] = useState('');

  return (
    <ControlledTiptap
      content={html}
      onUpdateHTML={setHtml}
      features={{
        underline: true,
        color: true,
        link: true,
        image: true,
        highlight: true,
        align: true,
        lists: true,
        table: true,
      }}
    />
  );
}
```

## 2) Add a toolbar (optional)
- Reuse the repost toolbar: [components/toolbar/editor-toolbar-repost.tsx](cci:7://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/toolbar/editor-toolbar-repost.tsx:0:0-0:0)
- Or the comment toolbar: [components/toolbar/editor-toolbar-comment.tsx](cci:7://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/toolbar/editor-toolbar-comment.tsx:0:0-0:0)

How to wire:
- Get an `Editor` ref from `ControlledTiptap` (it forwards a Tiptap `Editor`).
- Pass the `editor` to a toolbar component.

Example with repost toolbar:
```tsx
import ToolbarPlugin from '@/components/toolbar/editor-toolbar-repost';
import type { Editor } from '@tiptap/core';

const editorRef = useRef<Editor | null>(null);

<ControlledTiptap ref={editorRef} ... />
{editorRef.current && (
  <ToolbarPlugin
    editor={editorRef.current}
    onImageUpload={(url) => /* handle uploaded image URL */}
    onTriggerLinkPaste={() => /* open link input */}
    onCommentPaste={() => /* open comment input */}
  />
)}
```

Note: The toolbar buttons use the editor chain (e.g., `editor.chain().focus().toggleBold().run()`), so they work anywhere you plug them in.

## 3) Go lower-level: useTiptapEditor hook
Use when you want full control over rendering and a fully custom toolbar.

- Call [useTiptapEditor({ features, onUpdateHTML, editorProps, handleImageFile })](cci:1://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/text-editor/useTiptapEditor.ts:35:0-168:2)
- Render `<EditorContent editor={editor} />`
- Build your own buttons using editor commands

Example:
```tsx
import { EditorContent } from '@tiptap/react';
import { useTiptapEditor } from '@/components/text-editor/useTiptapEditor';

const editor = useTiptapEditor({
  content: '',
  features: { link: true, image: true, color: true },
  onUpdateHTML: setHtml,
  handleImageFile: (file) => { /* upload or preview */ },
  editorProps: {
    attributes: { class: 'prose prose-invert p-4' },
  },
});

<EditorContent editor={editor} />
```

## 4) Handling images and links
- Image drag-drop/paste: pass `handleImageFile` to [useTiptapEditor](cci:1://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/text-editor/useTiptapEditor.ts:35:0-168:2) or `ControlledTiptap`.
- File upload button: use `FileUploader` (as in [editor-toolbar-repost.tsx](cci:7://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/toolbar/editor-toolbar-repost.tsx:0:0-0:0)).
- Links: use toolbar actions or build your own with `editor.commands.setLink(...)`.

## 5) Rendering read-only HTML
When showing saved content, render sanitized HTML:
```tsx
import DOMPurify from 'dompurify';

<div
  dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }}
/>
```

## 6) Styling and placeholder
- Editor classes can be customized via `editorProps.attributes.class`.
- Placeholder: add a positioned element when `editor.isEmpty`, or set your own placeholder text in `editorProps.attributes.placeholder`.

## When to choose what
- Use `ControlledTiptap`: default choice for new features.
- Add [editor-toolbar-repost.tsx](cci:7://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/toolbar/editor-toolbar-repost.tsx:0:0-0:0) or [editor-toolbar-comment.tsx](cci:7://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/toolbar/editor-toolbar-comment.tsx:0:0-0:0): when you want prebuilt controls fast.
- Use [useTiptapEditor](cci:1://file:///home/godswilljames/Documents/Github/namusoft/ratel/ts-packages/web/src/components/text-editor/useTiptapEditor.ts:35:0-168:2): when building a custom editing experience or custom toolbar.

## Quick checklist for reuse
- Import `ControlledTiptap`
- Pass `content` and `onUpdateHTML`
- Enable required `features`
- Add a toolbar if needed
- Sanitize on display with `DOMPurify` when rendering saved content


