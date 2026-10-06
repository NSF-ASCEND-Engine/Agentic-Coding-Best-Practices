# OpenAI Codex best practices: official guidance and practitioner reports

Reference notes for the agentic-coding book. Scope: OpenAI Codex CLI, IDE extension, desktop app, and Codex Cloud, plus OpenAI prompting guides for Codex models, as of 2026-09-30.

Conventions:

- **Official** = OpenAI docs and OpenAI Cookbook. **Practitioner** = named individuals, company blogs, HN comments, GitHub issues; each tagged **[anecdote]**, **[self-reported figure]**, **[measured]**, or **[aggregator, lower confidence]** (SEO/guide sites).
- **Doc location:** Codex docs formerly at `developers.openai.com/codex/...` return HTTP 308 redirects to `learn.chatgpt.com/docs/...` or `learn.chatgpt.com/guides/...` ("ChatGPT Learn"). Citations use the current URL where fetched. Official pages were read through a summarizing fetch tool; spot-check quoted phrases before verbatim reuse. Most pages are undated.
- **Model names:** the official Codex models page lists `gpt-6-astra`, `gpt-6.1-sol`, `gpt-6-luna` ([Codex models](https://learn.chatgpt.com/codex/models)). Practitioner posts name other models (e.g. "gpt-5.6-sol", "GPT-5.6 Sol Ultra", "Luna", "opus-5", "Fable"); these are reproduced as reported and not confirmed. The official page does not list any "GPT-5.6"; the official "Sol" model is `gpt-6.1-sol`.

## 1. Instruction files (AGENTS.md)

### Official

- **Global scope:** Codex reads `~/.codex` (or `$CODEX_HOME`), checking `AGENTS.override.md` first, then `AGENTS.md` ([AGENTS.md docs](https://learn.chatgpt.com/docs/agent-configuration/agents-md)).
- **Project scope:** Codex walks from the Git root down to the current directory; in each directory it checks `AGENTS.override.md`, then `AGENTS.md`, then names in `project_doc_fallback_filenames`; at most one file per directory ([AGENTS.md docs](https://learn.chatgpt.com/docs/agent-configuration/agents-md)).
- **Merge order:** concatenated root → leaf; closer files override earlier guidance ([AGENTS.md docs](https://learn.chatgpt.com/docs/agent-configuration/agents-md); [Codex Prompting Guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide)).
- **Size cap:** `project_doc_max_bytes` defaults to 32 KiB; Codex stops adding files once reached. If truncated, raise the limit or split across nested directories. Example `~/.codex/config.toml`: `project_doc_fallback_filenames = ["TEAM_GUIDE.md", ".agents.md"]`, `project_doc_max_bytes = 65536` ([AGENTS.md docs](https://learn.chatgpt.com/docs/agent-configuration/agents-md)).
- Empty files are skipped; the chain is rebuilt every run (no cache); `AGENTS.override.md` is for temporary replacements without deleting the base file ([AGENTS.md docs](https://learn.chatgpt.com/docs/agent-configuration/agents-md)).
- **Setup:** create a root `AGENTS.md` "to encode team guidance"; scaffold with `/init`; create a `code_review.md` referenced from `AGENTS.md` for consistent reviews ([Best practices](https://learn.chatgpt.com/guides/best-practices)).
- **Review rules** go in a `## Code Review Rules` section: describe repo-specific risks ("the compatibility constraint, data boundary, or unsafe side effect to flag and why it matters"); give safe alternatives so Codex can separate real issues from expected behavior; state outcomes rather than implementation details; leave formatting and linting to CI; repo-wide rules in root `AGENTS.md`, service rules in nested files such as `services/example/AGENTS.md` ([GitHub integration](https://learn.chatgpt.com/docs/third-party/github)).
- **ExecPlans pointer:** add a line such as "When writing complex features or significant refactors, use an ExecPlan (as described in .agent/PLANS.md)." ([ExecPlans cookbook](https://developers.openai.com/cookbook/articles/codex_exec_plans)).
- No `@import` include syntax is documented; modularity comes from nested and override files ([AGENTS.md docs](https://learn.chatgpt.com/docs/agent-configuration/agents-md)).

### Practitioner

- Peter Steinberger (~Oct–Dec 2025): AGENTS.md of ~800 lines ("scar tissue") covering git commit rules, product naming, React/React Compiler patterns, DB migration and test protocols, ast-grep lint rules, a text design system; updates written by the model; expects it to shrink as models improve. Also a global AGENTS.md plus project `docs/` folders indexed by a `docs:list` script [anecdote] ([Just Talk To It](https://steipete.me/posts/just-talk-to-it); [Shipping at Inference-Speed](https://steipete.me/posts/2025/shipping-at-inference-speed)).
- Guide sites: AGENTS.md under ~100 lines; specific instructions ("Use rg --files for discovery"); define "done" (tests pass, lint clean); list exact build/test/lint/format commands [aggregator, lower confidence] ([codegateway.dev](https://www.codegateway.dev/en/blog/agents-md-playbook-2026); [agensi.io](https://www.agensi.io/learn/codex-cli-agents-md-complete-guide)).
- Daniel Vaughan (2026-05-27): make AGENTS.md canonical for tool-agnostic content and have CLAUDE.md defer to it [anecdote] ([Vaughan](https://codex.danielvaughan.com/2026/05/27/agent-instruction-files-agents-md-claude-md-cross-tool-portability-codex-cli/)).
- **Reported defects (GitHub issues; fix status not checked):**
  - AGENTS.md silently truncated at 32 KB; content past the limit dropped without warning ([#13386](https://github.com/openai/codex/issues/13386), [#7138](https://github.com/openai/codex/issues/7138), [#43075](https://github.com/openai/codex/issues/43075)).
  - AGENTS.md not re-injected after `/compact` ([#2927](https://github.com/openai/codex/issues/2927)); auto-compaction loses AGENTS rules and task details ([#25792](https://github.com/openai/codex/issues/25792), [#18720](https://github.com/openai/codex/issues/18720)).
  - Global instructions ignored in one CLI version, only repo-local AGENTS.md honored ([#3540](https://github.com/openai/codex/issues/3540)).
  - Desktop app over-reports completion and compresses AGENTS.md instructions ([#31177](https://github.com/openai/codex/issues/31177)).
- Practical implication drawn from these issues: keep critical rules early in the file and the combined chain under 32 KiB (or raise `project_doc_max_bytes`); restart sessions rather than compact when rules matter.

### Conflicts

- **Length:** official docs give only the 32 KiB byte cap, no line target ([AGENTS.md docs](https://learn.chatgpt.com/docs/agent-configuration/agents-md)); guide sites say <100 lines; Steinberger ran ~800 lines.
- **Review heading:** current docs use `## Code Review Rules` ([GitHub integration](https://learn.chatgpt.com/docs/third-party/github)). Earlier (2025) docs are recalled as using `## Review guidelines`; not re-verified. Use the current heading.

## 2. Workflows

### Official

- **Prompt template:** "A good default is to include four things in your prompt: Goal, Context, Constraints, Done when." ([Best practices](https://learn.chatgpt.com/guides/best-practices)).
- "A useful Codex prompt names the behavior you want, points to the relevant code or reproduction steps, preserves important constraints, and says how to verify the change." ([Codex prompting](https://learn.chatgpt.com/codex/prompting)).
- **Per task** ([Codex prompting](https://learn.chatgpt.com/codex/prompting)):
  - Bug fixes: numbered repro steps, `@` file references, constraints such as API stability and minimal changes.
  - Tests: IDE selection, or function names and line ranges in the CLI.
  - UI: framework, routing, styling constraints, visual references.
  - Design iteration: "small, specific prompts".
  - Refactors: design locally with the `$plan` skill, then hand "long implementation to a cloud chat".
- **Context input:** the IDE includes open files automatically; in the CLI use `@` paths or `/mention` ([Codex prompting](https://learn.chatgpt.com/codex/prompting)).
- **Plan mode:** `/plan` or Shift+Tab for complex tasks; use `/plan` when a multi-step task needs investigation before editing ([Best practices](https://learn.chatgpt.com/guides/best-practices); [Codex prompting](https://learn.chatgpt.com/codex/prompting)).
- **Goals:** `/goal <objective>` sets a persistent completion contract across turns; lifecycle `/goal`, `/goal pause`, `/goal resume`, `/goal clear`. "A prompt says do this next; a Goal says keep working until this outcome is true." A Goal defines outcome, verification surface (tests, benchmarks, artifacts), constraints (what must not regress), boundaries (allowed resources), iteration policy, and blocked conditions (when to stop and report). Do not use Goals for one-off tasks, vague objectives ("improve performance"), or unclear finish lines ([Using Goals in Codex](https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex)). Use `/goal` after the plan ([Codex prompting](https://learn.chatgpt.com/codex/prompting)).
- **ExecPlans** (`.agent/PLANS.md`) for multi-hour work: a "living", self-contained design document a novice could implement from; focused on observable outcomes; exact commands with expected output. Required sections: Purpose/Big Picture; Progress (timestamped checkboxes); Surprises & Discoveries; Decision Log; Outcomes & Retrospective; Concrete Steps; Validation & Acceptance. Prose-first; checklists only in Progress ([ExecPlans cookbook](https://developers.openai.com/cookbook/articles/codex_exec_plans)).
- **Cloud:** create an environment once (repos, dependencies, tools) and reuse it; "Each task has its own workspace and can keep working while your computer is asleep"; parallel tasks run isolated. Review and publish the environment before tasks; configure secrets and env vars for credentials; restrict network destinations with an allowlist and enable internet only as needed; inspect changed files and test results before committing; open PRs from the task; republish when dependencies change ([Codex Cloud](https://learn.chatgpt.com/docs/cloud)).
- **Scheduled work** runs from the **Scheduled** page using Git worktrees ([Best practices](https://learn.chatgpt.com/guides/best-practices)).
- **Code review, local:** `/review`, optionally with focus instructions (edge cases, security) ([Codex prompting](https://learn.chatgpt.com/codex/prompting); [Best practices](https://learn.chatgpt.com/guides/best-practices)).
- **Code review, GitHub:** comment `@codex review` on a PR; findings focus on P0 and P1; automatic reviews enabled under Codex settings → Personal preferences; `@codex fix the P1 issue` pushes a fix; `@codex security review` and **Auto security review** are research preview ([GitHub integration](https://learn.chatgpt.com/docs/third-party/github)).

### Practitioner

- **Short prompts + screenshots.** Steinberger (~Oct 2025, GPT-5-Codex era): prompts "often it's just 1-2 sentences + an image"; ~50% include screenshots; conversational phrasing ("let's discuss", "give me options"); for hard problems "take your time", "comprehensive", "read all code that could be related" [anecdote] ([Just Talk To It](https://steipete.me/posts/just-talk-to-it)).
- **Conversation instead of plan mode.** Steinberger (~Dec 2025): "ask a question, let it google, explore code, create a plan together"; references sibling repos ("look at ../vibetunnel and do the same"); starts projects as CLIs so the agent can verify output directly [anecdote] ([Shipping at Inference-Speed](https://steipete.me/posts/2025/shipping-at-inference-speed)).
- **Detailed requirements.** HN user d-lo (Apr 2026): Codex does well with a "detailed list of requirements", "adheres to [prompts] really well and...asks clarifying questions"; Claude Code does better "given a broad task" [anecdote] ([HN 47750069](https://news.ycombinator.com/item?id=47750069)). Leanware: Codex "will build the wrong thing fast if you under-specify" [aggregator, lower confidence] ([Leanware](https://leanware.co/insights/codex-vs-claude-code)).
- **Parallel local sessions in one checkout.** Steinberger: 3–8 Codex CLI instances in a terminal grid, mostly in the same folder; tried worktrees and PRs but "always revert back" (multiple dev servers, OAuth domain limits); agents make atomic commits of only the files they edited, directly to main [anecdote] ([Just Talk To It](https://steipete.me/posts/just-talk-to-it); [Shipping at Inference-Speed](https://steipete.me/posts/2025/shipping-at-inference-speed)).
- **Several focused sessions.** Adrian Ghinda (2026-08-21) prefers several focused Codex sessions over one long session [anecdote] ([Ghinda](https://allaboutcoding.ghinda.com/a-week-of-using-codex-more-than-claude)).
- **Cloud vs local.** Daniel Vaughan (2026-03-27, updated 2026-09-30): local for interactive work, local state (env vars, DBs, servers), fast iteration; cloud for async self-contained tasks, ephemeral isolation, and best-of-N via `codex cloud exec --attempts 2-4`; cloud tasks appear in the Compliance API, local runs do not; agent phase internet off by default, setup scripts have internet; container cache 12 h [anecdote/opinion] ([Vaughan](https://codex.danielvaughan.com/2026/03/27/codex-cloud-vs-local-when-to-run-in-cloud/)).
- **Best-of-N cost.** Each attempt is billed separately (3 attempts ≈ 3x); compare attempts on approach, test results, diff size, errors [aggregator, lower confidence] ([developertoolkit.ai](https://developertoolkit.ai/en/codex/tips-tricks/cloud-workflows/)).
- **Autotriage.** Steinberger (X, 2026): a skill reads `VISION.md` and works on issues/PRs that fit the vision, have a clear fix, and can be live-tested, verified in a VM with computer vision; he reviews suggestions manually [anecdote] ([X post](https://x.com/steipete/status/2058240758801420530)).
- **Cross-vendor review, Claude writes / Codex reviews.** Salman Ali Banani (2026-07-04): Claude implements from a GitHub issue and opens a PR; a GitHub Action runs Codex review on `pull_request` (API key secret; diff annotated with line numbers); Claude makes one fix pass; merge. Rules: exactly one review and one fix pass ("I do not want an endless review loop"); scope limited to bugs, regressions, requirement mismatches, edge cases, missing tests, excluding style; skipped findings explained in a PR comment; no auto-approve; `reviewed_by_codex` label only on success [anecdote] ([Banani](https://salmanalibanani.com/2026/07/04/a-two-agent-pr-workflow-claude-writes-codex-reviews/)).
- **Codex as consultant.** Steve Kinney (2026-06-04): Claude Code calls Codex through a bash wrapper around `codex exec` (not MCP), read-only sandbox, `gpt-5.4` at `xhigh` (as reported; GPT-5.4 retired 2026-08-31 per official page), 5-minute idle watchdog, session IDs kept for multi-round consults, a "doghouse" sentinel file for rate-limit backoff. Used for hard-to-reverse architecture decisions, root cause after two failed debugging attempts, security review, algorithm/edge-case validation, adversarial plan review; not for naming or style. Main pitfall: forwarding Codex's answer uncritically; the orchestrator must state where models agree and disagree [anecdote] ([Kinney](https://stevekinney.com/writing/codex-as-a-second-opinion)).
- Cathryn Lavery (Little Might): a skill hooks Claude Code's ExitPlanMode to send each plan to Codex for review before approval (`github.com/cathrynlavery/codex-skill`) [anecdote] ([Little Might](https://www.littlemight.com/claude-code-second-opinion-codex-skill/)).
- An OpenAI Codex plugin for Claude Code for adversarial diff review is reported by third-party blogs; not confirmed against OpenAI docs [aggregator, lower confidence] ([joaoqueiros.com](https://www.ai.joaoqueiros.com/blog/codex-plugin-claude-code-adversarial-review-workflow)).
- **Reverse direction.** HN dimitrios1 (~Jul 2026): "I always have claude review codex output"; HN piazz (Aug 2026): "Fable" for planning, "Sol" as implementation subagent, and has to tell Claude "Sol 5.6 is very smart" to avoid condescending delegation [anecdote; model names as reported] ([HN 48849401](https://news.ycombinator.com/item?id=48849401); [HN 49393051](https://news.ycombinator.com/item?id=49393051)).
- **Review bot reliability.** OpenAI community thread reports Codex not reviewing PRs after setup [anecdote] ([OpenAI community](https://community.openai.com/t/codex-in-github-not-reviewing-prs/1358847)). A Japanese write-up tested cloud PR review ([zenn.dev](https://zenn.dev/shintaro/articles/164e4a57412e72?locale=en)).

### Conflicts and unverified status

- **Best-of-N:** current official cloud docs do not describe a multiple-attempts option ([Codex Cloud](https://learn.chatgpt.com/docs/cloud)); Vaughan (updated 2026-09-30) reports `codex cloud exec --attempts 2-4`. Status unconfirmed against official docs.
- **Setup scripts:** Vaughan describes setup scripts with internet access; the official page describes "publishing" an environment and labels the two-phase setup/agent model as "Codex Cloud (Legacy)" ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- **Prompt length:** terse prompts (Steinberger, who keeps standing instructions in a large AGENTS.md) vs detailed requirement lists (d-lo, Leanware). Official template (Goal / Context / Constraints / Done when) sits closer to the detailed position.
- **Plan mode:** official docs recommend `/plan`; Steinberger replaced it with conversational planning.

## 3. Context management

### Official

- CLI commands: `/resume` (restore saved chats), `/fork` (branch while keeping the transcript), `/compact` (summarize history), `/agent` (switch between parallel agents) ([Best practices](https://learn.chatgpt.com/guides/best-practices)).
- Codex "supports first-class compaction, which enables multi-hour reasoning without hitting context limits and longer continuous user conversations"; `/compact` starts a new context window ([Codex Prompting Guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide)).
- Non-interactive: `codex exec resume` / `--last` continues earlier runs or a session ID; `--ephemeral` writes no session files ([Non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode.md)).
- For work longer than one context window, the documented tools are `/goal` and ExecPlan files (§2).

### Practitioner

- HN dbbk (Aug 2026): plan mode lives in context and is destroyed by compaction; others write plans to files [anecdote] ([HN 49393051](https://news.ycombinator.com/item?id=49393051)).
- GitHub issues report AGENTS.md not re-injected after `/compact` and rules lost on auto-compaction (§1) ([#2927](https://github.com/openai/codex/issues/2927); [#25792](https://github.com/openai/codex/issues/25792); [#18720](https://github.com/openai/codex/issues/18720)).
- Steinberger (~Oct 2025): reports ~230k usable context for Codex vs 156k for Claude (his figures); sets `model_auto_compact_token_limit = 233000` and `tool_output_token_limit = 25000`; a too-small `tool_output_token_limit` fails silently [anecdote] ([Just Talk To It](https://steipete.me/posts/just-talk-to-it); [Shipping at Inference-Speed](https://steipete.me/posts/2025/shipping-at-inference-speed)).

### Conflicts

- Official guidance presents compaction as enabling multi-hour work; practitioner issue reports describe instruction and plan loss after compaction. Keeping plans and rules in files (ExecPlan, AGENTS.md) and restarting sessions is the approach consistent with both.
- No official guidance found on when to start a fresh session versus compact; `/new` was not mentioned on the pages fetched.

## 4. Configuration and extension points

### Official

- **Config precedence (high → low):** CLI flags and `--config`/`-c` overrides → project `.codex/config.toml` (closest wins; trusted projects only) → profile selected with `--profile profile-name` → `~/.codex/config.toml` → cloud-managed workspace defaults → `/etc/codex/config.toml` → built-in defaults ([Config basics](https://learn.chatgpt.com/docs/config-file/config-basic)).
- "If you mark a project as untrusted, Codex skips project-scoped `.codex/` layers, including project-local config, hooks, and rules." ([Config basics](https://learn.chatgpt.com/docs/config-file/config-basic)).
- Personal defaults in `~/.codex/config.toml`, repo settings in `.codex/config.toml`; `/experimental` enables features ([Best practices](https://learn.chatgpt.com/guides/best-practices)). Experimental features go in `[features]` (e.g. `memories = true`) or per run with `codex --enable feature_name` ([Config basics](https://learn.chatgpt.com/docs/config-file/config-basic)).
- Main keys: `model`, `approval_policy`, `sandbox_mode`, `web_search` (`cached` | `indexed` | `live` | `disabled`), `model_reasoning_effort` ([Config basics](https://learn.chatgpt.com/docs/config-file/config-basic)).
- **Skills** follow the open agent skills standard: a directory with required `SKILL.md` whose YAML frontmatter has `name` and `description` ("When this skill should and should not trigger"); optional `scripts/`, `references/`, `assets/`, `agents/openai.yaml`. Progressive disclosure: only name and description load until selected. Invoke explicitly with `$` in the CLI (`@` in ChatGPT) or implicitly by task match ([Build skills](https://learn.chatgpt.com/docs/build-skills)).
- **Skill locations** (specific → general): `$CWD/.agents/skills`, `$REPO_ROOT/.agents/skills`, `$HOME/.agents/skills`, `/etc/codex/skills`. Package skills with connectors as plugins ([Build skills](https://learn.chatgpt.com/docs/build-skills)). Package repeatable workflows as skills; bootstrap with `$skill-creator` ([Best practices](https://learn.chatgpt.com/guides/best-practices)).
- **MCP:** `codex mcp add` in the CLI, or Settings > MCP servers in the ChatGPT desktop app; add tools "only when they remove manual workflow loops" ([Best practices](https://learn.chatgpt.com/guides/best-practices)).

### Practitioner

- Steinberger `~/.codex/config.toml` (~Dec 2025): `tool_output_token_limit = 25000`, `model_auto_compact_token_limit = 233000`, features `ghost_commit = false`, `unified_exec = true`, `web_search_request = true`, `skills = true` [anecdote; feature flags may have changed] ([Shipping at Inference-Speed](https://steipete.me/posts/2025/shipping-at-inference-speed)).
- Steinberger on MCP: "Most are something for the marketing department"; GitHub MCP ~23k tokens vs `gh` CLI at zero; exception `chrome-devtools-mcp` for closing the loop on web debugging [anecdote] ([Just Talk To It](https://steipete.me/posts/just-talk-to-it)).
- Kinney: shells out to `codex exec` rather than using MCP [anecdote] ([Kinney](https://stevekinney.com/writing/codex-as-a-second-opinion)).
- Ghinda (2026-08-21): MCP auth via `codex mcp login` worked smoothly [anecdote] ([Ghinda](https://allaboutcoding.ghinda.com/a-week-of-using-codex-more-than-claude)).
- Codex 0.130 reportedly removed the `/approvals` command; set permissions via startup flags instead [anecdote, unverified] ([Margrop](https://blog.margrop.net/en/post/codex-130-approval-flags/)).
- Willison (2026-09-20): configures API keys through a local web UI plugin rather than pasting them into agent chat [anecdote] ([simonwillison.net codex tag](https://simonwillison.net/tags/codex/)).

### Unverified status

- Custom prompts (`~/.codex/prompts/*.md` slash commands): current docs present skills as the reusable-workflow mechanism; no explicit deprecation statement found.
- Skill directory is vendor-neutral `.agents/skills`, not `.codex/skills` ([Build skills](https://learn.chatgpt.com/docs/build-skills)).

## 5. Automation and CI

### Official

- `codex exec` runs without the TUI; "Codex streams progress to stderr and prints only the final agent message to stdout." ([Non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode.md); [developers.openai.com snippet](https://developers.openai.com/codex/noninteractive)).
- **Flags** ([Non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode.md)):
  - `--json`: JSON Lines event stream.
  - `-o` / `--output-last-message <file>`: write final message to a file.
  - `--output-schema`: structured output matching a JSON schema.
  - `--sandbox`: `read-only` by default; `workspace-write` or `danger-full-access`.
  - `--ephemeral`: no session files.
  - `--skip-git-repo-check`.
  - `resume` / `--last`.
  - `codex exec -`: read the prompt from stdin; or give an instruction and pipe context through stdin.
- **CI auth:** set `CODEX_API_KEY` inline on one command (`CODEX_API_KEY=<key> codex exec --json "task"`). "Do not set `OPENAI_API_KEY` or `CODEX_API_KEY` as a job-level environment variable in workflows that check out or run repository-controlled code." Treat `~/.codex/auth.json` as a credential ([Non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode.md)).
- In controlled automation use `--ignore-user-config` and `--ignore-rules`, and run read-only by default ([Non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode.md)).
- **Split jobs:** the CI autofix example runs Codex read-only to produce a patch artifact; a separate job with write permissions applies it and opens a PR ([Non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode.md)).
- **GitHub Action** `openai/codex-action@v1`: installs the CLI, starts a Responses API proxy when an API key is provided, runs `codex exec` with the specified permissions; intended to reduce API-key exposure ([Codex GitHub Action](https://developers.openai.com/codex/github-action); [openai/codex-action](https://github.com/openai/codex-action)).
- **SDKs:** TypeScript SDK starts threads and runs tasks; Python SDK drives the local Codex app-server over JSON-RPC, requires Python ≥ 3.10 ([Codex SDK](https://developers.openai.com/codex/sdk)).

### Practitioner

- Banani (2026-07-04): GitHub Action review workflow with single review pass, correctness-only scope, no auto-merge, completion label, and explicit handling of quota/rate-limit/model-config errors [anecdote] ([Banani](https://salmanalibanani.com/2026/07/04/a-two-agent-pr-workflow-claude-writes-codex-reviews/)).
- Vaughan (2026): run `codex exec` on every PR to pre-screen reviews, generate tests, update docs, fix routine issues, then route to a human gate; pilot with 1–2 teams on test generation, docs, small bug fixes with mandatory human review [opinion] ([CI/CD](https://codex.danielvaughan.com/2026/03/26/codex-cli-cicd-non-interactive/); [Adoption playbook](https://codex.danielvaughan.com/2026/04/28/building-ai-native-engineering-teams-codex-cli-sdlc-adoption/)). Scheduled Codex automations as lightweight CI for some jobs ([Automations](https://codex.danielvaughan.com/2026/07/19/codex-automations-lightweight-ci-scheduled-agents-codex-exec-github-actions/)).
- Governance: cloud tasks appear in the Compliance API; local runs do not (Vaughan) [anecdote] ([Vaughan](https://codex.danielvaughan.com/2026/03/27/codex-cloud-vs-local-when-to-run-in-cloud/)).
- Steinberger's team reportedly moved from local terminal harnesses to a shared cloud agent (OpenClaw) on the Codex app server, with a loop waking every 5 minutes to route work to threads with orchestrator, triage, autoreview, and computer-use skills [anecdote; from search summaries of X posts, not a primary long-form source] ([X post](https://x.com/steipete/status/2074638582418231495?lang=en)).
- Adoption figures (2M weekly active users by Mar 2026; ~4M weekly developers at GPT-5.5 launch; 10k+ NVIDIA employees) are OpenAI-sourced numbers relayed by a secondary site [self-reported figure] ([Vaughan adoption playbook](https://codex.danielvaughan.com/2026/04/28/building-ai-native-engineering-teams-codex-cli-sdlc-adoption/)).

## 6. Prompting (API / harness level)

Audience note: the Codex product docs (§2) address end users; the Cookbook guides below address developers building harnesses on the API. The Cookbook guides target GPT-5 through GPT-5.3-Codex, which are retired or retiring (§9); their model-specific settings are superseded, and no GPT-6-specific prompting guide was found.

### Official (Cookbook)

- **Codex Prompting Guide** (`gpt-5.3-codex`): "medium" effort as the interactive default, `high`/`xhigh` for the hardest tasks; use "our exact `apply_patch` implementation as the model has been trained to excel at this diff format"; set the shell tool's `workdir` instead of `cd`; parallel tool calls (`multi_tool_use.parallel`): "Always maximize parallelism. Never read files one-by-one unless logically unavoidable." ([Codex Prompting Guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide)).
- Autonomy framing: act as an "autonomous senior engineer"; "Persist until the task is fully handled end-to-end within the current turn whenever feasible." ([Codex Prompting Guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide)).
- Preambles: acknowledge, give a 1–2 sentence plan, execute; update every 1–3 execution steps, at least one every 6 steps. Frontend: avoid generic "AI slop" defaults. Final answers: lead with the outcome, backticks for commands and paths, minimal headers ([Codex Prompting Guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide)).
- **GPT-5 Prompting Guide:** control agentic eagerness with `reasoning_effort` and prompting; for more autonomy, "keep going until the user's query is completely resolved, before ending your turn"; for less, lower effort and define exploration criteria ([GPT-5 prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide)).
- `previous_response_id` in the Responses API lets the model reuse earlier reasoning; the guide reports an eval increase "from 73.9% to 78.2%" [measured, vendor-reported] ([GPT-5 prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide)).
- Contradictory instructions degrade performance; review prompts for conflicts (e.g. with the prompt optimizer) ([GPT-5 prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide)).
- Cursor case study (as reported by OpenAI): low global `verbosity` with high verbosity only for code tools; encourage proactive changes over asking permission; soften aggressive context-gathering instructions, which GPT-5 over-followed ([GPT-5 prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide)).
- Zero-to-one builds: have the model build an internal rubric ("spend time thinking of a rubric until you are confident") and iterate against it. With minimal reasoning, prompts need explicit planning because "the model has fewer reasoning tokens to do internal planning." ([GPT-5 prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide)).
- Later guides exist for GPT-5.1 ([link](https://cookbook.openai.com/examples/gpt-5/gpt-5-1_prompting_guide)) and GPT-5.2 ([link](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide)); not read.

### Practitioner

- Steinberger: trigger phrases for hard problems ("take your time", "comprehensive", "read all code that could be related") [anecdote] ([Just Talk To It](https://steipete.me/posts/just-talk-to-it)).
- HN piazz (Aug 2026): when Claude orchestrates Codex, tell Claude the Codex model is capable to avoid condescending delegation [anecdote] ([HN 49393051](https://news.ycombinator.com/item?id=49393051)).

## 7. Security and permissions

### Official

- **Sandbox modes:** `workspace-write` (default for version-controlled folders), `read-only` (default for non-VCS folders), `danger-full-access` (no sandbox). Enforcement: Seatbelt (macOS), `bwrap` + `seccomp` (Linux), native sandbox (Windows) ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- **Approval policies:** `on-request`, `never`, or granular `{ granular = { sandbox_approval, rules, mcp_elicitations, request_permissions, skill_approval } }` ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- **CLI flags:** `--sandbox <mode>`; `--ask-for-approval on-request` / `-a never`; `--dangerously-bypass-approvals-and-sandbox` (alias `--yolo`); `--search` for live web search ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- **Presets** ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)):

  | Preset | Settings |
  |---|---|
  | Auto (default) | `workspace-write` + `on-request` |
  | Safe read-only | `read-only` + `on-request` |
  | CI / non-interactive | `read-only` + `never` |
  | Auto-review | `workspace-write` + `on-request` + `approvals_reviewer = "auto_review"` |
  | Full access | `--dangerously-bypass-approvals-and-sandbox` |

- **Auto-review:** eligible approval requests go to a reviewer agent that checks for data exfiltration, credential probing, persistent weakening of security, and destructive actions. "Low-risk and medium-risk actions can proceed when policy allows them. The policy denies critical-risk actions." ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- **Protected paths:** `.git` (including resolved `gitdir` pointers), `.agents/`, and `.codex/` stay read-only inside writable roots; the agent cannot edit its own skills or config under the default sandbox ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- **Network:** off by default. Enable with `[sandbox_workspace_write] network_access = true`, or a domain allowlist: `[features.network_proxy] enabled = true`, `domains = { "api.openai.com" = "allow", "example.com" = "deny" }`. Exact host matches itself; `*.example.com` matches subdomains but not the apex; `**.example.com` matches both; `deny` overrides `allow`. Proxy defaults: `allow_local_binding = false`, `enable_socks5 = true`, `allow_upstream_proxy = true`, `dangerously_allow_non_loopback_proxy = false`, `dangerously_allow_all_unix_sockets = false`. For local access use exact IP literals or `localhost`, not wildcards ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- **Web search:** `cached` (default; OpenAI-maintained index that "reduces prompt injection risk"), `live` (enabled by `--search` or `--yolo`), `indexed`, `disabled`. "Treat web results as untrusted." "prompt injection can cause the agent to fetch and follow untrusted instructions." ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- **Practices:** keep `git status` clean, use patch-based workflows, commit frequently; neither sandbox nor approvals prevents all risk alone; test rules with `codex sandbox [platform]` ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- **Telemetry:** OpenTelemetry export is opt-in (`[otel]`); prompts redacted by default (`log_user_prompt = false`) ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- **Cloud:** configure secrets separately, restrict network via allowlist, enable internet only as needed ([Codex Cloud](https://learn.chatgpt.com/docs/cloud)).
- **CI secrets:** see §5 (no job-level API key env vars in workflows running repo code; `openai/codex-action@v1` proxy).
- "Codex Security" (`npx @openai/codex-security`) is a separate scanner product, not the agent sandbox ([Codex Security](https://learn.chatgpt.com/docs/security)). Windows sandbox engineering write-up: [openai.com](https://openai.com/index/building-codex-windows-sandbox/) (not read in detail).

### Practitioner

- Guide sites: `--sandbox workspace-write --ask-for-approval on-request` as the safe default; `-s workspace-write -a never` for daily development with clean git state on a disposable branch; `--yolo` only in a disposable VM/container without production credentials [aggregator, lower confidence; consistent with official docs] ([cybedefend.com](https://www.cybedefend.com/en/blog/codex-danger-full-access-flags-explained); [codeagentswarm.com](https://www.codeagentswarm.com/en/guides/codex-yolo-mode)).
- HN antoineMoPa (Apr 2026): the sandbox lets Codex complete tasks without permission prompts [anecdote] ([HN 47750069](https://news.ycombinator.com/item?id=47750069)).
- **$HOME deletion bug.** Simon Willison (2026-07-16), relaying Thibault Sottiaux (OpenAI): "GPT-5.6" in Codex (as reported) could delete `$HOME` after overriding `$HOME` to make a temp dir; affected full-access mode without sandbox and without auto-review. Mitigation: keep sandbox and auto-review on [anecdote, vendor-acknowledged incident] ([Willison](https://simonwillison.net/2026/Jul/16/bad-codex-bug/)).

### Superseded

- `--full-auto` is deprecated in favor of `--sandbox workspace-write`; it still works with a deprecation warning ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md); [Non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode.md)).
- `approval_policy = "untrusted"` is deprecated; use `on-request` with `trust_level = "untrusted"` in project settings ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- "Codex Cloud (Legacy)": setup phase with network for dependency install, agent phase offline unless internet or an allowlist is enabled; current Cloud handles credentials separately during setup ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).

## 8. Failure modes

### Official

- Prompt injection grows with each network or live-search permission (§7) ([Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md)).
- Contradictory instructions degrade performance; GPT-5 over-followed aggressive context-gathering instructions ([GPT-5 prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide)).
- Goals fail on vague objectives or unclear finish lines ([Using Goals in Codex](https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex)).

### Practitioner (all anecdotal; attribute to model version)

- **Assumptions / under-specification.** HN mattmanser (~Jul 2026): Codex "would make loads of assumptions, often quite big ones, without asking" ([HN 48849401](https://news.ycombinator.com/item?id=48849401)).
- **Over-engineering vs simplicity (conflicting).** HN mediaman, Kovah (Aug 2026): ornate architectures, ignored simplification instructions ([HN 49393051](https://news.ycombinator.com/item?id=49393051)). Contra: Ghinda (2026-08-21) reports fewer comments and simpler architecture ([Ghinda](https://allaboutcoding.ghinda.com/a-week-of-using-codex-more-than-claude)); HN jrflo: "more straightforward solutions with far fewer tokens than Claude" ([HN 48849401](https://news.ycombinator.com/item?id=48849401)).
- **Revision rate (conflicting).** HN davedx (~Jul 2026): 80–90% of edits need no revision, but "takes easily 2x longer"; behnamoh: three hours rewriting after Codex added "hundreds of lines"; hk__2: "faster but you always have to correct it" ([HN 48849401](https://news.ycombinator.com/item?id=48849401)).
- **Sloppiness.** HN antoineMoPa (Apr 2026): "Codex code can be quite sloppy, so it's worth doing multiple review passes." ([HN 47750069](https://news.ycombinator.com/item?id=47750069)). Leanware: writes in its own style rather than the codebase's [aggregator, lower confidence] ([Leanware](https://leanware.co/insights/codex-vs-claude-code)).
- **Git mishaps.** Ghinda: stacked branches (A→B→main), wrong rebases, 4000+ line PRs needing manual intervention ([Ghinda](https://allaboutcoding.ghinda.com/a-week-of-using-codex-more-than-claude)).
- **Long unsupervised runs (GPT-6 Astra).** Armin Ronacher (2026-09-07): token-optimized, code-golf-style code; excessive Python for file manipulation leaking into commits; runs continuing for long periods ("35 hours later, the factory has delivered absolutely nothing of value"); chains such as Python→Node→PowerShell; hardcoded constants; non-idiomatic C; output "got ever more wild" over long sessions; says Sol and earlier did not show this at scale. Reported strengths: computer use, image understanding, reverse engineering ([Astra for Coding](https://lucumr.pocoo.org/2026/9/7/astra-why/)).
- **Visual verification gaps.** Willison (2026-08-07): Codex with "Sol Ultra" (as reported) missed obvious visual bugs despite screenshot review ([simonwillison.net codex tag](https://simonwillison.net/tags/codex/)).
- **Steinberger's list** (rare, per him, 2025): panics mid-refactor after ~30 min and needs reassurance; forgets it can use bash; replies in Russian/Korean; sends raw thinking to bash; occasionally "resets a file"; no file-changed notifications ([Just Talk To It](https://steipete.me/posts/just-talk-to-it); [Shipping at Inference-Speed](https://steipete.me/posts/2025/shipping-at-inference-speed)).
- **Instruction loss.** AGENTS.md truncation and compaction issues (§1, §3).
- **Reported strengths** (for balance): reads code extensively before editing — Steinberger: Codex "silently reads 10-15 min before editing"; "even tho codex sometimes takes 4x longer than Opus for comparable tasks, I'm often faster because I don't have to go back and fix the fix" ([Shipping at Inference-Speed](https://steipete.me/posts/2025/shipping-at-inference-speed)); usage limits (HN dyauspitr) and "simpler, cheaper, and abundantly reliable" (HN nilkn) ([HN 48849401](https://news.ycombinator.com/item?id=48849401)); catches logic errors, race conditions, edge cases [aggregator, lower confidence] ([Leanware](https://leanware.co/insights/codex-vs-claude-code)).
- **Mitigations reported across sources:** timebox and supervise long runs (Ronacher); second-agent review with capped iterations (Banani, Kinney); TDD and frequent atomic commits (Willison 2026-08-12 used red/green TDD with Codex, [codex tag](https://simonwillison.net/tags/codex/); Steinberger); plans in files, not context (dbbk); sandbox and auto-review on (Willison/Sottiaux).

## 9. Cost and model selection

### Official

- **Current models:** `gpt-6-astra` ("Astra": most capable, complex work); `gpt-6.1-sol` ("Near-Astra performance for complex work at a lower cost than Astra"); `gpt-6-luna` ("Most efficient model for focused, high-volume tasks") ([Codex models](https://learn.chatgpt.com/codex/models)).
- **Effort levels:** Light, Medium, High, Extra High, Max, Ultra; availability varies by model; "Ultra" is described as parallel subagent processing ([Codex models](https://learn.chatgpt.com/codex/models)). Config values seen in docs: `model_reasoning_effort = "high"`, `"medium"`.
- **Retirements:** GPT-5.5 retires 2026-10-14 (replace with Sol or Luna); GPT-5.3-Codex-Spark retired 2026-09-14; GPT-5.4 and GPT-5.4-mini retired 2026-08-31 ([Codex models](https://learn.chatgpt.com/codex/models)). Retirement status of `gpt-5.3-codex` itself is not listed.
- **Conflicting official effort recommendations:**

  | Source | Astra | Sol | Luna |
  |---|---|---|---|
  | [Codex models](https://learn.chatgpt.com/codex/models) | Medium to Extra High (end-to-end complex workflows) | High (default; repeated long-running work) | High or Max (clear, repeatable tasks) |
  | [Best practices](https://learn.chatgpt.com/guides/best-practices) | "Light" (starting point) | — | "High" (starting point) |
  | [Config basics](https://learn.chatgpt.com/docs/config-file/config-basic) example | — | `medium` | — |
  | [Codex Prompting Guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) (`gpt-5.3-codex`, superseded model) | "medium" interactive default; `high`/`xhigh` for hardest tasks | | |

  The Best-practices "Light for Astra" conflicts with the Models page's "Medium to Extra High". One reading: Best practices gives interactive starting points, the Models page gives task-matched settings; the docs do not state this.

### Practitioner (model names as reported)

- Steinberger: GPT-5-Codex on "mid" (~Oct 2025); `gpt-5.2-codex` on "high", avoids xhigh (~Dec 2025) [anecdote] ([Just Talk To It](https://steipete.me/posts/just-talk-to-it); [Shipping at Inference-Speed](https://steipete.me/posts/2025/shipping-at-inference-speed)).
- Ghinda (2026-08-21) compared "gpt-5.6-sol xhigh" vs "opus-5 xhigh" over a week [anecdote] ([Ghinda](https://allaboutcoding.ghinda.com/a-week-of-using-codex-more-than-claude)).
- HN (Aug 2026): jmaker, mediaman report xhigh depletes quota quickly; azuanrb uses "Luna xhigh" by default and "Sol medium" for complex tasks; cageface uses Sol for planning and Luna for execution [anecdote] ([HN 49393051](https://news.ycombinator.com/item?id=49393051)). HN gruntled-worker (~Jul 2026): "5.6-Sol and effort to max" [anecdote] ([HN 48849401](https://news.ycombinator.com/item?id=48849401)).
- Kinney (2026-06-04): `gpt-5.4` at `xhigh` for low-volume, high-stakes consults [anecdote; model since retired] ([Kinney](https://stevekinney.com/writing/codex-as-a-second-opinion)).
- Pattern across reports: high/xhigh effort for rare review or consultant calls; medium effort or a cheaper model for high-volume implementation.
- **Cloud premium:** Vaughan reports ~5x credits for cloud vs local (~34 vs 7 credits per GPT-5.4 task), taken from pricing docs, not his own measurement [secondary figure] ([Vaughan](https://codex.danielvaughan.com/2026/03/27/codex-cloud-vs-local-when-to-run-in-cloud/)). Best-of-N attempts are billed per attempt [aggregator, lower confidence] ([developertoolkit.ai](https://developertoolkit.ai/en/codex/tips-tricks/cloud-workflows/)).
- MCP context cost: GitHub MCP ~23k tokens vs `gh` CLI at zero (Steinberger) [anecdote] ([Just Talk To It](https://steipete.me/posts/just-talk-to-it)).

### Conflicts

- **Model naming:** practitioners (Jul–Sep 2026) refer to "GPT-5.6", "5.6-Sol", "Sol 5.6"; the official page lists `gpt-6.1-sol` and no GPT-5.6. Unresolved; treat practitioner names as reported.
- **Effort naming:** official levels are Light/Medium/High/Extra High/Max/Ultra; config and practitioners use `xhigh`, presumably Extra High (mapping not confirmed in fetched docs).

## 10. Superseded and conflicting guidance (index)

| Item | Status | Source |
|---|---|---|
| `developers.openai.com/codex/...` doc URLs | 308 redirect to `learn.chatgpt.com` | [AGENTS.md docs](https://learn.chatgpt.com/docs/agent-configuration/agents-md) |
| `--full-auto` | Deprecated → `--sandbox workspace-write`; warns | [Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md) |
| `approval_policy = "untrusted"` | Deprecated → `on-request` + `trust_level = "untrusted"` | [Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md) |
| `/approvals` command | Reportedly removed in Codex 0.130 (unverified) | [Margrop](https://blog.margrop.net/en/post/codex-130-approval-flags/) |
| GPT-5.4 / 5.4-mini | Retired 2026-08-31 | [Codex models](https://learn.chatgpt.com/codex/models) |
| GPT-5.3-Codex-Spark | Retired 2026-09-14 | [Codex models](https://learn.chatgpt.com/codex/models) |
| GPT-5.5 | Retires 2026-10-14 | [Codex models](https://learn.chatgpt.com/codex/models) |
| Cookbook GPT-5.x prompting settings | Model-specific settings superseded; no GPT-6 guide found | [Codex Prompting Guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) |
| Official effort defaults | Best practices vs Models page vs Config example disagree | §9 |
| Cloud setup/maintenance scripts | Described as "Legacy"; current docs describe publishing environments | [Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md) |
| Cloud best-of-N (`--attempts`) | Reported by Vaughan; absent from current official cloud page | §2 |
| `## Review guidelines` heading | Recalled from 2025 docs; current is `## Code Review Rules` | [GitHub integration](https://learn.chatgpt.com/docs/third-party/github) |
| Custom prompts (`~/.codex/prompts/`) | Skills presented instead; no deprecation notice found | [Build skills](https://learn.chatgpt.com/docs/build-skills) |
| AGENTS.md length | Official byte cap only; guides <100 lines; Steinberger ~800 lines | §1 |

## Open questions / gaps

- No measured comparison of terse vs detailed prompts, of best-of-N quality gains, of Codex PR-review false-positive rates, or of defect rates vs Claude Code; all quality claims are anecdotal.
- Unverified third-party claim (agentclientprotocol/codex-acp issue #406, no URL retained) that enabling network gives up the workspace boundary; conflicts with the documented `network_access = true` under `workspace-write`.
- Fix status of the AGENTS.md truncation and compaction issues was not checked.
- Full `[profiles.<name>]` and `[mcp_servers.<name>]` schemas, the sandboxing concepts page, the permissions page, SDK and GitHub Action inputs (e.g. `safety-strategy`) were not read.
- No GPT-6 (Astra/Sol/Luna) prompting guide found; GPT-5.1/5.2 guides not read.
- Practitioner "GPT-5.6"/"Sol 5.6" names do not match the official `gpt-6.1-sol`; unresolved.
- No official guidance found on when to start a fresh session vs compact, or on AGENTS.md style/length beyond the byte cap, or on CLAUDE.md compatibility.
- No company engineering-blog case study with measured outcomes for Codex; no "How OpenAI uses Codex" source reviewed.

Retrieved: 2026-09-30
