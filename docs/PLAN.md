# Healthbuddy — Master Plan

## Context
You want one mobile app (iPhone and Android) that replaces several fitness apps. It should hold your workout routine, workout logs, food and calories, water intake and weekly weigh-ins, with an AI coach built in. You can ask the coach questions, plan with it, and have it adjust your plan. The app should remind you to go to the gym, eat and drink water, and let you share meal plans and workouts with friends. It is for you first, with possible commercialisation later. You said you have little coding experience, so Claude builds it and walks you through running it on your phone.

**Where it differs from existing apps** (Hevy/Strong for lifting, MyFitnessPal/MacroFactor/Cal AI for food, Fitbod for AI workouts): each of those covers one area. Yours combines four things:
1. A **proactive AI coach** that can see *all* your data (lifts, food, water, weight) and acts on it. It reviews your week every Sunday, changes your plan and nudges you when you drift.
2. **One app for everything**: workouts, nutrition, water, weight and recovery.
3. A **muscle heatmap and recovery view**: which muscles you trained this week and which are recovered.
4. **Social sharing**: share or import a friend's routine or meal plan with one link.

## Tech stack (beginner-friendly and ready to commercialise)
| Need | Choice | Why |
|---|---|---|
| App (iOS + Android) | **Expo (React Native) + TypeScript**, Expo Router | One codebase. You test it on your own phone instantly via the Expo Go app, with no Xcode needed at first. |
| Backend, login, database | **Supabase** (Postgres, Auth, Storage, Edge Functions, Cron) | Generous free tier; handles accounts and sharing between friends. |
| AI coach | **Claude API** (`claude-sonnet-5-5` for the coach, `claude-haiku-5-5` for quick food parsing), called only from a Supabase Edge Function | The API key never goes inside the app. Tool use lets the coach read and change your data. Built-in web search covers diet and exercise research. |
| Meal photos | Claude vision through the same Edge Function | Photo → estimated foods, calories and macros, which you confirm or edit |
| Voice food logging | On-device speech-to-text (e.g. `expo-speech-recognition`), then the same text parsing as typed meals | Say "two eggs and toast" → foods you confirm. One log screen with three modes: Photo (incl. barcode), Speak, Type |
| Barcodes | `expo-camera` barcode scanner + **Open Food Facts** API | Free and global |
| Exercise library | **free-exercise-db** (public domain, ~800 exercises with muscle tags and images) | Seeded into the database |
| Muscle heatmap | `react-native-body-highlighter` | Front and back body SVG, coloured by training volume or recovery |
| Charts | `victory-native` | Weight trend, calories and lift progress (estimated 1-rep max) |
| Reminders | `expo-notifications`: local scheduled notifications (gym, meals, water, Sunday weigh-in) plus Expo push for coach nudges | Works offline. Push lets the coach message you. |
| Local state and offline use | Zustand + React Query, with an SQLite cache (`expo-sqlite`) | Logging in the gym works without signal |
| Later | Apple Health / Google Health Connect (steps, weight scale sync), RevenueCat subscriptions, Apple Watch companion app (SwiftUI; rest timer, log sets with the crown, speak a meal, water, complications) | For the commercial version |

Estimated running cost for one user: Supabase free, Claude API roughly $3–10 a month depending on chat use. Store accounts, only needed when publishing: Apple $99 a year, Google $25 once.

## Data model (Supabase tables)
- `profiles`: goals (cut, bulk or maintain), height, sex, age, activity level, calorie and macro targets, water target, reminder times
- `exercises`: library, with primary and secondary muscles, equipment, instructions and image
- `routines` → `routine_days` → `routine_exercises`: the documented plan (sets, reps, rest, progression rule)
- `workout_sessions` → `set_logs`: actual weight, reps and RPE (rating of effort), with timestamps
- `foods` (cache of Open Food Facts and custom foods), `meal_plans` → `meal_plan_items`, `meal_logs`: source = photo, barcode, voice, text or plan
- `water_logs`, `weight_logs`, `body_measurements` (optional photos)
- `coach_messages` (chat history), `coach_memory` (long-term facts such as injuries, preferences and dislikes)
- `friendships`, `shares` (a share token pointing at a routine or meal plan; the friend can import a copy)

Row-level security means each user sees only their own data plus what is explicitly shared with them.

## The AI coach (the core feature)
Edge Function `coach` → Claude with a system prompt containing your profile, goals and `coach_memory`, plus **tools**:
- `get_summary(range)`: training volume per muscle, calories against target, water, weight trend
- `get_workout_history`, `get_meal_logs`, `search_exercises`
- `create_routine` / `update_routine`: writes a documented plan into `routines`, shown as a card you approve
- `create_meal_plan`: hits your calorie and macro targets and avoids foods you dislike
- `log_meal_from_text` ("2 eggs and toast") and `log_from_photo`
- `set_reminder`, `remember_fact`
- Claude's web search tool for diet and exercise research questions

**Proactive part:** a Supabase Cron job runs each Sunday. It asks you to weigh in (a notification). Once you log your weight, it runs a weekly review: adherence, weight trend and lift progress. It suggests calorie or plan changes and sends a push notification with a short summary. A daily evening check runs too: if you are under your protein target or missed a workout, you get a nudge.

Safety: all plan changes need a confirmation tap. A disclaimer says the coach is not medical advice. Calorie targets have sane minimums.

## App screens (bottom tabs)
1. **Today**: today's workout, a calorie and macro ring, a water counter (+250 ml tap), the next meal from the plan with a tick-off, and a weigh-in prompt on Sundays
2. **Train**: the routine, starting a workout (set logger with the previous session's numbers prefilled, a rest timer), history, and the muscle heatmap and recovery view
3. **Eat**: log a meal by photo, barcode, voice, text or plan item; the meal plan; daily totals
4. **Coach**: a chat with the AI that renders inline cards (plan proposals, charts)
5. **Progress**: weight trend chart (7-day average), lift PRs (personal records), body measurements, weekly reports
- **Settings**: goals, reminder schedule, friends, sharing

## Build phases (each one ends with something usable on your phone)
1. **Foundation**: Expo project, Supabase project, login, profile and goals onboarding (calculates calorie and macro targets using the Mifflin-St Jeor formula), tab layout
2. **Workouts**: seed the exercise library, build and edit routines, the live workout logger, history, the muscle heatmap
3. **Nutrition + water + weight**: barcode, voice, text and photo logging, daily totals, water tracker, weekly weigh-in, progress charts
4. **Reminders**: notification schedule screen (gym days and time, meal times, water every N hours in waking hours, Sunday weigh-in)
5. **AI coach**: chat, tools, routine and meal-plan generation, coach memory, web search
6. **Proactive coach**: Sunday review, daily nudges, push notifications
7. **Social**: friends list, share a routine or meal plan via link or QR code, import a copy, a friend activity feed (optional)
8. **Polish and commercial path** (later): Apple Health / Health Connect, offline sync hardening, branding, App Store / Play Store build via EAS, subscriptions

## What you'll need to set up (I'll guide each step)
- A free Supabase account and project
- An Anthropic API key (console.anthropic.com), stored as a Supabase secret
- The Expo Go app on your phone, and a free Expo account
- A GitHub repo to save the code (recommended, since this cloud session's files are temporary)

## Verification at each phase
- Run `npx expo start`, scan the QR code with Expo Go, and test the feature on your actual phone
- Coach: scripted test prompts, e.g. "make me a 4-day upper/lower plan", "I ate a chicken burrito", "should I lower calories?". Check that the right tools are called and the database rows are created
- Notifications: schedule a reminder one minute out and confirm it arrives
- TypeScript type-check plus a few unit tests for the calorie and macro calculations and the progression logic
