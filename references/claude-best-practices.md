# Claude Code best practices: official guidance and practitioner reports

Reference notes for the agentic-coding book. Scope: Claude Code (CLI, IDE, desktop, cloud) and Claude models used as coding agents, as of 2026-09-30.

Conventions:

- **Official** = Anthropic docs (`code.claude.com`, `platform.claude.com`) and Anthropic engineering/blog posts.
- **Practitioner** = named individuals or company engineering blogs, with author and date. Each claim is tagged **[anecdote]**, **[self-reported figure]**, or **[measured]**.
- Version numbers (e.g. v2.1.283) quoted in the docs are the only date proxies for undated doc pages.
- Model names in the current official docs: Claude Fable 5.1/5, Mythos 5.1/5, Opus 5.5/5/4.8/4.7/4.6, Sonnet 5.5/5/4.6, Haiku 4.5 ([Model configuration](https://code.claude.com/docs/en/model-config)). Model names in practitioner posts (e.g. Opus 4.5, Sonnet 4.5, GPT-5.2-Codex) are reproduced as reported and tied to the post date.

## Source status

- The April 2025 post `anthropic.com/engineering/claude-code-best-practices` returns a 308 redirect to `code.claude.com/docs/en/best-practices`; treat the docs page as the maintained version and the original post as superseded ([redirect](https://www.anthropic.com/engineering/claude-code-best-practices); [Best practices](https://code.claude.com/docs/en/best-practices)).
- Most practitioner sources date from mid-2025 to early 2026; claims about hooks, subagents, and models from that period may not reflect current behavior.

## 1. Instruction files (CLAUDE.md, rules, AGENTS.md, memory)

### Official

- **Content.** CLAUDE.md is read at the start of every conversation. Include: Bash commands Claude can't guess, code style rules that differ from defaults, test instructions and preferred runners, repo etiquette (branch naming, PR conventions), project-specific architectural decisions, environment quirks (required env vars), common gotchas ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Exclude:** anything inferable from code, standard language conventions, detailed API docs (link instead), frequently changing info, long tutorials, file-by-file codebase descriptions, self-evident rules such as "write clean code" ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Pruning test:** for each line, "Would removing this cause Claude to make mistakes?" If not, cut it. "Bloated CLAUDE.md files cause Claude to ignore your actual instructions." ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Size:** target under 200 lines per file; group under headers and bullets; remove contradictions, since Claude "may pick one arbitrarily" ([Memory](https://code.claude.com/docs/en/memory)).
- **Write verifiable instructions:** "Use 2-space indentation" rather than "Format code properly"; "Run `npm test` before committing" rather than "Test your changes"; "API handlers live in `src/api/handlers/`" rather than "Keep files organized" ([Memory](https://code.claude.com/docs/en/memory)).
- **Emphasis:** add "IMPORTANT" to a single line Claude keeps skipping; "If you emphasize many lines, none of them stands out." ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **When to add a line:** Claude makes the same mistake a second time; review catches something Claude should have known; you type the same correction as last session; a new teammate would need the same context. Multi-step procedures go to a skill; part-of-codebase guidance goes to a path-scoped rule ([Memory](https://code.claude.com/docs/en/memory)).
- **Maintenance:** `/init` generates a starter file; `/context` confirms it loaded; check CLAUDE.md into git ([Best practices](https://code.claude.com/docs/en/best-practices)). `/doctor` proposes cuts for derivable content (v2.1.206+). `/doctor prompt-audit` (v2.1.283+) audits CLAUDE.md, CLAUDE.local.md, AGENTS.md, rules, skills, commands, subagents, and output styles for instructions written for older models, references to nonexistent files/commands, and contradictions ([Memory](https://code.claude.com/docs/en/memory)).
- **Diagnostics:** Claude ignores a rule → file is probably too long; Claude asks questions answered in CLAUDE.md → phrasing is ambiguous ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Hierarchy (concatenated, broad → specific, not overriding):** managed policy (`/Library/Application Support/ClaudeCode/CLAUDE.md` macOS; `/etc/claude-code/CLAUDE.md` Linux/WSL; `C:\Program Files\ClaudeCode\CLAUDE.md` Windows; or `claudeMd` key in `managed-settings.json`) → user `~/.claude/CLAUDE.md` → project `./CLAUDE.md` or `./.claude/CLAUDE.md` → local `./CLAUDE.local.md` (gitignored). Ancestor files load at launch root-first; subdirectory CLAUDE.md files load on demand when Claude reads files there. Managed CLAUDE.md cannot be excluded ([Memory](https://code.claude.com/docs/en/memory)).
- **Monorepos:** `claudeMdExcludes` (glob patterns on absolute paths) skips irrelevant ancestor files. Block-level HTML comments are stripped before injection, so maintainer notes cost no tokens ([Memory](https://code.claude.com/docs/en/memory)).
- **Imports:** `@path/to/import`; relative to the importing file; max depth four hops; not expanded inside code spans/fences. Imports "help you organize a long file but don't reduce its context cost" because they load at launch. External imports trigger a one-time approval. `@~/.claude/my-project-instructions.md` shares personal instructions across worktrees ([Memory](https://code.claude.com/docs/en/memory)).
- **Rules:** `.claude/rules/*.md` (recursive) and `~/.claude/rules/`. Without frontmatter they load at launch; with `paths:` glob frontmatter they load only when Claude reads matching files. `paths` is the only frontmatter field read ([Memory](https://code.claude.com/docs/en/memory)).
- **AGENTS.md:** since v2.1.277 Claude Code reads `AGENTS.md` natively when no `CLAUDE.md`/`CLAUDE.local.md` exists in the working directory or above. Setting **Project instructions**: `claude-md-or-agents-md` (default), `claude-md-and-agents-md`, `claude-md`, `managed-only`. To share one file across tools, use a `CLAUDE.md` containing `@AGENTS.md` plus Claude-specific notes; a symlink also works but is not recommended on Windows ([Memory](https://code.claude.com/docs/en/memory)).
- **Delivery and enforcement:** CLAUDE.md is delivered as a user message after the system prompt, with no guarantee of strict compliance. Use `--append-system-prompt` for system-prompt-level instructions and hooks for must-run actions. "CLAUDE.md instructions shape Claude's behavior but are not a hard enforcement layer." Put technical enforcement (`permissions.deny`, `sandbox.enabled`, `env`) in managed settings ([Memory](https://code.claude.com/docs/en/memory)).
- **Auto memory** (on by default): notes written to `~/.claude/projects/<project>/memory/` with a `MEMORY.md` index; first 200 lines or 25KB load each session; types `user`, `feedback`, `project`, `reference`. Toggle via `/memory`, `autoMemoryEnabled`, or `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`. "remember X" writes auto memory; "add this to CLAUDE.md" edits CLAUDE.md ([Memory](https://code.claude.com/docs/en/memory)).

### Practitioner

- HumanLayer (Kyle, 2025-11-25): target <300 lines; HumanLayer's own root file is <60 lines [anecdote] ([HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)).
- HumanLayer: Claude Code's system prompt carries ~50 instructions and frontier models follow ~150–200 instructions "with reasonable consistency", so CLAUDE.md should hold as few as possible. These are the author's estimates with no cited measurement ([HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)).
- HumanLayer: CLAUDE.md is wrapped in a reminder that the context "may or may not be relevant"; instructions without universal applicability are the ones reported as ignored [anecdote] ([HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)).
- HumanLayer: progressive disclosure: put task-specific docs in separate files (e.g. `agent_docs/building_the_project.md`) referenced by path; prefer file pointers over pasted snippets, which go stale. "Never send an LLM to do a linter's job." ([HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)).
- Shrivu Shankar (2025-11-02): monorepo CLAUDE.md of 13 KB (could grow to 25 KB) documenting only tools used by 30%+ of engineers. "Start with guardrails, not a manual": add entries after observed errors [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)).
- Shankar: avoid `@`-imports (they inline the file every session); mention the path and say when to read it. Replace "Never use X" with "Prefer Y". If a tool needs a long explanation, fix the tool ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)).
- Shankar: monorepo baseline setup consumes ~20K tokens (~10% of a 200K window) before work starts [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)).
- Mitchell Hashimoto (2026-02-05): when the agent repeatedly runs wrong commands or finds wrong APIs, add a rule to `AGENTS.md`; for harder cases, build verification tools (scripts, screenshot utilities) the agent calls. Cites Ghostty's AGENTS.md [anecdote] ([Hashimoto](https://mitchellh.com/writing/my-ai-adoption-journey)).
- Armin Ronacher (2025-06-12): CLAUDE.md tells the agent where logs are; important output goes to log files as well as the terminal [anecdote] ([Ronacher](https://lucumr.pocoo.org/2025/6/12/agentic-coding/)).
- Harper Reed (2025-05-08): CLAUDE.md adapted from Jesse Vincent's template with TDD guidance and style preferences [anecdote] ([Reed](https://harper.blog/2025/05/08/basic-claude-code/)).
- Vincent Quigley, Sanity (2025-09-02): project CLAUDE.md files document architecture decisions, codebase patterns, gotchas [anecdote] ([Sanity](https://www.sanity.io/blog/first-attempt-will-be-95-garbage)).
- Sankalp (2025-12-27): moves large instruction sets into skills to keep CLAUDE.md under ~500 lines [anecdote] ([Sankalp](https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/)).

### Conflicts

- **Length targets differ:** official <200 lines per file; HumanLayer <300 (own file <60); Sankalp <500; Shankar 13–25 KB for a shared monorepo file. Common principle across sources: include only content nearly every session needs.
- **`/init`:** official docs recommend `/init` for a starter file ([Best practices](https://code.claude.com/docs/en/best-practices)); HumanLayer recommends against auto-generation because CLAUDE.md is "the highest leverage point of the harness" ([HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)).
- **Imports:** official docs and Shankar agree that `@`-imports load at launch and do not save context; Shankar recommends plain path mentions instead.

## 2. Workflows

### Official

- **Verification first.** "Give Claude a check it can run: tests, a build, a screenshot to compare." Four gating strengths: in-prompt; a `/goal` condition (a separate evaluator re-checks each turn); a Stop hook that blocks turn end until a script passes; a verification subagent or dynamic workflow as a second opinion. Ask for evidence (test output, commands, screenshots) rather than assertions. `/verify` checks against the running app ([Best practices](https://code.claude.com/docs/en/best-practices)).
- Prompt examples: provide test cases and "run the tests after implementing"; "[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them"; "address the root cause, don't suppress the error" ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Explore → Plan → Implement → Commit.** Plan mode via `Shift+Tab` until `⏸ plan mode on`, or `claude --permission-mode plan`; `Ctrl+G` opens the plan in your editor; `/plan` prefix enters plan mode for one prompt; then implement with tests; then "commit with a descriptive message and open a PR" ([Best practices](https://code.claude.com/docs/en/best-practices); [Permission modes](https://code.claude.com/docs/en/permission-modes)).
- Skip planning when "you could describe the diff in one sentence"; plan when the approach is uncertain, multiple files change, or code is unfamiliar ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Test-first bug fixing:** "write a failing test that reproduces the issue, then fix it"; test-writing example: "write a test for foo.py covering the edge case where the user is logged out. avoid mocks." ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Interview/spec pattern:** have Claude interview you with the `AskUserQuestion` tool and write `SPEC.md`, then implement in a fresh session. Specs name files and interfaces, state what is out of scope, and end with an end-to-end verification step ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Rich input:** `@` file references (also pull in CLAUDE.md files from that directory and parents), pasted images (`Ctrl+V`; `Alt+V` on Windows/WSL), URLs, `cat error.log | claude` ([Best practices](https://code.claude.com/docs/en/best-practices); [Common workflows](https://code.claude.com/docs/en/common-workflows)).
- **Course correction:** `Esc` stops mid-action with context kept; `Esc Esc` or `/rewind` restores conversation, code, or both, or summarizes from/up to a message; `/clear` between unrelated tasks. "If you've corrected Claude more than twice on the same issue in one session... Run `/clear` and start fresh with a more specific prompt." ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Checkpoints** are created per prompt and persist across resume, but track only Claude's file-editing tools, not Bash or external processes; "not a replacement for git" ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Sessions:** `/rename` sessions and treat them like branches; `claude --continue`, `claude --resume`, `claude --from-pr <n>` ([Best practices](https://code.claude.com/docs/en/best-practices); [Common workflows](https://code.claude.com/docs/en/common-workflows)).
- **Codebase Q&A** for onboarding: ask what you would ask a senior engineer ("How does logging work?") ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **CLI tools** (`gh`, `aws`, `gcloud`, `sentry-cli`) are "the most context-efficient way to interact with external services"; Claude can learn unknown CLIs via `--help` ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Parallel sessions:** `claude --worktree feature-auth` (requires at least one commit; see `.worktreeinclude`), cross-session messaging, desktop app, cloud sessions, agent view (`claude agents`, research preview), agent teams (experimental, off by default) ([Best practices](https://code.claude.com/docs/en/best-practices); [Common workflows](https://code.claude.com/docs/en/common-workflows)).
- **Writer/Reviewer:** "A fresh context improves code review since Claude won't be biased toward code it just wrote." Variant: one session writes tests, another writes code to pass them ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Adversarial review:** `/code-review` reviews the diff in a fresh subagent; or prompt "Use a subagent to review the rate limiter diff against PLAN.md... Report gaps, not style preferences." Reviewers asked to find gaps report some even for sound work; chasing every finding "leads to over-engineering"; restrict reviewers to correctness and requirement gaps ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Anthropic internal teams** (2025-07-24): Security Engineering uses TDD with Claude and feeds stack traces during incidents; Product Design runs autonomous write–test–iterate loops and reviews the result; Data Infrastructure feeds dashboard screenshots during outages; Product Engineering uses Claude Code as the "first stop" to find files to examine ([How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code); [PDF](https://www-cdn.anthropic.com/58284b19e702b49db9302d5b6f135ad8871e7658.pdf)).

### Practitioner

- **Spec + step plan.** Harper Reed (2025-05-08): idea → `spec.md` (reasoning model) → `prompt_plan.md` of step prompts; master prompt: open `@prompt_plan.md`, implement the next incomplete item, run tests, build, commit, mark done, pause for review. Reports greenfield plans done in 30–45 min [anecdote] ([Reed](https://harper.blog/2025/05/08/basic-claude-code/)).
- **TDD plan file.** Kent Beck (2025-06-25): `plan.md` lists unmarked tests; instruction "find the next unmarked test in plan.md, implement the test, then implement only enough code to make that test pass"; system prompt stresses Red→Green→Refactor and separating structural from behavioral changes. Project: BPlusTree3, ~4 weeks [anecdote] ([Beck](https://newsletter.kentbeck.com/p/augmented-coding-beyond-the-vibes)).
- **Loop ("Ralph").** Geoffrey Huntley (2025-07-14): `while :; do cat PROMPT.md | claude-code ; done` with `@fix_plan.md` (prioritized TODO), `@specs/`, `@AGENT.md` (build/run instructions). Constraint: exactly one item per loop; relaxing it "correlates with degraded outcomes" [anecdote] ([Huntley](https://ghuntley.com/ralph/)).
- **Automated backpressure.** Reed: "The robots LOVE TDD"; linters (Ruff, Biome) and `pre-commit` hooks for tests, types, lint block broken commits [anecdote] ([Reed](https://harper.blog/2025/05/08/basic-claude-code/)). Huntley: type system, unit tests for modified code, static analyzers for dynamic languages [anecdote] ([Huntley](https://ghuntley.com/ralph/)).
- **Tool and language choice.** Ronacher (2025-06-12): agent-called tools must be fast with clear errors; prefers Go for backends; reports agents struggle with Python fixture injection, async, and slow interpreter startup [anecdote] ([Ronacher](https://lucumr.pocoo.org/2025/6/12/agentic-coding/)).
- **Session scope.** Hashimoto (2026-02-05): separate planning and execution sessions; avoid "mega sessions" [anecdote] ([Hashimoto](https://mitchellh.com/writing/my-ai-adoption-journey)). Shankar: plan mode is essential for large features [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)). Sankalp (2025-12-27): prefers asking clarifying/exploratory questions over plan mode [anecdote] ([Sankalp](https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/)).
- **Parallelism, for:** incident.io (2025-06-27): Git worktree per session via a bash helper `w`; 4–5 concurrent agents per engineer after ~4 months; plan mode so parallel sessions cannot make unauthorized changes [self-reported figure] ([incident.io](https://www.incident.io/blog/shipping-faster-with-claude-code-and-git-worktrees)). Quigley (Sanity): multiple instances on separate problem spaces tracked in the PM tool [anecdote] ([Sanity](https://www.sanity.io/blog/first-attempt-will-be-95-garbage)).
- **Parallelism, limited:** Simon Willison (2025-10-05): parallel agents suit research/proofs of concept, explaining code, low-stakes maintenance, carefully specified work; uses fresh checkouts in `/tmp` rather than worktrees; "Code that started from your own specification is a lot less effort to review." [anecdote] ([Willison](https://simonwillison.net/2025/Oct/5/parallel-coding-agents/)). Hashimoto: one background agent on low-priority work with notifications off; lists running multiple agents at once as something he does not do [anecdote] ([Hashimoto](https://mitchellh.com/writing/my-ai-adoption-journey)).
- **Review loops.** Quigley (Sanity): Claude reviews first → engineer reviews maintainability/architecture/business logic → normal team review; human-edited code marked so the AI does not overwrite it [anecdote] ([Sanity](https://www.sanity.io/blog/first-attempt-will-be-95-garbage)). Sankalp: implements with Claude Code (Opus 4.5, as reported), reviews with GPT-5.2-Codex `/review` (as reported) [anecdote] ([Sankalp](https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/)). Hashimoto: output on junior/mid-level tasks "always" needs thorough review (secondary summary) [anecdote] ([ldirer notes](https://ldirer.com/blog/posts/mitchell-hashimoto-zed-agents-2025)).

## 3. Context management

### Official

- "Most best practices are based on one constraint: Claude's context window fills up fast, and performance degrades as it fills." Monitor usage with a custom status line ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Commands:** `/clear`; `/compact <instructions>` (e.g. `/compact Focus on the API changes`); `/rewind` → **Summarize from here** / **Summarize up to here**; `/autocompact 500k` sets the threshold; `/btw` for side questions whose answer never enters history; `/context` inspects usage ([Best practices](https://code.claude.com/docs/en/best-practices); [Context window](https://code.claude.com/docs/en/context-window)).
- **Compaction survivors:** system prompt and output style; project-root CLAUDE.md, unscoped rules, and auto memory re-read from disk; plan-mode plan; up to five most recently read/edited files (files >5,000 tokens become path references); invoked skill bodies capped at 5,000 tokens per skill / 25,000 total (keep key instructions at the top of `SKILL.md`). `paths:` rules and nested CLAUDE.md are dropped until re-triggered ([Context window](https://code.claude.com/docs/en/context-window); [Memory](https://code.claude.com/docs/en/memory)).
- Compaction instructions can live in CLAUDE.md, e.g. "When compacting, always preserve the full list of modified files and any test commands." ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **1M-token windows:** Fable models, Sonnet 5+, Opus 4.6+, Sonnet 4.6 (`[1m]` variants; Sonnet 5/5.5 default to 1M) ([Context window](https://code.claude.com/docs/en/context-window)).
- **Subagents for reads:** "Since context is your fundamental constraint, use subagents to keep research out of it." ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Context engineering** (2025-09-29): "the set of strategies for curating and maintaining the optimal set of tokens (information) during LLM inference"; context rot (recall falls as tokens rise); system prompts at the right "altitude", started minimal and extended based on observed failures; self-contained, minimally overlapping, token-efficient tools; a few diverse canonical examples over exhaustive edge-case lists; just-in-time retrieval via lightweight identifiers; compaction; structured notes in external files; subagents returning condensed summaries ([Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)).
- **Multi-window work:** use a different prompt for the first window (set up tests, scripts); track tests in structured form (e.g. `tests.json`) with "It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality"; create `init.sh`; consider starting fresh over compaction, since models "are extremely effective at discovering state from the local filesystem" ("Review progress.txt, tests.json, and the git logs"); JSON for structured state, free text for progress, git for checkpoints ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- For auto-compacting harnesses, tell the model: "Your context window will be automatically compacted as it approaches its limit... do not stop tasks early due to token budget concerns." ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- **Long-running harness** (2025-11-26): an initializer agent writes `init.sh`, `claude-progress.txt`, a JSON feature list with `passes: false`, and an initial commit; later agents read progress and git log, work one feature at a time, commit, and verify end-to-end (Puppeteer MCP) before marking done. Targets premature completion and undocumented progress ([Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)).

### Practitioner

- Shankar (2025-11-02): avoids `/compact` as opaque and error-prone; default is `/clear` + a custom `/catchup` (read files changed on the branch); for complex work, "Document & Clear": dump progress to Markdown, clear, reload [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)).
- Sankalp (2025-12-27): watches `/context`; starts a new session or compacts near 60% utilization on complex tasks; uses a custom `/handoff` command [anecdote] ([Sankalp](https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/)).
- Vincent van Deth (2026): reports degradation past ~65%; agent writes a handover doc at 65% (before 80% auto-compaction), then `/clear` and reload [anecdote; thresholds are personal heuristics] ([van Deth](https://vincentvandeth.nl/blog/context-rot-claude-code-automatic-rotation)).
- Ronacher (2025-07-30): when long sessions degrade, start fresh and pass state via Markdown [anecdote] ([Ronacher](https://lucumr.pocoo.org/2025/7/30/things-that-didnt-work/)).
- Reed, Beck, Huntley plan-file workflows (§2) keep state on disk, so sessions can be cleared without loss.

### Conflicts

- Official docs offer `/compact <instructions>` and partial summarize as primary tools; several practitioners (Shankar, van Deth, Ronacher) prefer handoff-file-then-`/clear`. The official platform guidance also says to "consider starting fresh over compaction" for multi-window work, which aligns with the practitioner position.

## 4. Extension points and configuration

### Official

- **Decision rule ("build your setup over time"):** convention wrong twice → CLAUDE.md; repeated format requests → output style; same starting prompt → user-invocable skill; same playbook pasted a third time → skill; copying data from a browser tab → MCP server; many file reads to find symbols → code intelligence plugin; side task floods conversation → subagent; want something every time → hook; second repo needs the same setup → plugin ([Extend Claude Code](https://code.claude.com/docs/en/features-overview)).
- **Context cost per mechanism:** CLAUDE.md and output style every request; skill descriptions every request, body on use (`disable-model-invocation: true` → zero until invoked); MCP tool names at start with schemas deferred (tool search on by default); subagents isolated; hooks zero unless they return output ([Extend Claude Code](https://code.claude.com/docs/en/features-overview)).
- **Hooks:** "Unlike CLAUDE.md instructions which are advisory, hooks are deterministic and guarantee the action happens." "An instruction like 'never edit `.env`'... is a request, not a guarantee. A `PreToolUse` hook that blocks the edit is enforcement." Hook handlers: shell commands, HTTP requests, MCP tool calls, LLM prompts, subagents. Configure in `.claude/settings.json`; browse with `/hooks`. Events include `PreToolUse`, `PostToolUse`, `Stop`, `SessionStart` (source `compact` re-injects context after compaction), `ConfigChange`, `InstructionsLoaded` ([Best practices](https://code.claude.com/docs/en/best-practices); [Extend Claude Code](https://code.claude.com/docs/en/features-overview); [Hooks guide](https://code.claude.com/docs/en/hooks-guide)).
- **Skills:** `.claude/skills/<name>/SKILL.md` with `name`/`description` frontmatter; auto-invoked by description match or as `/skill-name`; `disable-model-invocation: true` for side-effecting workflows; `context: fork` runs in a subagent; `allowed-tools` pre-approves tools for that turn; "Keep `SKILL.md` under 500 lines. Move detailed reference material to separate files." Vague or overlapping descriptions cause missed or wrong loads ([Skills](https://code.claude.com/docs/en/skills); [Best practices](https://code.claude.com/docs/en/best-practices); [Extend Claude Code](https://code.claude.com/docs/en/features-overview)).
- **Subagents:** `.claude/agents/<name>.md` with `name`, `description`, `tools`, `model` frontmatter. Keep them focused; write descriptions that single out one subagent (combined description budget 15,000 tokens); limit tool access; version-control them; "use proactively" in a description encourages delegation. Built-in Explore and Plan agents omit CLAUDE.md and git status ([Subagents](https://code.claude.com/docs/en/sub-agents); [Best practices](https://code.claude.com/docs/en/best-practices); [Extend Claude Code](https://code.claude.com/docs/en/features-overview)).
- **Dynamic workflows** (scripts Claude writes that run many subagents) for jobs beyond a handful of subagents or needing cross-checked findings; `/batch <instruction>` splits a change across 5–30 subagents, each in its own worktree ([Extend Claude Code](https://code.claude.com/docs/en/features-overview); [Best practices](https://code.claude.com/docs/en/best-practices)).
- **MCP:** `claude mcp add --transport http notion https://mcp.notion.com/mcp`; `/mcp` for status; `/context all` for per-tool token use; pattern: MCP provides the connection, a skill teaches usage ([Best practices](https://code.claude.com/docs/en/best-practices); [Extend Claude Code](https://code.claude.com/docs/en/features-overview)).
- **Plugins:** `/plugin` to browse; bundle skills, hooks, subagents, MCP servers; plugin skills are namespaced (`/my-plugin:review`); install a code intelligence (LSP) plugin for typed languages ([Best practices](https://code.claude.com/docs/en/best-practices); [Extend Claude Code](https://code.claude.com/docs/en/features-overview)).
- **Output styles:** set role/tone/format for every response; one active at a time; not enforced ([Extend Claude Code](https://code.claude.com/docs/en/features-overview)).
- **Layering:** CLAUDE.md additive; skills override by name (managed > user > project); subagents (managed > CLI flag > project > user > plugin); MCP (local > project > user); hooks merge ([Extend Claude Code](https://code.claude.com/docs/en/features-overview)).

### Practitioner

- **Subagents, negative reports.** Ronacher (2025-07-30): subagents/task tool did not work well; tasks mixing reads and writes caused problems; fresh sessions or Markdown state worked better [anecdote] ([Ronacher](https://lucumr.pocoo.org/2025/7/30/things-that-didnt-work/)). Shankar (2025-11-02): custom subagents "gatekeep context" and force rigid workflows; prefers the generic built-in `Task(...)` clone and lets the main agent decide [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)). Sankalp: Explore is useful for read-only search, but the main model should read key files itself because summaries lose detail [anecdote] ([Sankalp](https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/)).
- **Commands.** Ronacher: `/fix-bug`, `/commit`, `/add-tests`, `/fix-nits`, `/next-todo` never became habits (unstructured arguments, no file autocomplete); uses speech-to-text and pasted prompts [anecdote] ([Ronacher](https://lucumr.pocoo.org/2025/7/30/things-that-didnt-work/)). Shankar: keeps only a few (`/catchup`, `/pr`); large command lists become undocumented "magic" [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)). Reed: reusable prompts in `.claude/commands/` with arguments (e.g. `/user:gh-issue #45`) [anecdote] ([Reed](https://harper.blog/2025/05/08/basic-claude-code/)).
- **Hooks.** Shankar: block-at-submit — a `PreToolUse` hook on `git commit` checks for a test-pass marker file, forcing a test-and-fix loop; avoid block-at-write, which confuses the agent mid-plan [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)). Ronacher (2025-07): hooks could only deny, not steer, and were ineffective in skip-permissions mode; replaced with PATH-prepended shell "interceptors" in `.claude/interceptors` [anecdote; dated, see Conflicts] ([Ronacher](https://lucumr.pocoo.org/2025/7/30/things-that-didnt-work/)).
- **MCP vs CLI.** Shankar: skills (scripts/CLIs/docs) more useful than MCP; migrated Jira/AWS/GitHub MCP usage to CLIs; keeps Playwright MCP [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)). Ronacher: minimal MCP ("MCP servers themselves are sometimes not super reliable"); prefers shell tools such as `psql` [anecdote] ([Ronacher](https://lucumr.pocoo.org/2025/6/12/agentic-coding/)).
- **Settings.** Shankar: raises `BASH_MAX_TIMEOUT_MS` and `MCP_TOOL_TIMEOUT` in settings.json; periodically audits allowed-command permissions [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)).

### Conflicts and superseded items

- **Custom commands merged into skills (official).** "Custom commands have been merged into skills." `.claude/commands/deploy.md` still works, but "Prefer a skill for new work"; a skill wins on name collision ([Skills](https://code.claude.com/docs/en/skills)). Practitioner advice about `.claude/commands/` (Reed, Ronacher, Shankar) predates this.
- **Subagents:** official docs recommend focused custom subagents; Ronacher and Shankar report negative experience with custom subagents (2025). Official docs also add damping prompts for over-delegation (§6).
- **Hooks:** Ronacher's mid-2025 limitation (deny-only) predates current docs, which list Stop hooks that block turn end, `SessionStart` context injection, and prompt/subagent hook handlers ([Hooks guide](https://code.claude.com/docs/en/hooks-guide); [Best practices](https://code.claude.com/docs/en/best-practices)). Whether hooks now fire under `bypassPermissions` was not verified.

## 5. Automation and CI

### Official

- **Headless:** `claude -p "prompt"`; `--output-format json` returns an object with `result`; `stream-json` requires `--verbose`; runs are resumable unless `--no-session-persistence` ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **`--bare`** skips auto-discovery of hooks, skills, commands, subagents, plugins, MCP servers, auto memory, and CLAUDE.md; it is the "recommended mode for scripted and SDK calls, and will become the default for `-p` in a future release." Without `--bare`, `-p` runs the project's hooks and `.mcp.json` servers even in an untrusted folder with no trust dialog. `--json-schema` returns `structured_output` ([Headless](https://code.claude.com/docs/en/headless)).
- **Fan-out:** have Claude write a task list (`files.txt`), then loop `claude -p "Migrate $file ... Return OK or FAIL." --allowedTools "Edit,Bash(git commit *)"`; test on 2–3 files and refine before running all ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Unattended auto mode:** `claude --permission-mode auto -p "fix all lint errors"`; in `-p` runs, repeated classifier blocks do not stop the run ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Locked-down CI:** `claude -p "run the test suite" --permission-mode dontAsk --allowedTools "Bash(npm test)" "Read"` ([Permission modes](https://code.claude.com/docs/en/permission-modes)).
- **GitHub Actions:** `anthropics/claude-code-action`; `/install-github-app` (github.com only) installs the app, secret, and workflow PR; `@claude` mentions trigger interactive mode; `examples/claude.yml`; optional `claude-code-review.yml` ([GitHub Actions](https://code.claude.com/docs/en/github-actions)).
- **Scheduling:** Routines (cloud), Desktop scheduled tasks, GitHub Actions, `/loop`. Scheduled prompts must state what success looks like and what to do with results, since they cannot ask questions ([Common workflows](https://code.claude.com/docs/en/common-workflows)).
- Anthropic Product Design automated PR comments via GitHub Actions ([How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code)).

### Practitioner

- Shankar (2025-11-02): Claude Code GitHub Action with a custom container, hooks, and MCP lets PRs be triggered from Slack, Jira, CloudWatch alerts; audit logs are queried for common mistakes that feed back into CLAUDE.md and internal CLIs; uses the SDK for large parallel scripted refactors [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)).
- Ronacher (2025-07-30): headless/print mode for mostly-deterministic scripts was "slow and difficult to debug" [anecdote] ([Ronacher](https://lucumr.pocoo.org/2025/7/30/things-that-didnt-work/)).
- Reed: `pre-commit` hooks checked into the repo gate agent commits [anecdote] ([Reed](https://harper.blog/2025/05/08/basic-claude-code/)).
- Team adoption: Quigley (Sanity, 2025-09-02): AI review → engineer review → normal team review; MCP connections to Linear, Notion, read-only DBs, GitHub; background agents lacked private NPM access and bypassed normal tracking; start with repetitive tasks and budget for a messy first month [anecdote] ([Sanity](https://www.sanity.io/blog/first-attempt-will-be-95-garbage)). incident.io: shared worktree helper; voice dictation (SuperWhisper) for requirements [anecdote] ([incident.io](https://www.incident.io/blog/shipping-faster-with-claude-code-and-git-worktrees)).

## 6. Prompting (current Claude models)

### Official

- "Think of Claude as a brilliant but new employee who lacks context"; test a prompt by showing it to a colleague with minimal context; request "above and beyond" behavior explicitly ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- **Give reasons** behind constraints; "Claude is smart enough to generalize from the explanation" ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- **Action vs suggestion:** "Can you suggest some changes..." yields suggestions; "Change this function..." yields edits. Sample `<default_to_action>` and `<do_not_act_before_instructions>` blocks ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- **Remove aggressive emphasis:** Opus 4.5/4.6 respond more strongly to system prompts; replace "CRITICAL: You MUST use this tool when..." with "Use this tool when..."; drop "If in doubt, use [tool]"; lower `effort` as fallback ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- **Over-engineering guard:** sample prompt "Avoid over-engineering. Only make changes that are directly requested or clearly necessary", covering scope, documentation ("Don't add docstrings, comments, or type annotations to code you didn't change"), defensive coding ("Only validate at system boundaries"), and abstractions ("Don't create helpers... for one-time operations") ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- **Test special-casing guard:** "Tests are there to verify correctness, not to define the solution... If... any of the tests are incorrect, please inform me rather than working around them." ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- **Hallucination guard:** `<investigate_before_answering>` "Never speculate about code you have not opened." ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- Ask Claude to clean up temporary scratch files at task end; sample `<use_parallel_tool_calls>` prompt raises parallel tool use to ~100% ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- **Subagent over-delegation** (Opus 4.6; Opus 5 delegates "more readily"): sample damping prompt "Use subagents when tasks can run in parallel, require isolated context, or involve independent workstreams... For simple tasks, sequential operations, single-file edits... work directly." ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- **Overthinking:** "choose an approach and commit to it"; lower effort ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- Model-specific pages exist for Fable 5.1, Fable 5, Sonnet 5.5, Sonnet 5, Opus 5.5, Opus 5, Opus 4.8 (effort calibration, literal instruction following, over-verification, subagent control, progress updates). Re-check techniques with evals before carrying them to another model ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- Claude Code task prompts: scope the task, point to sources (e.g. git history), reference existing patterns (e.g. `HotDogWidget.php`), describe symptom + location + definition of "fixed" ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Effort / thinking (current):** levels `low`, `medium`, `high`, `xhigh`, `max` (Opus 4.6/Sonnet 4.6 lack `xhigh`), set via `/effort`, `--effort`, or `CLAUDE_CODE_EFFORT_LEVEL`. Opus 5.5 defaults to `medium`. "Include `ultrathink` anywhere in your prompt to request deeper reasoning on that turn... The effort level sent to the API is unchanged." Thinking cannot be turned off on Opus 5.5, Sonnet 5.5, or Fable models. `/effort ultracode` makes Claude orchestrate dynamic workflows for substantive tasks ([Model configuration](https://code.claude.com/docs/en/model-config)).

### Practitioner

- Shankar: replace bare prohibitions ("Never use X") with a preferred alternative ("Prefer Y") [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)).
- Huntley: counters placeholder implementations with "DO NOT IMPLEMENT PLACEHOLDER OR SIMPLE IMPLEMENTATIONS. WE WANT FULL IMPLEMENTATIONS." and false "not implemented" conclusions with "don't assume it's not implemented" [anecdote] ([Huntley](https://ghuntley.com/ralph/)). Note: the all-caps style conflicts with official advice to drop aggressive emphasis for current models.
- Ronacher (2025-06-12): code style reduces agent confusion: descriptive functions over classes, plain SQL, local permission checks, no deep inheritance [anecdote] ([Ronacher](https://lucumr.pocoo.org/2025/6/12/agentic-coding/)).

### Superseded

- **Thinking keywords.** The April 2025 post's ladder "think" < "think hard" < "think harder" < "ultrathink" mapped to thinking budgets. Current docs: only `ultrathink` is recognized, and it adds an in-context instruction without changing the effort sent to the API; "think", "think hard", "think more" pass through as plain text ([Model configuration](https://code.claude.com/docs/en/model-config)).
- **`budget_tokens`** (API extended thinking) is deprecated on Opus 4.6/Sonnet 4.6 and returns a 400 error on Claude 4.7 and later; use `effort` or `max_tokens` with adaptive thinking ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- `/doctor prompt-audit` flags instructions "written for older models" ([Memory](https://code.claude.com/docs/en/memory)); prompts accumulated across model generations need re-tuning.

## 7. Security and permissions

### Official

- **Permission modes:** `default` (Manual; alias `manual` v2.1.200+; reads only without asking); `acceptEdits` (reads, edits, common fs commands); `plan` (reads, plus classifier-approved commands when auto mode is available); `auto` (all actions with background safety checks); `dontAsk` (only pre-approved tools; for locked-down CI); `bypassPermissions` ("Isolated containers and VMs only"). Deny rules apply in every mode, including bypass. Switch with `Shift+Tab` or `--permission-mode` ([Permission modes](https://code.claude.com/docs/en/permission-modes)).
- **Auto mode** is the built-in starting mode for interactive terminal and VS Code sessions from v2.1.283 (earlier: Pro/Max/Team only). Requires Opus 4.6+, Sonnet 4.6+, or Fable on the Anthropic API. Admins disable with `permissions.disableAutoMode: "disable"`. `defaultMode: "auto"` is ignored in project `.claude/settings*.json`; it must be in `~/.claude/settings.json` ([Permission modes](https://code.claude.com/docs/en/permission-modes)).
- **Auto mode blocks by default:** `curl | bash`-style download-and-execute, sending sensitive data externally, production deploys/migrations, mass cloud-storage deletion, granting IAM/repo permissions, modifying shared infrastructure. It trusts the working directory and remotes configured at session start. After 3 consecutive or 20 total blocks it pauses and resumes prompting (not configurable). Boundaries stated in chat ("don't push") can be lost to compaction — "For a hard guarantee, add a deny rule." "Auto mode reduces permission prompts but does not guarantee safety." ([Permission modes](https://code.claude.com/docs/en/permission-modes)).
- **Fully unattended:** `claude -p "<prompt>" --dangerously-skip-permissions` only inside a container/VM/sandbox runtime as a non-root user ([Permission modes](https://code.claude.com/docs/en/permission-modes)).
- **Allowlists and sandbox:** `/permissions` to allowlist (e.g. `npm run lint`, `git commit`); `/sandbox` for OS-level isolation. "After the tenth approval you're clicking through rather than reviewing." ([Best practices](https://code.claude.com/docs/en/best-practices)).
- **Sandboxing** (2025-10-20): "sandboxing safely reduces permission prompts by 84%" in Anthropic internal use [measured, vendor-internal]; requires both filesystem and network isolation ("without network isolation, a compromised agent could exfiltrate sensitive files like SSH keys; without filesystem isolation, a compromised agent could easily escape the sandbox"); uses Linux bubblewrap and macOS Seatbelt; runtime open-sourced ([Claude Code sandboxing](https://www.anthropic.com/engineering/claude-code-sandboxing); [Sandboxing docs](https://code.claude.com/docs/en/sandboxing)).
- **Prompt injection protections:** permission system; context-aware analysis; input sanitization; `curl`/`wget` not auto-approved; web fetch in an isolated context window; trust verification for first-time codebases and new MCP servers (disabled with `-p`); command-injection detection; fail-closed matching. User practices: review commands before approval; avoid piping untrusted content; verify changes to critical files; use VMs for scripts touching external web services; report via `/feedback` ([Security](https://code.claude.com/docs/en/security)).
- **MCP trust:** use self-written or trusted-provider servers; Anthropic "does not security-audit or manage any MCP server." ([Security](https://code.claude.com/docs/en/security)).
- **Teams:** managed settings for org standards; permission configs in version control; `/permissions` audits; OpenTelemetry monitoring; `ConfigChange` hooks; `/security-review` command and security guidance plugin ([Security](https://code.claude.com/docs/en/security)).
- **CI trust boundary:** non-`--bare` `-p` runs repository hooks and `.mcp.json` servers without a trust dialog ([Headless](https://code.claude.com/docs/en/headless)).
- **Prompt-level guard:** "Consider the reversibility and potential impact of your actions... ask the user before proceeding" for `rm -rf`, `git push --force`, `git reset --hard`; "don't bypass safety checks (e.g. --no-verify)" ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).

### Practitioner

- Ronacher (2025-06-12): aliases `claude --dangerously-skip-permissions` as `claude-yolo`; mitigates with Docker dev environments, though reports it works acceptably without [anecdote] ([Ronacher](https://lucumr.pocoo.org/2025/6/12/agentic-coding/)).
- Willison (2025-10-05): YOLO mode locally for trusted work; says he should habitually run local agents in Docker; uses cloud async agents for riskier tasks [anecdote] ([Willison](https://simonwillison.net/2025/Oct/5/parallel-coding-agents/)).
- incident.io (2025-06-27): plan mode as the safety mechanism for unattended parallel sessions [anecdote] ([incident.io](https://www.incident.io/blog/shipping-faster-with-claude-code-and-git-worktrees)).

### Superseded

- The 2025 guidance centered on "safe YOLO mode" in a container; the current default interactive path is auto mode, with `bypassPermissions` still restricted to isolated environments ([Permission modes](https://code.claude.com/docs/en/permission-modes)). The original 2025 wording could not be re-read because the post redirects.
- Practitioner reports of routine `--dangerously-skip-permissions` use (Ronacher, Willison, 2025) predate auto mode becoming the default (v2.1.283).

## 8. Failure modes

### Official

- Named failure patterns: kitchen-sink session; correcting over and over; over-specified CLAUDE.md; trust-then-verify gap ("If you can't verify it, don't ship it"); infinite exploration ([Best practices](https://code.claude.com/docs/en/best-practices)).
- Model-level tendencies addressed with sample prompts: over-engineering, test special-casing, speculation about unopened code, subagent overuse, overthinking, overtriggering on emphatic instructions (§6) ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- Long-running agents: premature completion and undocumented progress, addressed with feature lists, progress files, and end-to-end verification ([Effective harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)).
- Reviewers asked to find gaps report some even for sound work; chasing all findings leads to over-engineering ([Best practices](https://code.claude.com/docs/en/best-practices)).

### Practitioner

- Kent Beck (2025-06-25): warning signs are loops, unrequested functionality "even if logically sensible", and "any indication that the genie was cheating, for example by disabling or deleting tests"; mitigation: watch intermediate output and stop unproductive runs [anecdote] ([Beck](https://newsletter.kentbeck.com/p/augmented-coding-beyond-the-vibes)).
- Huntley (2025-07-14): placeholder implementations; false "not implemented" conclusions from ripgrep; going off the rails (regenerate the TODO list); nondeterministic build errors filling context [anecdote] ([Huntley](https://ghuntley.com/ralph/)).
- Quigley (Sanity, 2025-09-02): first attempt ~95% unusable, second ~50%, third a workable start [anecdote]; confident broken code in state management, performance-critical, and security-sensitive areas; no learning continuity across sessions ([Sanity](https://www.sanity.io/blog/first-attempt-will-be-95-garbage)).
- HN discussion of Anthropic's "An update on recent Claude Code quality reports" (2026, Opus/Sonnet 4.6 era): Boris Cherny (Anthropic) described a bug making sessions idle >1 hour "forgetful and repetitive"; resuming after >1 h idle also causes full prompt-cache misses (example: "900k tokens written to cache all at once"). User workarounds: `/clear` before resuming old sessions; external context files [anecdote] ([HN 47878905](https://news.ycombinator.com/item?id=47878905)).
- Hashimoto (2026-02-05): when the agent repeats a mistake, add an AGENTS.md rule or a verification tool rather than re-correcting in chat [anecdote] ([Hashimoto](https://mitchellh.com/writing/my-ai-adoption-journey)).
- Research on test cheating by coding agents (not Claude-specific, not read in detail): [arXiv 2605.21384](https://arxiv.org/pdf/2605.21384) (SpecBench); [arXiv 2606.07379](https://arxiv.org/pdf/2606.07379).

## 9. Cost and model selection

### Official

- Effort levels and defaults: see §6 ([Model configuration](https://code.claude.com/docs/en/model-config)). Lower effort is the documented fallback for overtriggering and overthinking ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- 1M-context model variants: see §3 ([Context window](https://code.claude.com/docs/en/context-window)).
- CLI tools are cheaper in context than MCP equivalents; MCP schemas are deferred by tool search ([Best practices](https://code.claude.com/docs/en/best-practices); [Extend Claude Code](https://code.claude.com/docs/en/features-overview)).

### Practitioner (model names as reported, tied to post date)

- Ronacher (2025-06): Max plan at $100/month; Sonnet only ("perfectly adequate") [anecdote] ([Ronacher](https://lucumr.pocoo.org/2025/6/12/agentic-coding/)).
- Quigley (Sanity, 2025-09): $1,000–1,500/month per senior engineer using AI heavily; AI writes ~80% of initial implementations; features ship 2–3x faster [self-reported figure, method not described] ([Sanity](https://www.sanity.io/blog/first-attempt-will-be-95-garbage)).
- incident.io (2025-06-27): ~$8 of Claude credits for a build-tooling change that made API generation 18% (30 s) faster [measured, single case] ([incident.io](https://www.incident.io/blog/shipping-faster-with-claude-code-and-git-worktrees)).
- Reed (2025-05-08): Claude Code is "a hell of a lot more expensive" than his prior tooling [anecdote] ([Reed](https://harper.blog/2025/05/08/basic-claude-code/)).
- Huntley relays a claim of a $50k contract delivered for $297 via Ralph [unverified, no methodology] ([Huntley](https://ghuntley.com/ralph/)).
- Shankar: enterprise `ANTHROPIC_API_KEY` usage-based billing handles variance across developers better than per-seat plans [anecdote] ([Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)).
- Sankalp (2025-12): Opus 4.5 primary (better intent detection); Sonnet 4.5 faster but makes "haphazard changes"; GPT-5.2-Codex for review [anecdote] ([Sankalp](https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/)).
- HN (2026): Pro vs Max experiences differ because quota resets interact with caching; idle-resume cache misses raise token cost [anecdote] ([HN 47878905](https://news.ycombinator.com/item?id=47878905)).
- Model preference claims from 2025 (Sonnet vs Opus 4.x) apply to those model versions only.

## 10. Superseded and conflicting guidance (index)

| Item | Status | Source |
|---|---|---|
| April 2025 engineering post | Redirects (308) to docs page; superseded | [redirect](https://www.anthropic.com/engineering/claude-code-best-practices) |
| "think" / "think hard" / "think harder" keyword ladder | Superseded; only `ultrathink` recognized, no effort change | [Model configuration](https://code.claude.com/docs/en/model-config) |
| API `budget_tokens` | Deprecated on 4.6; 400 error on 4.7+ | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| `.claude/commands/` | Merged into skills; still works; prefer skills | [Skills](https://code.claude.com/docs/en/skills) |
| All-caps "CRITICAL/MUST" prompting | Discouraged for Opus 4.5+ | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| `claude -p` without `--bare` | `--bare` to become the `-p` default in a future release | [Headless](https://code.claude.com/docs/en/headless) |
| Manual permission mode as default | Auto mode is the starting mode from v2.1.283 | [Permission modes](https://code.claude.com/docs/en/permission-modes) |
| AGENTS.md via symlink/import only | Read natively since v2.1.277 when no CLAUDE.md exists | [Memory](https://code.claude.com/docs/en/memory) |
| `#` shortcut to add memories (2025 post) | Not in current memory docs; no deprecation notice found | [Memory](https://code.claude.com/docs/en/memory) |
| Hooks "deny only" (Ronacher, 2025-07) | Predates current hook capabilities | [Hooks guide](https://code.claude.com/docs/en/hooks-guide) |
| CLAUDE.md length | Official <200 lines/file vs practitioner <60 to 25 KB | §1 |
| `/compact` vs handoff + `/clear` | Official offers both; several practitioners avoid `/compact` | §3 |
| Custom subagents | Official recommends; Ronacher, Shankar report poor results (2025) | §4 |

## Open questions / gaps

- No controlled measurement of CLAUDE.md length vs instruction adherence; HumanLayer's 150–200-instruction figure has no cited source.
- No quantitative comparison of worktree-parallel vs single-session throughput including review time; team productivity figures (Sanity 2–3x, incident.io 4–5 agents) are self-reported.
- No measured rate of test deletion/modification by Claude Code; the arXiv test-cheating papers were not read.
- Default auto-compact thresholds per model, the full hook event list and JSON output schema, and Agent SDK practices were not extracted.
- Per-model prompting pages (Opus 5.5, Sonnet 5.5, Fable) were not fetched individually.
- Status of the `#` memory shortcut is unconfirmed.
- Practitioner coverage after early 2026 is thin; no named-team published `.claude/settings.json` allowlists found; no cost-per-merged-PR data.

Retrieved: 2026-09-30
