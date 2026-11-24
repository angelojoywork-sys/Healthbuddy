# Healthbuddy — instructions for Claude sessions

Healthbuddy is a personal AI fitness coach mobile app (iOS + Android): workouts, nutrition, water, weight, muscle recovery heatmap, reminders, an AI coach with tools, and sharing with friends. The owner is a beginner coder, so explain steps plainly and give exact commands to run on their phone/computer.

**Source of truth:** `docs/PLAN.md` (master plan: stack, data model, screens, phases).

## Stack (do not change without the owner's approval)
- Expo (React Native) + TypeScript + Expo Router
- Supabase (Postgres, Auth, Storage, Edge Functions, Cron)
- Claude API, called ONLY from Supabase Edge Functions. Never put API keys in the app.
- expo-notifications, expo-camera (barcodes), Open Food Facts, free-exercise-db, react-native-body-highlighter, victory-native, Zustand + React Query

## Two kinds of sessions

### Plan session (the owner says "plan phase N" or similar)
1. Read `docs/PLAN.md` and `docs/PROGRESS.md`.
2. Write a detailed, step-by-step plan to `docs/plans/phase-N-<short-name>.md`: files to create, database migrations, screens, libraries, how to test on the phone, and a "Done when" checklist.
3. Do NOT write app code. Commit and push only the plan file.

### Build session (the owner says "implement phase N" or similar)
1. Read `docs/plans/phase-N-*.md` and `docs/PROGRESS.md`.
2. Implement exactly that plan. If something must deviate, note it in PROGRESS.md.
3. Run the type check and tests before committing.
4. Update `docs/PROGRESS.md` (what's done, what's left, any setup steps the owner must do).
5. Commit and push. Give the owner the exact steps to try it on their phone (`npx expo start` → scan with Expo Go).

## Conventions
- App code goes in `app/` (routes), `src/` (components, lib, stores); Supabase code goes in `supabase/` (migrations, functions).
- Secrets go in `.env` (git-ignored) or Supabase secrets. Never commit them.
- Keep commits small and descriptive.

## Working process (portfolio case study)
Follow the working process in @taste.md for all work on this product. Every project becomes a portfolio case study, so:

- Don't jump to UI. Check which process step we are on (research, synthesis, journey map, flow, sketch, wireframe, hi-fi, deploy, measure) and work on that step.
- Keep flows and solutions simple (happy path first).
- Document decisions and learnings as we go in `process-log.md` (see `taste.md`).
- Only add to or remove from `taste.md` when the user asks.

## How to talk to the user
- Educate: teach the why in one line, then give the most efficient, effective way to do it.
- Minimal words. Short answers, no walls of text. Recommend one option, don't survey.
