# CLAUDE.md - AI Assistant Guide for Novel

## Project Overview

**Novel** is an open-source Notion-style WYSIWYG editor with AI-powered autocompletions, built on Tiptap (ProseMirror-based).

### Target Users
- Developers building rich text editors in React applications
- Content creators needing AI-assisted writing tools
- Teams requiring embeddable, customizable editors

### Problem Solved
Provides a drop-in, highly customizable rich text editor with built-in AI features like text continuation, improvement, and grammar fixing, eliminating the need to build editor infrastructure from scratch.

## Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| **Core** | | |
| Next.js | 15.1.4 | Web framework (demo app) |
| React | 18.2.0 | UI library |
| TypeScript | ~5.4-5.7 | Type safety |
| Tiptap | 2.11.2 | Rich text editor (ProseMirror) |
| **Build** | | |
| pnpm | 9.5.0 | Package manager |
| Turborepo | 2.3.3 | Monorepo build |
| tsup | 8.3.5 | Library bundler |
| **AI** | | |
| Vercel AI SDK | 3.0.12 | AI streaming |
| @ai-sdk/openai | 1.1.0 | OpenAI provider |
| OpenAI | 4.28.4 | OpenAI client |
| **Styling** | | |
| TailwindCSS | 3.4.1 | Utility CSS |
| Radix UI | various | Accessible components |
| **Services** | | |
| Vercel Blob | 0.22.1 | Image uploads |
| Vercel KV | 1.0.1 | Rate limiting |
| Upstash | 1.0.1 | Rate limiting |
| **Quality** | | |
| Biome | 1.9.4 | Linting/formatting |
| Husky | 9.1.7 | Git hooks |
| Commitlint | 19.6.1 | Commit standards |
| Changesets | 2.27.11 | Versioning |

## Project Structure

```
novel/
├── apps/
│   └── web/                    # Next.js demo application
│       ├── app/                # App router pages
│       │   ├── api/           # API routes (generate, upload)
│       │   ├── layout.tsx     # Root layout
│       │   └── page.tsx       # Home page
│       ├── components/        # UI components
│       │   └── tailwind/      # Styled editor components
│       │       ├── selectors/ # Text/link/color selectors
│       │       ├── generative/# AI features UI
│       │       └── ui/        # Base UI components
│       ├── hooks/             # Custom React hooks
│       ├── lib/               # Utilities
│       └── styles/            # CSS and fonts
├── packages/
│   ├── headless/              # Core editor package (npm: novel)
│   │   └── src/
│   │       ├── components/    # EditorRoot, EditorContent, EditorBubble
│   │       ├── extensions/    # Tiptap extensions (AI highlight, slash command, etc.)
│   │       ├── plugins/       # Image upload plugins
│   │       └── utils/         # Jotai store, atoms, helpers
│   └── tsconfig/              # Shared TypeScript configs
├── .changeset/                # Changeset configs
├── .github/                   # GitHub Actions (release workflow)
├── .husky/                    # Git hooks
├── turbo.json                 # Turborepo config
├── pnpm-workspace.yaml        # Workspace definition
└── biome.json                 # Biome linting config
```

## Key Files

| File | Purpose |
|------|---------|
| `packages/headless/src/index.ts` | Main library exports |
| `packages/headless/src/components/editor.tsx` | Core EditorRoot/EditorContent |
| `packages/headless/src/extensions/slash-command.tsx` | Slash command implementation |
| `apps/web/app/api/generate/route.ts` | AI text generation endpoint |
| `apps/web/app/api/upload/route.ts` | Image upload endpoint |
| `apps/web/components/tailwind/advanced-editor.tsx` | Full-featured editor example |
| `turbo.json` | Build orchestration |

## Development Setup

### Prerequisites
- Node.js 18+
- pnpm 9.5.0+

### Quick Start

```bash
# Install dependencies
pnpm install

# Run development server
pnpm dev

# Build all packages
pnpm build

# Run linting
pnpm lint

# Format code
pnpm format:fix
```

### Environment Variables

Create `apps/web/.env` from `.env.example`:

```env
# Required
OPENAI_API_KEY=sk-...           # OpenAI API key

# Optional
OPENAI_BASE_URL=                # Custom OpenAI endpoint
BLOB_READ_WRITE_TOKEN=          # Vercel Blob for images
KV_REST_API_URL=                # Vercel KV for rate limiting
KV_REST_API_TOKEN=              # Vercel KV token
```

## Common Commands

```bash
# Development
pnpm dev                        # Start dev servers
pnpm build                      # Build all packages
pnpm typecheck                  # Run TypeScript checks
pnpm clean                      # Clean build artifacts

# Code Quality
pnpm lint                       # Run Biome linter
pnpm lint:fix                   # Fix linting issues
pnpm format                     # Check formatting
pnpm format:fix                 # Fix formatting

# Publishing
pnpm changeset                  # Create changeset
pnpm version:packages           # Update versions
pnpm publish:packages           # Publish to npm
```

## Architecture Notes for AI Assistants

### Data Flow
1. **User Input** -> EditorContent (Tiptap EditorProvider)
2. **Editor State** -> Managed by Tiptap/ProseMirror
3. **Global State** -> Jotai atoms (queryAtom, rangeAtom)
4. **AI Requests** -> POST to /api/generate -> OpenAI -> Stream response
5. **Image Uploads** -> POST to /api/upload -> Vercel Blob

### Key Abstractions

1. **EditorRoot/EditorContent**: Wrapper components providing Jotai Provider and Tiptap context
2. **EditorBubble**: Floating toolbar that appears on text selection
3. **EditorCommand**: Slash command popup using cmdk
4. **Extensions**: Tiptap extensions for features (AIHighlight, CustomKeymap, Mathematics, etc.)
5. **Plugins**: ProseMirror plugins (UploadImagesPlugin)

### State Management
- Uses **Jotai** for lightweight atom-based state
- Store in `packages/headless/src/utils/store.ts`
- Key atoms: `queryAtom` (AI query), `rangeAtom` (selection range)

### Component Patterns
```typescript
// Compound component pattern
<EditorRoot>
  <EditorContent extensions={[...]} initialContent={...}>
    <EditorBubble>
      <EditorBubbleItem />
    </EditorBubble>
    <EditorCommand>
      <EditorCommandItem />
    </EditorCommand>
  </EditorContent>
</EditorRoot>
```

## Dependency Issues & Solutions

### Critical Issues

#### 1. React Version Mismatch with Next.js 15
**Problem**: Next.js 15.1.4 is designed for React 19, but project uses React 18.2.0
**Impact**: May cause hydration issues, missing features, or future incompatibilities
**Solution**:
```bash
# Option A: Downgrade Next.js
pnpm --filter novel-next-app add next@14.2.x

# Option B: Upgrade React (when stable)
pnpm --filter novel-next-app add react@19 react-dom@19
pnpm --filter novel add react@19 --peer
```

#### 2. No Test Suite
**Problem**: Zero test files found in entire codebase
**Impact**: No regression protection, risky refactoring, unclear expected behavior
**Solution**: Add testing infrastructure
```bash
pnpm add -D -w vitest @testing-library/react @testing-library/jest-dom jsdom
```

### High Priority Issues

#### 3. Inconsistent @types/node Versions
**Problem**: web uses 20.11.24, headless uses 22.10.6
**Solution**: Standardize in root package.json
```json
{
  "pnpm": {
    "overrides": {
      "@types/node": "^20.11.24"
    }
  }
}
```

#### 4. Duplicate Biome Versions
**Problem**: Root has ^1.9.4, web has ^1.7.2
**Solution**: Remove from apps/web/package.json, use workspace root version

#### 5. Multiple AI SDK Dependencies
**Problem**: Three AI packages (@ai-sdk/openai, ai, openai) may confuse contributors
**Impact**: Unclear which to use, potential bundling bloat
**Solution**: Document that `@ai-sdk/openai` + `ai` is the modern stack; `openai` can be removed

### Medium Priority Issues

#### 6. Missing Peer Dependencies Declaration
**Problem**: `novel` package has `react` as peer dep but doesn't declare `react-dom`
**Solution**: Add to packages/headless/package.json:
```json
{
  "peerDependencies": {
    "react": ">=18",
    "react-dom": ">=18"
  }
}
```

#### 7. Tiptap Extensions Version Lock
**Problem**: All 15+ Tiptap extensions pinned to ^2.11.2 individually
**Solution**: Consider using @tiptap/extensions meta-package or automate version sync

## Code Quality Issues

### Security Vulnerabilities

1. **Unprotected API Endpoint**
   - Location: `apps/web/app/api/generate/route.ts`
   - Issue: Rate limiting only active with KV configured; without it, unlimited requests
   - Fix: Implement fallback rate limiting or make KV required

2. **Missing Input Validation**
   - Location: `apps/web/app/api/generate/route.ts`
   - Issue: `prompt`, `option`, `command` parsed without validation
   - Fix: Add Zod schema validation
   ```typescript
   import { z } from 'zod';
   const schema = z.object({
     prompt: z.string().max(10000),
     option: z.enum(['continue', 'improve', 'shorter', 'longer', 'fix', 'zap']),
     command: z.string().optional()
   });
   ```

3. **No CORS Configuration**
   - Issue: API routes accessible from any origin
   - Fix: Add appropriate CORS headers in production

### Performance Bottlenecks

1. **Large Bundle Size Risk**
   - Issue: Importing entire Tiptap extensions individually
   - Solution: Tree-shaking analysis, lazy loading non-critical extensions

2. **No Image Optimization**
   - Issue: Uploaded images served without optimization
   - Solution: Use Next.js Image optimization or image processing service

### Anti-patterns

1. **Typo in Production Code**
   - Location: `apps/web/app/api/generate/route.ts:107`
   - Issue: `"You area an AI"` should be `"You are an AI"`

2. **Hardcoded Model**
   - Issue: `gpt-4o-mini` hardcoded in generate route
   - Solution: Make configurable via environment variable

3. **Missing Error Boundaries**
   - Issue: No React error boundaries in editor components
   - Solution: Wrap EditorRoot with error boundary

### Missing Error Handling

1. **API Route Errors**
   - Issue: OpenAI errors not properly caught and formatted
   - Solution: Add try-catch with proper error responses

2. **Image Upload Failures**
   - Issue: No graceful degradation for upload failures

## Testing Analysis

### Current State
**No tests exist in the codebase.**

### Missing Test Coverage
- Unit tests for utilities (getUrlFromString, isValidUrl, etc.)
- Component tests for Editor components
- Integration tests for AI features
- E2E tests for full editor workflow
- API route tests

### Recommended Test Structure
```
packages/headless/
├── src/
│   └── __tests__/
│       ├── utils.test.ts
│       ├── extensions.test.ts
│       └── components.test.tsx
apps/web/
├── __tests__/
│   ├── api/
│   │   └── generate.test.ts
│   └── e2e/
│       └── editor.spec.ts
```

## Prioritized Improvements

### Critical (Do First)
| Issue | Effort | Impact |
|-------|--------|--------|
| Add basic test suite | High | Prevents regressions |
| Fix React/Next.js version mismatch | Low | Stability |
| Add input validation to API routes | Medium | Security |

### High Priority
| Issue | Effort | Impact |
|-------|--------|--------|
| Implement fallback rate limiting | Medium | Security |
| Standardize dependency versions | Low | Maintainability |
| Add error boundaries | Low | Reliability |
| Fix typo in AI prompt | Trivial | Quality |

### Medium Priority
| Issue | Effort | Impact |
|-------|--------|--------|
| Make AI model configurable | Low | Flexibility |
| Add Zod validation | Medium | Type safety |
| Document AI SDK usage | Low | DX |
| Add CI/CD test workflow | Medium | Quality |

### Low Priority
| Issue | Effort | Impact |
|-------|--------|--------|
| Bundle size optimization | High | Performance |
| Image optimization pipeline | Medium | Performance |
| Add more comprehensive TypeScript strict mode | Medium | Type safety |
| Add Storybook for components | High | Documentation |

## CI/CD

### Current Setup
- **Release Workflow**: `.github/workflows/release.yaml` - Changesets-based publishing
- **No Testing Pipeline**: Tests not run in CI

### Recommended Additions
```yaml
# .github/workflows/ci.yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
      - run: pnpm install
      - run: pnpm lint
      - run: pnpm typecheck
      - run: pnpm test  # When tests exist
      - run: pnpm build
```

## API Routes

### /api/generate (POST)
Generates AI text completions.

**Request Body:**
```typescript
{
  prompt: string;      // Text context
  option: 'continue' | 'improve' | 'shorter' | 'longer' | 'fix' | 'zap';
  command?: string;    // For 'zap' option only
}
```

**Response**: Streaming text/event-stream

### /api/upload (POST)
Uploads images to Vercel Blob.

**Request**: FormData with image file
**Response**: `{ url: string }`

## External Resources

- **Demo**: https://novel.sh/
- **Documentation**: https://novel.sh/docs/introduction
- **GitHub**: https://github.com/steven-tey/novel
- **npm Package**: https://www.npmjs.com/package/novel
- **Svelte Port**: https://novel.sh/svelte
- **Vue Port**: https://novel.sh/vue
- **VSCode Extension**: https://novel.sh/vscode

## When Making Changes

1. **Follow existing patterns** - Check similar components in the same directory
2. **Use Biome** - Run `pnpm lint:fix && pnpm format:fix` before committing
3. **Create changesets** - Run `pnpm changeset` for any package changes
4. **Test locally** - Run `pnpm dev` and test in browser
5. **Type check** - Run `pnpm typecheck` before pushing
6. **Commit conventions** - Use conventional commits (enforced by commitlint)

## Commit Message Format
```
type(scope): description

# Types: build, chore, ci, clean, doc, feat, fix, perf, ref, revert, style, test
# Example: feat(headless): add markdown export extension
```

## Common Issues

| Issue | Solution |
|-------|----------|
| `pnpm install` fails | Ensure pnpm 9.5.0+, delete node_modules and pnpm-lock.yaml |
| Build fails | Run `pnpm typecheck` first to catch type errors |
| AI not working | Check OPENAI_API_KEY in .env |
| Images not uploading | Check BLOB_READ_WRITE_TOKEN in .env |
| Rate limited | Configure KV_REST_API_URL and TOKEN or wait |
| Turbo cache issues | Run `pnpm clean` then rebuild |
