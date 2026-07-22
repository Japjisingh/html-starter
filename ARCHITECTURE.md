# Pariksha — Production Build Plan

*How the mockup becomes a real product. Written for a lean founding team shipping in phases.*

---

## 1. The wedge: don't boil the ocean

Testbook and Oliveboard cover ~every exam. You can't out-breadth them on day one, and you shouldn't try. **Pick one exam vertical and win it before expanding.**

Best beachhead: **SSC (CGL/CHSL/MTS)** or **Banking (IBPS/SBI)**. Reasons — huge, repeat-heavy audience; well-defined syllabus; PYQs are structured and reusable; aspirants take *many* mocks (high engagement per user). Win one vertical's mock-test reputation, then port the engine to the next.

**Your edge is not content — it's the experience.** Adaptive difficulty, a real exam-hall UI, instant doubt-solving, and gamified daily habit. The mockup already demonstrates all four.

---

## 2. Recommended stack (2026, small team)

| Layer | Choice | Why |
|---|---|---|
| **Web** | Next.js (React) + Tailwind | Fast to build; the mockup's UI ports directly; SSR/SEO for free-content top-of-funnel |
| **Mobile** | React Native (Expo) or a PWA first | 80%+ of Indian aspirants are mobile-only; share logic with web. Start PWA, go native once retention proves out |
| **API** | Node/TypeScript (NestJS) or Python (FastAPI) | One language across web + API if Node; FastAPI if the ML team prefers Python |
| **DB** | PostgreSQL (Supabase / Neon) | Relational fits questions, attempts, users, leagues. Managed = no ops |
| **Cache / realtime** | Redis + WebSockets | Live leaderboards, 1v1 battles, timers, rate limits |
| **Search** | Postgres FTS → Meilisearch later | Question bank search, tag filtering |
| **AI** | Claude API (Anthropic) | Doubt-solver, explanation generation, question tagging, adaptive difficulty signals |
| **Auth** | Phone OTP (primary) + Google | India is phone-first; email is secondary |
| **Payments** | Razorpay | UPI, cards, net-banking, EMI — the India default |
| **Infra** | Vercel (web) + Railway/Fly/AWS (API) | Ship in days, scale later |
| **Analytics** | PostHog | Funnels, retention, feature flags, session replay in one |

---

## 3. Data model (core tables)

```
users(id, phone, name, target_exam, exam_date, plan_tier, created_at)
exams(id, name, pattern_json)                       -- sections, marks, timing, negative marking
questions(id, exam_id, section, topic, subtopic,
          difficulty, stem, options_json, answer_idx,
          explanation, source, language, status)     -- status: draft/reviewed/live
tests(id, exam_id, type, title, section, duration_s, is_free)   -- type: full-mock | sectional | pyq | daily
test_questions(test_id, question_id, order)
attempts(id, user_id, test_id, started_at, submitted_at,
         score, accuracy, percentile, time_taken_s)
attempt_answers(attempt_id, question_id, chosen_idx,
                is_correct, time_spent_s, marked_review)
topic_mastery(user_id, topic, elo, attempts, last_seen)   -- powers adaptive + spaced repetition
battles(id, a_user, b_user, question_set_json, a_score, b_score, winner)
streaks(user_id, current, longest, last_active_date)
xp_ledger(user_id, delta, reason, created_at)
```

The **`topic_mastery.elo`** column is the quiet heart of the product — it drives adaptive difficulty, weak-topic surfacing, spaced-repetition scheduling, and the predicted percentile.

---

## 4. The four differentiators — how they actually work

**Adaptive difficulty.** Give each question an ELO-style difficulty rating and each user a per-topic rating. Serve questions near the user's current rating (slightly above for growth). After each answer, update both ratings like a chess match. Cold-start from the question's author-tagged difficulty. No heavy ML needed to launch — a rating system is enough and is explainable.

**Predicted percentile.** For each test, fit a distribution of raw scores from real attempts. A user's percentile = their position in that distribution, adjusted for question difficulty attempted. Until you have volume, seed with a calibrated curve per exam pattern (the mockup uses a transparent formula as a stand-in).

**AI doubt-solver.** On any question, "Ask a doubt" opens a chat seeded with the question, the user's chosen answer, and the official explanation as context. Claude produces a step-by-step worked solution in the user's language. Cache answers per (question, common-misconception) so repeat doubts are instant and cheap. Guardrail: constrain to the question's topic; never invent facts beyond the provided explanation.

**Real exam-hall UI.** The mockup already nails this — timer, question palette with answered/marked/not-visited states, mark-for-review, section navigation, +2/−0.5 scoring. Match each exam's *exact* interface (SSC's is different from IBPS's). This is a genuine moat: aspirants pick the platform whose mock *feels* like the real test.

---

## 5. Content pipeline (the real operational challenge)

Content quality decides trust. Build a workflow, not a heap of PDFs:

1. **Ingest** PYQs and author new questions into a structured CMS (stem, 4 options, answer, explanation, topic, difficulty, source, language).
2. **AI-assist tagging** — Claude proposes topic, subtopic, and difficulty; a human reviewer confirms. Cuts tagging time ~5×.
3. **Two-eyes review** — every question reviewed by a second subject expert before `status = live`. Wrong answer keys are the #1 way edtech platforms lose trust.
4. **Bilingual from day one** — English + Hindi for SSC/Banking; add regional languages per exam. AI drafts the translation, a human verifies.
5. **Version & retire** — flag questions that skew too easy/hard from real attempt data; retire or re-tag.

> On copyright: PYQ *answers/solutions* you author are yours; question *stems* from official exams are facts/short factual items but reproduce them carefully and attribute the source. Original questions modelled on the pattern are the safest and most scalable path.

---

## 6. Monetisation

- **Free tier** — a few full mocks + daily practice + battles. This is the funnel; keep it genuinely useful.
- **Pass (₹/exam or ₹/year)** — unlimited mocks, all PYQs, detailed analytics, predicted rank, doubt-solver.
- **Super/AIO pass** — all exams bundled (the Testbook playbook).
- Battles, streaks and leagues drive daily active use → conversion. Sell the *analytics and rank prediction*, not just question count.

---

## 7. Phased roadmap

**Phase 0 — Prototype (done):** this interactive mockup. Use it for user interviews and to validate the exam-hall feel.

**Phase 1 — MVP (6–10 wks):** one exam (SSC or Banking). Auth (phone OTP), 20–30 real mocks + sectionals, exam-hall test engine, results + basic analytics, Razorpay, free/paid tiers. Ship as a PWA.

**Phase 2 — The hooks (next 6–8 wks):** topic-mastery ELO + adaptive sectionals, streaks/XP/daily plan, AI doubt-solver, predicted percentile from real data.

**Phase 3 — Engagement & scale:** 1v1 battles + leagues, second exam vertical, native app, referral loops, regional languages.

**Phase 4 — Moat:** personalised study plans, spaced-repetition revision, live all-India mocks on a schedule, creator/educator marketplace.

---

## 8. First five things to build

1. The exam-hall **test engine** (the mockup is your spec — port it to React/Next).
2. The **question CMS + review workflow** (nothing ships without clean content).
3. **Phone-OTP auth** + user profile with target exam & date.
4. **Attempts + results + analytics** (score, accuracy, topic breakdown, percentile).
5. **Razorpay** paywall around premium mocks.

Everything engaging — battles, adaptive, doubt-solver — layers on *after* these five are solid.
