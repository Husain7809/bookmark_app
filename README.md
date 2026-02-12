# Smart Bookmark App

Production-style bookmark manager built for interview assignment.

## Live Scope
- Google OAuth login (Supabase Auth)
- Protected dashboard
- Add bookmark (`title` + `url`)
- Delete bookmark
- Private user data with Supabase RLS
- Real-time updates across tabs
- Responsive UI
- Dark/Light theme (persisted in `localStorage`)
- Form validation with Zod + React Hook Form

## Tech Stack
- Next.js (App Router) + TypeScript
- Tailwind CSS v4
- Supabase (`@supabase/supabase-js`, `@supabase/ssr`)
- React Hook Form + Zod
- Sonner (toast notifications)

## Project Structure
```txt
app/
  auth/callback/route.ts
  dashboard/
  login/
components/
  auth/
  dashboard/
  providers/
  theme/
hooks/
lib/
  supabase/
  validators/
services/
supabase/
  migrations/
  schema.sql
types/
```

## Environment Variables
Create `.env.local` (or `.env`) with:

```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

## Setup
```bash
npm install
npm run dev
```

## Supabase Database Setup
This project uses SQL migrations in `supabase/migrations`.

### Option A: Dashboard SQL Editor
Run:
- `supabase/migrations/20260212172837_create_bookmarks_table.sql` (or latest migration)

### Option B: Supabase CLI
```bash
npx -y supabase login
npm run db:link -- --project-ref <your-project-ref>
npm run db:push
```

## NPM Scripts
```bash
npm run dev
npm run build
npm run start
npm run lint
npm run db:status
npm run db:link -- --project-ref <project-ref>
npm run db:push
npm run db:new -- <migration_name>
```

## Security Notes
- RLS enabled on `public.bookmarks`
- Policies restrict `select`, `insert`, `delete` to `auth.uid() = user_id`
- No service role key exposed in frontend

## Real-time Behavior
- Supabase Realtime subscription listens on `public.bookmarks`
- Includes tab-sync fallback using `BroadcastChannel`
- Add/delete actions update current tab and other tabs without refresh

## Problems Faced and How I Solved Them

1. **Supabase client error: missing URL/API key**
- Problem: Browser client showed missing env vars even though values existed.
- Cause: Dynamic env access (`process.env[key]`) can fail for client bundling.
- Fix: Switched to direct access:
  - `process.env.NEXT_PUBLIC_SUPABASE_URL`
  - `process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY`

2. **Google login error: `Unsupported provider`**
- Problem: OAuth flow returned provider-not-enabled error.
- Cause: Google provider disabled in Supabase Auth settings.
- Fix: Enabled Google provider and configured OAuth redirect URLs.

3. **`PGRST205` table not found in schema cache**
- Problem: App queries failed for `public.bookmarks`.
- Cause: SQL file existed locally but migration was not applied to project DB.
- Fix: Added proper Supabase migration and applied it with SQL Editor / `db push`.

4. **Realtime not updating reliably across tabs**
- Problem: Multi-tab update could fail when Realtime config/state was inconsistent.
- Fix: Kept Supabase Realtime and added `BroadcastChannel` fallback for same-browser tab sync.

5. **UI/UX polish for assignment quality**
- Problem: Basic starter UI looked generic.
- Fix: Upgraded login/dashboard design, icon-based actions, profile header, theme toggle, and mobile responsiveness.

## Interview Notes
- Focused on separation of concerns:
  - `services/` for data/API calls
  - `hooks/` for state/realtime orchestration
  - `components/` for UI
- Prioritized production constraints:
  - auth guard
  - RLS-based privacy
  - typed validation
  - clear error and loading states

## Future Improvements
- Edit bookmark
- Tagging and sorting
- Pagination for large bookmark lists
- E2E tests (Playwright)
- CI checks (lint + typecheck + tests)
