# Lessons from Using AI · An Engineer's Field Notes

> [简体中文](README.md) | Front-line discipline for getting work done with AI: how to save tokens, how to keep it from lying, how to turn human decisions into machine-run process.
> Updated: 2025-11-25 · Field-tested in daily research/engineering pipelines (Claude CLI, codex, MCP tool stack).

**TL;DR**: Treat AI like a very capable engineer who jumps to conclusions — plan first, lock constraints in, verify artifacts, close honestly; expensive compute on decisions, cheap compute on execution.

---

## Contents
1. [Context Discipline](#1-context-discipline) — the first rule of saving tokens
2. [Double Insurance (Hooks + Memory)](#2-double-insurance-hooks--memory) — make constraints land automatically
3. [Self-Check Loop](#3-self-check-loop) — never trust "done"
4. [Sub-agent Division of Labor](#4-sub-agent-division-of-labor) — main brain + hands
5. [Knowledge Base and Anti-Fabrication](#5-knowledge-base-and-anti-fabrication)
6. [Ask a Human at Blockers, Don't Grind](#6-ask-a-human-at-blockers-dont-grind)
7. [Uncovered Ground](#7-uncovered-ground) — honest closure
8. [Seven Engineering Hardening Rules](#8-seven-engineering-hardening-rules)
9. [Separating Decisions from Execution](#9-separating-decisions-from-execution--token-economics) — Token Economics
10. [Turning the Decision Mechanism into a Product](#10-turning-the-decision-mechanism-into-a-product--two-layer-dispatch)

---

## 1. Context Discipline
**The problem**: context is the AI's working memory, and the easiest place for garbage to take over. Measured it once: reading a results file pushed context usage to 519%, and everything after that was a struggle in noise.

- **Treat the context progress bar as your dashboard**; the moment one tool call returns more than a few thousand characters of raw content, switch to "sub-agent reads it, I only want the conclusion".
- **Treat `clear` as the period at the end of a stage**: only after every artifact of that stage is written to disk and every conclusion goes into persistent memory/docs do you clear. Clear too early = lose work; clear too late = drag garbage around.
- **For logs, locate with grep/head/tail first; read files with offset/limit** — don't read a 3MB log like an essay.

## 2. Double Insurance (Hooks + Memory)
**The principle**: getting constraints to stick is far cheaper than stepping in the same hole again.

- **Plan first**: before touching a non-trivial task, produce an internal plan (step breakdown / which Skills and memory to pull in / the pitfalls this kind of task has bitten us with / closing self-check list). Skip it for light tasks; mandatory for long ones.
- **Hooks enforce hard constraints** (no auto-shutdown, ask before anything goes out, banned-word blocking), **memory carries experience** (how we got it wrong last time, what the agreed definitions are). You need both layers: hooks block "don't do that", memory reminds "do this".
- **⚠️ Hook failures are silent** — a deleted script, a syntax error, a timeout, and the framework raises no error; it just stops reminding you. After changing machines, changing config, or upgrading the CLI, you must verify with a test prompt that it actually fires.

## 3. Self-Check Loop
**The problem**: AI "assumes" — it claims something is done when it never did it, or when it's only a guess.

- **Never trust `*_DONE` / "all done" markers; trust artifacts only**: file exists, non-zero bytes, all fields present, numbers match the criteria. Hard-learned: all three steps of a script failed and it still printed `ALL_DONE`.
- **The three-part artifact check (machine + absolute path + byte count)**: saying "written at X" must come with machine + absolute path + byte count. "I've got it ready" is not written to disk.
- **Self-Q&A must include a falsification round**: across multiple review rounds, at least one round must be "assume the conclusion is wrong, go find counter-evidence"; the criterion must match a string unique to the conclusion (burned before: `grep "构建成立"` hit a field name — always true).
- **Any file transfer must be md5-checked on both sides**; before claiming "all / none / 0 remaining", grep first, and state the search scope.

## 4. Sub-agent Division of Labor
**The pattern**: the main agent is the brain (sets the plan, judges results, writes to disk); sub-agents are the hands (read big files, patrol, run commands).

- Read-only tasks parallelize freely; **parallel write tasks step on each other** — either dispatch disjoint directories, or serialize.
- **A sub-agent's conclusion is not evidence**: it's good at search/read/run, not at settling definitions; spot-check the "done" reports against the three-part artifact check.
- Send thinking work to a "full-capability" sub-agent (a fork that inherits context), mechanical work to a lightweight one; don't downgrade judgment work to save prompts.

## 5. Knowledge Base and Anti-Fabrication
- **The knowledge base is where a long task starts, and the precondition for the AI not making things up**: build/verify the primary sources first, then let it work.
- **Citations must really exist and be checkable item by item**: get hold of the original for every reference before writing it; if you can't verify it, don't write it; second-hand sources are leads only. A hallucinated citation is academic misconduct, not a typo.
- **External state has a TTL**: API availability, software versions, site structure, journal formatting all drift. Mark conclusions that depend on external state with the date you pulled the data, and re-verify before reuse.

## 6. Ask a Human at Blockers, Don't Grind
Login walls, anti-scraping, institutional auth, missing resources — when you can't get through but it's the optimal path, **stop and ask a human** (turn on the proxy, log in, hand over credentials); don't detour down a suboptimal path, and don't ram the wall over and over burning tokens. Same rule: **confirm before irreversible / outbound / money-spending actions**; reversible internal actions you just do.

## 7. Uncovered Ground
When wrapping up, always state clearly: **what ground you covered, what ground you left uncovered**. Never make universal claims like "nothing else / everything done" — universal claims are almost always wrong, and they infect the next round. An honest statement of boundaries is worth more than a bloated "done".

## 8. Seven Engineering Hardening Rules
1. **Skill/prompt size blowup is the next pitfall**: cheat-sheet + pointers (<5KB), split history out and read on demand.
2. **Falsification-round rule**: see §3; 10 rounds without 1 round spent hunting counter-evidence = parroting.
3. **Don't parallelize write tasks** (see §4).
4. **Date-stamp external state** (see §5).
5. **Prove hooks actually fire** (see §2).
6. **Autonomy tiers**: reversible internal actions on your own; privacy / outbound / spending needs confirmation first.
7. **Take a dated snapshot before config changes / bulk deletes**: break it and you can roll back.

---

## 9. Separating Decisions from Execution — Token Economics
**Observation**: token waste comes in two flavors — spent where no thinking was needed (execution taken for a decision), and spent where nothing had been thought through (a decision taken for execution). The first burns money; the second burns money **and** amplifies error: one bad decision makes everything downstream redo.

**The four-question triage** — before starting each step, ask four questions; the more "yes" answers, the more it's worth spending expensive compute on deciding:

| Question | Decision step (strong model, multiple candidates, judge review) | Execution step (cheap model, single pass, machine-checked gate) |
|---|---|---|
| High cost of error / irreversible? | Picking options, setting criteria, writing claims | Filling content, fixing format, batch rewrites |
| Open answer space? | No standard answer; trade-offs to weigh | Spec card in hand; produce to order |
| Result not machine-checkable? | Whether the wording overstates, whether the argument holds | Word count / citations / tests machine-verifiable |
| Output constrains every downstream step? | **Outline, criteria, checklists = most valuable** | Only consumes upstream text, produces no constraints |

**The outline-first pattern** (writing example): ① strong model produces an outline with recommendations (decision materialized) → ② cheap model fills in section by section off the outline (known work) → ③ machine gate (word count / citations / red lines) → ④ strong model reviews and revises judgments. Keep the "thinking" compute at ①④, drop the "writing" compute to ②.

**Empirical note**: the gain distribution we saw in controlled experiments backs up this intuition — prompt contracts (a kind of decision scaffolding) gain mostly on decision-type tasks (research ideation, +17.7 points), and are unstable on by-the-book execution tasks; counter-examples exist too: stuff a long preamble into the execution budget and it squeezes out the deliverable itself (output truncated). **Misplaced compute isn't just waste, it's harm.**

**Failure signals**: an execution step starts freelancing (going past the spec) = bounce it back to the decision step to fill in the spec; a decision step's output packed with execution detail = waste, cut it.

## 10. Turning the Decision Mechanism into a Product · Two-Layer Dispatch
Turn the division of labor above — "humans set the rules, machines execute" — into a one-stop tool users never have to think about:

- **Two-layer API**: users configure their own **decision layer** and **execution layer** keys (BYOK; the tool holds no keys and doesn't resell);
- **Task recognition layer**: when a request comes in, identify the task type first, light up the label, and route automatically by that task's "how to decide, how to execute";
- **Optimization entry**: user states their main line of work; with consent, the tool spends a few tokens running small-scale controlled comparisons, optimizes the routing config, and reports back;
- **Meter**: decision-layer / execution-layer tokens accounted separately; consumption is visible;
- **Honest positioning**: no claim of "automatic best-picking" — we tested it, and open best-picking gets falsified on real task distributions; the tool only does **division of labor and mine-clearing** (excluding known-bad models, each exclusion cited), leaving best-picking to the user and the data.

> The empirical tool behind these notes is [contract-core](https://github.com/Li-start363/contract-core) — a "contract" mechanism that makes the cheap model think before writing, with measured data and an honest failure map.

*Field notes, not doctrine. Open an issue if you have counter-examples.*
