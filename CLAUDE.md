# core-comments -- CLAUDE.md

Polymorphic threaded comment system that attaches to any content type.

## Module Boundary

**core-comments owns:** Comment schema (`targetType`/`targetId`/`content` fields), comments router with public + admin endpoints, DI deps, public components (CommentSection, CommentForm, CommentItem, CommentCount), styles, seed.

**Project owns:** Admin pages, `config/deps/comments-deps.ts`, integration into page templates (e.g. PostDetail).

## DI (`setCommentsDeps()`)

- `sendNotification` -- notify parent comment author on reply (skips self-replies)
- `onCommentCreated?` -- lifecycle callback (e.g. record activity event)
- `onCommentDeleted?` -- lifecycle callback (e.g. record activity event)

## Schema

`cms_comments` -- polymorphic via `targetType` + `targetId`. Threading via nullable `parentId` (self-reference). Soft-delete via `deletedAt`. Status: 0=pending, 1=approved, 2=rejected, 3=spam. Logged-in users auto-approved. Optional `authorName` display override.

## Router Endpoints

Live list: `routers/comments.ts` (JSDoc per procedure → `src/generated/procedure-docs.ts` / MCP `search_tools`). Non-obvious semantics only: `countMany` caps at 100 IDs; `create` auto-approves logged-in users; public endpoints return approved-only.

## Components

Live list: `components/` (JSDoc'd props; map with `bun run indigo map core-comments`). Non-obvious: `CommentSection` imports the module CSS itself; `CommentItem` renders replies recursively to depth 3; `CommentForm` caps content at 5000 chars.

## Styles

`styles/comments.css` -- all classes prefixed `comment-`. Uses CSS custom properties from design system (`--text-primary`, `--text-muted`, `--border-subtle`, `--surface-primary`, `--surface-secondary`, `--radius-md`, `--color-brand-600`, `--color-error-600`). Imported by `CommentSection`.

## Admin Page

`_templates/app/dashboard/(panel)/comments/page.tsx` -- Moderation queue with status filter tabs (All/Pending/Approved/Rejected/Spam with counts), search, table with author/comment/target/status/date/actions columns. Actions: approve, reject, spam, delete. Uses `ConfirmDialog` for delete confirmation, `toast` for success/error feedback.

## Seed

`seed/index.ts` -- `seedComments()` creates 12 demo comments (8 on blog posts + 4 on showcase items, mix of top-level and threaded replies). `hasCommentsData()` skip check. Deterministic UUIDs (`00000000-0000-4000-c100-*`) for idempotent seeding.

## Key Features

- **Polymorphic** -- attach to any content type via `targetType` string
- **Threaded** -- `parentId` self-reference, flat API (client builds tree up to depth 3)
- **Moderation** -- pending/approved/rejected/spam statuses, admin queue with batch actions
- **Auto-approve** -- logged-in user comments approved immediately
- **Cursor pagination** -- efficient for real-time comment loading
- **Batch counts** -- `countMany` avoids N+1 on listing pages
- **User enrichment** -- joins user table for name/image on public, +email on admin
- **Edit/Delete** -- users can edit/delete their own comments inline

## Integration

To add comments to a page template, import and render `CommentSection`:

```tsx
import { CommentSection } from '@/core-comments/components/CommentSection';

<CommentSection targetType="post" targetId={post.id} />
```

For inline count badges on list pages:

```tsx
import { CommentCount } from '@/core-comments/components/CommentCount';

<CommentCount targetType="post" targetId={post.id} />
```

## Module Config

- Seed: `seedComments` with `hasCommentsData` check
- Nav: Comments item under `content` group (MessageSquare icon)
- Project files: `config/deps/comments-deps.ts`, `app/dashboard/(panel)/comments/page.tsx`
