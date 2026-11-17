# Python: Coder → Senior Developer (self-study repo)

A structured curriculum for going from "I know Python syntax" to "I can build, test, ship, and maintain production systems."

## How it's organized

- `syllabus.md` is the roadmap: 14 chapters, from syntax revision (Ch 0) to a production-grade capstone (Ch 13). Chapters cover idiomatic code, packaging, testing, tooling, databases, APIs, concurrency, design patterns, DevOps, observability, security, and scale.
- Each `chapters/NN-name/` folder has:
  - `README.md`: the concepts and explanations
  - `practice_problems.md`: exercises
  - `project.md`: that chapter's step-by-step increment to **TaskFlow**, a task tracker you grow across the whole course
- `TaskFlow/` is the running project, and `problem_solving/` has easy, medium and hard drills for extra practice.

## How to study

1. Go in order, and don't skip Ch 0. Gaps there cause trouble later.
2. For each chapter: read the README, then do the practice problems, then do the `project.md` increment in TaskFlow. Write your own answers in the chapter's `.py` file, and keep a notes file if you like.
3. Check yourself against each chapter's "You'll Know You're Ready When…" line before moving on.
4. Budget roughly 12–20 weeks part-time. Spend the most time on Ch 3–7, which are the biggest step up from hobbyist to professional.

**Approach:** Go chapter by chapter in order: read the README, attempt the practice problems yourself, then build that chapter's TaskFlow increment, and only move on once you can pass the "You'll Know You're Ready When…" check without looking anything up.

**With AI assistants:** Attempt everything yourself first, then use the AI as a tutor and reviewer: ask it to explain concepts, give hints, and critique your finished code, rather than writing the solutions for you.

## Setup

Requires Python 3.12+ and [`uv`](https://docs.astral.sh/uv/):

```bash
uv sync
```
