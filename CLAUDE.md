# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Quartz v4 repository - a static site generator for publishing digital gardens and notes as websites. Quartz transforms Markdown content into a fully-featured website with features like backlinks, graph view, full-text search, and more.

## Build Commands

### Development
```bash
npx quartz build --serve
```
Builds the site and starts a local development server at `http://localhost:8080` with hot-reload enabled.

### Production Build
```bash
npx quartz build
```
Builds the static site to the `public/` directory.

### Other Commands
- `npm run check` - Run TypeScript type checking and Prettier formatting check
- `npm run format` - Format all files with Prettier
- `npm test` - Run tests using tsx

### Build Flags
- `-d` or `--directory`: content folder (default: `content`)
- `-v` or `--verbose`: extra logging
- `-o` or `--output`: output folder (default: `public`)
- `--serve`: run local preview server
- `--port`: local server port
- `--concurrency`: number of threads for parsing

## Architecture

Quartz uses a plugin-based architecture with three types of plugins:

### Plugin Pipeline
1. **Transformers**: Map over content (parse frontmatter, generate descriptions, etc.)
2. **Filters**: Filter content (remove drafts, explicit publish, etc.)
3. **Emitters**: Reduce over content (generate RSS, tag pages, content index, etc.)

### Build Process
- Entry point: `quartz/bootstrap-cli.mjs` (defined in package.json bin)
- Main build logic: `quartz/build.ts`
- Uses esbuild for TypeScript transpilation and bundling
- Supports worker threads for parallel processing (>128 files)
- Uses unified/remark/rehype for Markdown parsing and transformation

### Content Processing Flow
1. Clean output directory
2. Glob all files from content folder
3. Parse Markdown files (text → mdast → hast)
4. Apply transformer plugins at each stage
5. Filter content using filter plugins
6. Emit files using emitter plugins
7. Generate static HTML using Preact SSR

## Configuration Files

### `quartz.config.ts`
Main configuration file containing:
- Site metadata (title, base URL, analytics)
- Theme configuration (fonts, colors)
- Plugin configuration (transformers, filters, emitters)
- Ignore patterns for content

### `quartz.layout.ts`
Defines the page layout structure:
- `sharedPageComponents`: Components shared across all pages (head, header, footer)
- `defaultContentPageLayout`: Layout for single content pages
- `defaultListPageLayout`: Layout for list pages (tags, folders)

## Key Directories

- `content/`: Markdown content files
- `quartz/`: Core Quartz source code
  - `components/`: React/Preact components for UI
  - `plugins/`: Transformer, filter, and emitter plugins
  - `processors/`: Parse, filter, and emit logic
  - `util/`: Utility functions
- `public/`: Build output (git-ignored)
- `docs/`: Quartz documentation

## Development Notes

- Uses Preact for SSR (not React)
- CSS processed with Lightning CSS for vendor prefixes
- Inline scripts split into `beforeDOMLoaded` and `afterDOMLoaded`
- Custom `nav` event dispatched for client-side scripts
- Hot reload via WebSocket on port 3001 when using `--serve`
- Supports SPA routing when enabled in config

## Plugin Development

Plugins are defined in `quartz/plugins/`:
- Transformers can modify text, add remark/rehype plugins, and define external resources
- Filters determine which content should be published
- Emitters generate output files and pages

Components are in `quartz/components/` and can include:
- TypeScript/TSX for rendering
- SCSS for styles
- Inline scripts (`.inline.ts`) for client-side behavior
