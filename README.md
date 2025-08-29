# Complete Nextra 4 Guide

This document consolidates all the essential information for understanding and immediately using Nextra 4 in practice, based on the structure of this repository (api-docs-site). It covers installation/configuration, routing, document writing (MDX), theme usage and customization, search, and conventions.

## TL;DR

- Next.js 15.3.4 App Router + Nextra 4.2.17 + `nextra-theme-docs` combination
- React 19.0.0 support
- Actual documents are managed as `.mdx` files in the `content/` folder
- `app/[[...mdxPath]]/page.jsx` connects all MDX routing with SEO meta and TOC
- Global layout is configured using `Layout` from `nextra-theme-docs` in `app/layout.tsx`
- Search uses Pagefind, generating indexes in `postbuild` and placing them in `public/_pagefind`
- Supports mixed TypeScript and JavaScript (both `.tsx` and `.jsx` files can be used)

---

## Project Structure (This Repository)

### Actual File Structure
```
api-docs-site/
├── app/
│   ├── [[...mdxPath]]/
│   │   └── page.jsx          # Dynamic MDX routing
│   └── layout.tsx            # Global layout
├── content/
│   ├── index.mdx             # Homepage
│   ├── about.mdx             # About page
│   └── built-in-components.mdx # Component documentation
├── docs/
│   └── nextra4-guide.md      # This guide document
├── public/
│   └── images/
│       └── general/
│           ├── icon.svg      # Favicon
│           └── logo.svg      # Logo
├── .gitignore                # Git exclusion settings
├── mdx-components.js         # MDX component configuration
├── next-env.d.ts             # Next.js type definitions
├── next.config.mjs           # Next.js + Nextra settings
├── package.json              # Package definitions
├── pnpm-lock.yaml           # pnpm lock file
├── README.md                # Project description
└── tsconfig.json            # TypeScript settings
```

### Core File Roles
- `next.config.mjs`: Apply `nextra()` plugin, configure options (`search`, `defaultShowCopyCode`)
- `app/layout.tsx`: Global layout, configure `Layout`/`Navbar`/`Footer`/`Head`/`Banner`, inject sidebar data with `getPageMap()`
- `app/[[...mdxPath]]/page.jsx`: Dynamic route. Load MDX pages and pass SEO meta/TOC through `generateStaticParamsFor`, `importPage`
- `mdx-components.js`: Import theme MDX components and merge with custom components as needed
- `content/`: Actual document (MDX) location. `index.mdx` is home (`/`)
- `public/`: Static assets (images, etc.) and `public/_pagefind` (search index - generated at build)
- `package.json` scripts: `dev`, `build`, `start`, `postbuild(pagefind indexing)`
- `next-env.d.ts`: Next.js type definition file (auto-generated, no modification needed)
- `.gitignore`: Excludes `.next`, `node_modules`, `/.idea`

---

## Core Nextra 4 Concepts

### 1) App Router Integration

- This repository uses Next.js App Router. `app/[[...mdxPath]]/page.jsx` absorbs all MDX paths.
- Key APIs
  - `generateStaticParamsFor('mdxPath')` from `nextra/pages`: Generate static paths corresponding to all MDX files in `content/`
  - `importPage(path)` from `nextra/pages`: Dynamically load specific MDX and return `{ default: MDXContent, toc, metadata }`
  - `getPageMap()` from `nextra/page-map`: Generate page map needed for entire document tree (sidebar/navigation)

### 2) Theme and Layout

- Import `Layout`, `Navbar`, `Footer` components from `nextra-theme-docs` directly and use them in `app/layout.tsx`
- Pass `pageMap={await getPageMap()}` to enable theme to render sidebar/TOC/navigation
- Set the base for each document's "Edit this page" link with `docsRepositoryBase`
- Add common `<head>` tags like favicon with `Head` component

### 3) Content Directory

- Documents are located in the `content/` folder by default
- File path and route mapping
  - `content/index.mdx` → `/`
  - `content/about.mdx` → `/about`
  - `content/built-in-components.mdx` → `/built-in-components`
- Sidebar/sorting/titles can be controlled with `_meta.json` (see 'Sidebar and _meta.json' below)
- **Without _meta.json**: Automatically sorted alphabetically by filename, and filenames are displayed as-is in the sidebar

### 4) MDX and Built-in Components

- Write documents in MDX. You can use Markdown + React components together
- Import and use built-in components from `nextra/components` directly
- Example in this repo: Using `<Cards>`, `<Steps>` in `content/index.mdx`

Built-in Components (Check official docs)
- Source: https://nextra.site/docs/built-ins and subpages

Layout Components
- `Banner` — https://nextra.site/docs/built-ins/banner
- `Head` — https://nextra.site/docs/built-ins/head
- `Search` — https://nextra.site/docs/built-ins/search

Content Components
- `Bleed` — https://nextra.site/docs/built-ins/bleed
- `Callout` — https://nextra.site/docs/built-ins/callout
- `Cards` — https://nextra.site/docs/built-ins/cards
- `FileTree` — https://nextra.site/docs/built-ins/filetree
- `Steps` — https://nextra.site/docs/built-ins/steps
- `Table` — https://nextra.site/docs/built-ins/table
- `Tabs` — https://nextra.site/docs/built-ins/tabs

Other Components
- `MDXRemote` — https://nextra.site/docs/built-ins/mdxremote
- `Playground` — https://nextra.site/docs/built-ins/playground
- `TSDoc` — https://nextra.site/docs/built-ins/tsdoc

#### FileTree Component
A built-in component to visually represent a file tree.

Exported from `nextra/components`.

**Props**
- `...props`: `HTMLAttributes<HTMLUListElement>` - All standard HTML UL element attributes

**Usage**
Create the file tree structure by nesting `<FileTree.Folder>` and `<FileTree.File>` components within a `<FileTree>`. Name each file or folder with the `name` attribute. Use `defaultOpen` to set the folder to open on load.

```mdx
import { FileTree } from 'nextra/components'
 
<FileTree>
  <FileTree.Folder name="content" defaultOpen>
    <FileTree.File name="_meta.js" />
    <FileTree.File name="contact.md" />
    <FileTree.File name="index.mdx" />
    <FileTree.Folder name="about">
      <FileTree.File name="_meta.js" />
      <FileTree.File name="legal.md" />
      <FileTree.File name="index.mdx" />
    </FileTree.Folder>
  </FileTree.Folder>
</FileTree>
```

#### MDXRemote Component
A React component that renders compiled MDX content.

Exported from `nextra/mdx-remote`.

**Props**
- `components`: `MDXComponents` - An object mapping names to React components. The key used will be the name accessible to MDX.
- `scope`: `Scope` - Pass-through variables for use in the MDX content. These variables will be available in the MDX scope.
- `compiledSource`: `string` - Raw JavaScript compiled MDX source code, a result of Nextra's `compileMdx` function.

**Example**
```mdx
import { compileMdx } from 'nextra/compile'
import { MDXRemote } from 'nextra/mdx-remote'
 
<MDXRemote
  compiledSource={await compileMdx('# Hello {myVariable} <MyComponent />')}
  components={{ MyComponent: () => <div>My Component</div> }}
  scope={{ myVariable: 'World' }}
/>
```

Note
- This repo actually imports and uses `Banner`, `Head` from `nextra/components` (ref: `app/layout.tsx`)
- For detailed props/slots/subcomponents of each component, refer to the respective official pages

### 5) Search (Pagefind)

- Enable `search: true` in `next.config.mjs`, and generate Pagefind index in `postbuild` of `package.json`:
  - `pagefind --site .next/server/app --output-path public/_pagefind`
- Development server (`next dev`) doesn't guarantee complete index. For accurate search behavior, verify after `build` statically

---

## Configuration Files and Code Flow

### next.config.mjs

```js
import nextra from "nextra";

const withNextra = nextra({
  search: true,
  defaultShowCopyCode: true,
});

export default withNextra({
  // Add other Next.js options as needed
  // output: 'export'
});
```

- `search`: Enable Nextra search UI
- `defaultShowCopyCode`: Display default copy button on code blocks
- You can specify general Next options like MDX settings (`mdxOptions`) or base path

### app/layout.tsx

Key Points
- Pass `pageMap`, `navbar`, `footer`, `banner`, `docsRepositoryBase`, etc. to `Layout`
- Inject common `<head>` tags with `Head` (pass additional tags as children)
- Display top notification banner with `Banner` component (save close state with storageKey)
- Can use Next.js `metadata` API together
- Required HTML attributes:
  - `lang="en"`: Language setting for SEO
  - `dir="ltr"`: Text direction setting (required)
  - `suppressHydrationWarning`: next-themes package compatibility

### app/[[...mdxPath]]/page.jsx

Key Points
- `generateStaticParams`: Generate all document paths statically
- `generateMetadata`: Connect `metadata` read from each document MDX to Next SEO meta
- `importPage`: Load MDX page and pass `{toc, metadata}` to wrapper
- `wrapper`: Wrap and render with theme-provided page wrapper (TOC/layout context)

### mdx-components.js

```js
import { useMDXComponents as getThemeComponents } from 'nextra-theme-docs'

const themeComponents = getThemeComponents()

export function useMDXComponents(components) {
  return { ...themeComponents, ...components }
}
```

- Pattern for overlaying custom components on top of theme default MDX components
- Allows custom components to be used directly in MDX

---

## Document Writing Guide (MDX)

### 1) Frontmatter

Write YAML Frontmatter at the top of document files to control title/description/options.

```mdx
---
title: Nextra Docs Starter
description: This document explains the overview of Nextra 4 starter.
toc: true
---

# Page H1 Title
Content...
```

Commonly used fields (may vary by theme/version)
- `title`: Page title (used in sidebar/header)
- `description`: SEO and preview description
- `toc`: Whether to display table of contents (true/false)

### 2) Links and Images

- Internal links: Connect with route paths like `[text](/about)`
- Images: Reference static files under `public/` with absolute paths like `/images/...`

```mdx
![Logo](/images/general/logo.svg)
```

### 3) Code Blocks

- Highlighting with language specification: ```ts, ```bash, etc.
- Display copy button with `defaultShowCopyCode: true` option

```ts
export function add(a: number, b: number) {
  return a + b
}
```

### 4) Built-in Component Examples

Import components at the top of MDX and use them.

```mdx
import { Callout, Steps, Cards, Tabs, FileTree } from 'nextra/components'

# Built-in Examples

<Callout type="info" emoji="ℹ️">
  This box is an informational callout.
</Callout>

<Steps>
  ### First Step
  Write content.

  ### Second Step
  Write more content.
</Steps>

<Cards>
  <Cards.Card title="Nextra 4" href="https://nextra.site" />
  <Cards.Card title="Pagefind" href="https://pagefind.app" />
</Cards>

<Tabs items={["npm", "pnpm", "yarn"]}>
  <Tabs.Tab>
    ```bash
    npm i nextra nextra-theme-docs
    ```
  </Tabs.Tab>
  <Tabs.Tab>
    ```bash
    pnpm add nextra nextra-theme-docs
    ```
  </Tabs.Tab>
  <Tabs.Tab>
    ```bash
    yarn add nextra nextra-theme-docs
    ```
  </Tabs.Tab>
</Tabs>

<FileTree>
  <FileTree.Folder name="content">
    <FileTree.File name="index.mdx" />
    <FileTree.File name="about.mdx" />
  </FileTree.Folder>
</FileTree>
```

More components: https://nextra.site/docs/built-ins

### 5) Sidebar and `_meta.json`

- Place `_meta.json` in each subfolder under `content/` to control sidebar order/titles/hiding.

Example `content/_meta.json`
```json
{
  "index": {
    "title": "Introduction"
  },
  "about": {
    "title": "About"
  },
  "built-in-components": {
    "title": "Built-ins"
  }
}
```

- Keys are filenames (without extension). Change display name with `title`, hide with `display: false` if needed.

### 6) Document Structure Conventions (Recommended)

- Use lowercase-kebab-case for filenames (`my-feature.mdx`). URLs and sidebar keys remain consistent.
- Divide each section with meaningful H2/H3 to improve automatic TOC quality.
- Split long pages and specify order with `_meta.json` in parent folder.
- Store images/diagrams in `public/images/<domain>/<file>`.
- Always specify language for code blocks.
- Use absolute paths (`/path`) for internal links to increase navigation stability.

---

## Search (Pagefind) Setup and Operation

- `package.json`:

```json
{
  "scripts": {
    "build": "next build",
    "postbuild": "pagefind --site .next/server/app --output-path public/_pagefind"
  }
}
```

- After build, create index from `.next/server/app` as source and output to `public/_pagefind`.
- Index may not be up-to-date during local development, so verify search based on `build` results.

---

## Deployment/Export Tips

- If static Export is needed, consider `output: 'export'` in `next.config.mjs` (according to project requirements).
- Some dynamic features may be limited during static Export.

---

## Common Issue Checklist

- Sidebar order is strange: Check if `_meta.json` configuration is missing in that folder (alphabetical sort if missing)
- Search is empty: Check if `public/_pagefind` was generated after `next build` and `postbuild` ran properly
- Images not visible: Check if path is `/images/...` (based on public)
- MDX component conflicts: Check for overlapping keys in `mdx-components.js`, register with unique names if needed
- TypeScript errors: Check if `next-env.d.ts` exists (auto-generated file)
- Build failure: Check Node.js 18+ and pnpm installation

---

## Quick Writing Example

`content/getting-started.mdx`
```mdx
---
title: Getting Started
description: Introduces project structure and build/search flow.
---

import { Callout } from 'nextra/components'

# Getting Started

This document briefly guides you through the repository structure and core Nextra 4 concepts.

<Callout type="info">All documents are located under content/.</Callout>
```

`content/_meta.json`
```json
{
  "index": { "title": "Home" },
  "getting-started": { "title": "Getting Started" },
  "about": { "title": "About" }
}
```

---

## Version Information and Compatibility

### Current Versions (2024)
```json
{
  "dependencies": {
    "next": "^15.3.4",
    "nextra": "^4.2.17",
    "nextra-theme-docs": "^4.2.17",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@types/node": "22.13.10",
    "@types/react": "19.0.10",
    "pagefind": "^1.3.0",
    "typescript": "^5.7.3"
  }
}
```

### Requirements
- Node.js 18.0.0 or higher
- pnpm recommended (npm, yarn also available)
- TypeScript 5.0+ (optional)

## Deployment Guide

### Static Export

Add to `next.config.mjs`:
```js
export default withNextra({
  output: 'export'  // Static HTML output
})
```

Note: Some dynamic features are limited during static export

## TypeScript/JavaScript Mixed Usage

This project uses TypeScript and JavaScript together:

- **TypeScript files**: `app/layout.tsx`, `tsconfig.json` configuration
- **JavaScript files**: `app/[[...mdxPath]]/page.jsx`, `mdx-components.js`
- **Automatic type checking**: Support gradual migration with `strict: false`
- **Path aliases**: Reference project root with `@/*`

## Minimal Nextra 4 Project Structure

Simplest Nextra 4 project structure:
```
my-docs/
├── app/
│   ├── [[...mdxPath]]/
│   │   └── page.jsx
│   └── layout.tsx
├── content/
│   └── index.mdx
├── mdx-components.js
├── next.config.mjs
└── package.json
```

A working documentation site can be built with just these 5 files

## Reference Links

- Nextra Official Documentation: https://nextra.site
- Theme (Docs) Options: https://nextra.site/docs/docs-theme/configuration
- Built-in Components: https://nextra.site/docs/built-ins
- Pagefind: https://pagefind.app
- This Starter Template: https://github.com/phucbm/nextra-docs-starter
- Live Demo: https://nextra-docs-starter.vercel.app

This document is structured to enable you to proceed from repo cloning → document writing → build/search all at once. If you have additional team rules or linting/prettier rules, merge them into this convention section for operation.
