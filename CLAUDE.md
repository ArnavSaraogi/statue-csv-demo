# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Statue is a template-based static site generator built on SvelteKit, Markdown, Tailwind, and Pagefind. It generates fully static sites from markdown content with zero-config search, reusable components, and a theming system.

## Development Commands

### Building and Running
```bash
npm run dev              # Start dev server at localhost:3000
npm run build            # Production build (prebuild runs generate:exports + orval)
npm run preview          # Preview production build with serve
npm i && npm run build && npm run preview  # Full rebuild cycle
```

### Testing
```bash
npm test                 # Run all tests (vitest + unit tests)
npm run test:watch       # Run tests in watch mode
./test/test-release.sh   # End-to-end release test (packs library, creates fresh SvelteKit app, tests install)
npm run docker:test      # Run hermetic Docker tests
```

### Code Quality
```bash
npm run lint             # Check formatting + linting
npm run lint:check       # ESLint only
npm run format           # Format with Prettier
npm run format:check     # Check formatting without fixing
```

### Template Management
Templates are in `templates/<name>/`. The root `src/routes/` + `content/` represents the default template.

```bash
npm run template:list    # List available templates
npm run template:load <name>     # Load template to root workspace (OVERWRITES src/routes, content, site.config.json)
npm run template:save <name>     # Save root workspace back to template folder
```

**Warning**: `template:load` overwrites root files. Commit changes first!

To restore default template: `git checkout src/routes content site.config.json`

### Component Exports
```bash
npm run generate:exports  # Auto-generates exports in src/lib/index.ts (runs before dev/build)
```

## Architecture

### Core Structure
- **`src/lib/`** - Shared core logic used by all templates
  - `components/` - Reusable Svelte components
  - `cms/` - Content processing (markdown parsing, template variables, caching)
  - `themes/` - CSS theme files with color schemes
  - `index.ts` - Package exports (auto-generated)

- **`src/routes/`** - Default template routes (source of truth)
- **`content/`** - Default template content (markdown files)
- **`templates/`** - Additional templates (blog, developer-portfolio, real-estate, swagger, etc.)

### Content Processing Flow
1. User creates `.md` files in `content/`
2. `src/lib/cms/content-processor.js` scans and parses markdown with mdsvex
3. Template variables (e.g., `{{site.name}}`) are replaced from `site.config.json`
4. SvelteKit routes render content using components
5. Static site generated at build time

### Key Files
- **`content-processor.js`** - Core CMS logic: markdown parsing, template variables, caching, directory scanning
  - Uses mdsvex for markdown → HTML conversion
  - Processes template variables like `{{site.name}}`, `{{contact.email}}`, `{{date.year}}`
  - Caches content in production, refreshes in dev
  - Exports: `getAllContent()`, `getContentByUrl()`, `getContentByDirectory()`, `getSidebarTree()`

- **`svelte.config.js`** - SvelteKit config with adapter-static, mdsvex preprocessing, aliases
- **`vite.config.js`** - Vite config with aliases ($content, $components, $cms), vitest setup
- **`postinstall.js`** - Copies template files to user's project on `npx statue init`
- **`scripts/statue-cli.js`** - CLI entry point for `npx statue` commands

### Template Variables
Defined in `site.config.json`, processed in markdown/metadata:
- `{{site.name}}`, `{{site.url}}`, `{{site.author}}`
- `{{contact.email}}`, `{{contact.phone}}`
- `{{social.twitter}}`, `{{social.github}}`
- `{{date.now}}`, `{{date.year}}`

## Component Development

### Creating Components
1. Create `src/lib/components/YourComponent.svelte`
2. Use TypeScript for props, theme CSS variables
3. Run `npm run generate:exports` (auto-exports in `src/lib/index.ts`)
4. Document in `src/lib/components/COMPONENTS_README.md`
5. Test with all themes (change import in `src/lib/index.css`)

### Component Guidelines
- **Use theme variables**: `var(--color-primary)`, `var(--color-foreground)`, etc.
- **TypeScript props**: `export let title: string;`
- **Responsive**: Use Tailwind utilities
- **Accessible**: Semantic HTML, ARIA labels
- See `ADDING_COMPONENTS.md` for detailed guide

### Available CSS Variables
```css
--color-background, --color-card, --color-border
--color-foreground, --color-muted
--color-primary, --color-secondary, --color-accent
--color-on-primary, --color-on-background
--color-hero-from, --color-hero-via, --color-hero-to
```

## Theme Development

Themes are in `src/lib/themes/`. Each theme uses `@theme {}` block with 13 required variables.

Switch themes by editing `src/lib/index.css`:
```css
@import "statue-ssg/src/lib/themes/black-white.css";
```

See `ADDING_THEMES.md` for creating new themes.

## Template Development

Templates contain their own `src/routes/`, `content/`, `site.config.json`, optionally `static/` and `scripts/`.

Requirements:
- Must have `[...slug]` and `[directory]` routes
- All `+page.server.js` files need `prerender = true`
- Use `$lib` imports (not `statue-ssg`) during development

See `ADDING_TEMPLATES.md` for detailed guide.

## Validation

Before PR, contributions are validated:
```bash
./scripts/validate-contribution.sh component MyComponent.svelte
./scripts/validate-contribution.sh theme my-theme.css
./scripts/validate-contribution.sh template templates/my-template
```

See `VALIDATION_GUIDE.md` for troubleshooting.

## Build Scripts

- **`scripts/generate-exports.js`** - Auto-generates `src/lib/index.ts` from components
- **`scripts/generate-seo-files.js`** - Generates sitemap.xml, robots.txt (runs postbuild)
- **`scripts/run-pagefind.js`** - Builds search index (runs postbuild)
- **`scripts/run-orval.js`** - Generates API types from OpenAPI specs (runs prebuild)
- **`scripts/generate-rss-feed.js`** - Generates RSS feed (runs postbuild)

## CLI Usage

```bash
npx statue init              # Initialize with default template
npx statue init -t blog      # Initialize with specific template
```

## Publishing (Maintainers Only)

```bash
# 1. Update version in package.json
# 2. Test release
./test/test-release.sh

# 3. Publish
npm publish  # prepublishOnly runs generate:exports + build automatically
```

## Important Notes

- **Server-side only**: `content-processor.js` runs server-side (uses Node.js fs, path). Never import in browser code.
- **Caching**: Content is cached in production, refreshed in dev mode.
- **Template variables**: Processed at build time, not runtime.
- **Static output**: Everything is prerendered; no runtime server needed.
- **Development mode**: Auto-reloads content changes. Production uses cache.
