OAT Engine
A spaced repetition study app built as a React artifact for Claude.ai. Designed for ADHD learners with fractured schedules — opens fast, saves everything, gets out of the way.
What it does
Four-phase learning progression: Worked examples → faded completion → stepwise retrieval → full problems. Phase-gated so you can't skip ahead before you're ready.
Spaced repetition scheduling: Performance-adaptive intervals. Correct answers push items further out; errors bring them back sooner.
ADHD-calibrated difficulty: Rolling accuracy and response time tracking. Drops difficulty after 2 consecutive errors. Warm-up problems at session start. Sessions end on a success when possible.
Interleaving: Mixes problem types within sessions with a topic cooldown to force discrimination practice.
Weekly heatmap: Shows study density over time instead of streak-based motivation. Irregular schedules look like patterns, not failures.
Content-agnostic: Load any subject — chemistry, trig, anatomy, contracts. Subject filter buttons appear dynamically based on loaded content.
AI explanations: "Explain this" button sends the problem to Claude's API for a personalized breakdown of what you got wrong.
Offline-capable persistent storage: Progress, content, and session history save locally. Export backups as JSON.
How to run it
This is a React component designed to run as a Claude.ai artifact. To use it:
Open a conversation on claude.ai
Upload oat-engine.jsx and ask Claude to run it as an artifact
The app loads with a built-in starter pack (SN1/SN2/E1/E2 + general chemistry)
Loading custom content
The app includes a Content Manager where you can paste JSON problem arrays. See oat-engine-content-guide.md for the full schema reference and examples.
Quick version: each problem needs id, subject, topic, subtopic, conceptId, phase (1–4), difficulty (1–5), problem_text, steps, answer, and explanation. Paste a JSON array into Content > Add Content > Validate > Load.
You can also give the content guide to any LLM and ask it to generate problem sets for you.
File structure
oat-engine.jsx              # The app (single-file React component)
oat-engine-content-guide.md # Schema reference + prompt template for generating content
README.md                   # This file
Design references
The learning mechanics are based on published research:
Spacing: Rohrer & Taylor (2006), Cepeda et al. (2008) — performance-adaptive intervals, not fixed 1-3-7-14
Interleaving: Taylor & Rohrer (2010), Eglington & Kang (2017) — blocking first, then interleaving for discrimination
Retrieval practice: Van den Broek et al. (2025) — stepwise retrieval within worked examples, not flashcard-style recall
Phase progression: Kalyuga (2003) expertise reversal effect, Renkl & Atkinson (2003) faded worked examples
ADHD calibration: Seymour et al. (2019) frustration tolerance, ARTS response-time-based sequencing
Session design: Alias & Razak (2025) microlearning meta-analysis — 8–15 min optimal range
Status
Active development. Built for personal daily use studying for the OAT (Optometry Admission Test), but the engine is subject-agnostic.
