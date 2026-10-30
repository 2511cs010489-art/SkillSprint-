# SkillSprint

SkillSprint is a Vite + React application backed by Supabase Auth, PostgreSQL, Row Level Security, RPCs, and Realtime. Students and mentors use the same shared records. Mentor profiles and courses are discoverable immediately, while the administrator retains moderation and account-suspension controls.

**Live application:** [https://skillsprint-chi.vercel.app/](https://skillsprint-chi.vercel.app/)

## Main features

- College-email authentication with Supabase Auth
- Student, mentor, and single-administrator workspaces
- Live mentor directory with course and skill filtering
- Mentor course publishing with structured dropdown selections
- Student mentorship requests and mentor acceptance workflow
- Individual task assignment, submissions, reviews, and portfolio proof
- Shared real-time student–mentor chat for every accepted sprint
- Administrator moderation, account suspension, reports, and disputes
- PostgreSQL Row Level Security and server-authorized RPC operations

## Technology stack

- React 19 and TypeScript
- Vite 6 and Tailwind CSS 4
- Supabase Auth, PostgreSQL, Realtime, RLS, and RPCs
- Vercel deployment
- Node.js and Express static-server option
- MySQL Workbench-compatible reference schema

## Supabase setup

Apply the migrations in order. They are additive and preserve existing production records:

1. `supabase/migrations/001_skillsprint.sql`
2. `supabase/migrations/002_security_requests_and_mentor_visibility.sql`
3. `supabase/migrations/003_fix_course_policy_recursion.sql`
4. `supabase/migrations/004_shared_realtime_backend.sql`
5. `supabase/migrations/005_repair_published_course_visibility.sql`
6. `supabase/migrations/006_single_admin_enforcement.sql`
7. `supabase/migrations/007_live_mentor_directory.sql`

Configure the one administrator account and the primary college domain in Supabase SQL Editor:

```sql
update public.security_config
set admin_email = '2808bshreshta@gmail.com',
    approved_college_email_domain = 'mallareddyuniversity.ac.in',
    updated_at = now()
where singleton;
```

The auth trigger accepts this exact administrator address, the configured college domain, and educational addresses ending in `.ac.in`. In **Authentication > URL Configuration**, set the deployed application URL as the Site URL and add it to Redirect URLs so confirmation links return to the live app.

Never put the Supabase service-role key in this frontend. The browser only uses the public anon key; RLS and security-definer RPCs enforce authorization on the server.

## Environment variables

Copy `.env.example` to `.env.local` for local development. Configure the same four values in Vercel or Railway:

```text
VITE_SUPABASE_URL
VITE_SUPABASE_ANON_KEY
VITE_ADMIN_EMAIL
VITE_APPROVED_COLLEGE_EMAIL_DOMAIN
```

Vercel uses `vercel.json` and publishes `dist`. Railway uses `railway.json`, starts the Express static server with `npm start`, and checks `/health`.

## Shared data behavior

- Mentor course creation is an atomic database RPC. It creates or reuses a skill, publishes the mentor-skill offering, and makes the course available to students immediately.
- The mentor directory is loaded through a database-authorized RPC that exposes only public mentor fields. It lists active mentors across accounts and supports filtering by their published course or skill.
- Administrators can still reject courses, suspend accounts, or remove mentor availability after publication.
- Student requests, mentor decisions, tasks, submissions, reviews, messages, portfolio items, and reports are awaited database mutations. The UI refetches only after success and shows the database error on failure.
- Realtime subscriptions cover `courses`, `mentor_skills`, `mentor_requests`, `tasks`, `submissions`, `reviews`, and `messages`. Student/mentor request and task subscriptions use user filters; RLS limits every payload. Subscriptions are removed on logout/unmount.
- A debounced workspace refetch follows each Realtime event. Explicit refetches also run after mutations so the initiating browser updates immediately.
- `localStorage` is used only for private UI conveniences: the activity list and target-skill selection. It is not a source of truth for shared records.

## Local run and validation

Node.js 20 or newer is recommended.

```bash
npm install
npm run dev
```

Open `http://localhost:3000`.

Run the complete local validation before deployment:

```bash
npm run lint
npm test
npm run build
```

## MySQL Workbench copy

Production continues to use Supabase PostgreSQL because SkillSprint depends on Supabase Auth, Row Level Security, RPCs, and Realtime. For local Workbench inspection and SQL practice, run `mysql/skillsprint_mysql.sql`. It creates a separate `skillsprint` schema with matching application entities and the same one-admin rule. It is not a live replacement or automatic mirror of Supabase production data.
