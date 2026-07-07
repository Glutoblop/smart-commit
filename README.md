# smart-commit

A [Claude Code](https://claude.com/claude-code) plugin that turns a pile of pending git changes
into clean, logically-grouped [Conventional Commits](https://www.conventionalcommits.org/) — with
you in control of every commit before it lands.

It splits the work across three models so the expensive reasoning only happens where it's needed:

| Piece | Model | Job |
|-------|-------|-----|
| `smart-commit` skill | main loop | orchestrates the flow, talks to you |
| `smart-committer` agent | **sonnet** (medium effort) | reads the diffs, proposes the grouping |
| `commit-runner` agent | **haiku** | dumb executor — stages + commits the approved plan, runs the push |

## Flow

1. You run `/smart-commit` (or just say "commit everything").
2. **smart-committer** reads all pending diffs and proposes a numbered list of commits (message + files).
3. The skill shows you the proposal and asks what you want — commit all, drop some, reword, regroup.
4. On your OK, **commit-runner** stages and commits exactly the approved plan.
5. The skill asks "Push to remote?" — if yes, **commit-runner** runs the push.

Nothing is committed or pushed without your explicit go-ahead.

## Install

### Option A — as a plugin (recommended)

This plugin is distributed through the **`gluto`** marketplace
([Glutoblop/gluto-claude](https://github.com/Glutoblop/gluto-claude)). Add the marketplace, then install:

```
/plugin marketplace add Glutoblop/gluto-claude
/plugin install smart-commit@gluto
```

Restart Claude Code (or reload plugins) and `/smart-commit` is available in every project. The
plugin ships the skill and both sub-agents together — no extra setup.

### Option B — manual (copy into your `~/.claude`)

If you don't want to use the plugin system, clone the repo and copy the pieces into your Claude
config directory:

```bash
git clone https://github.com/Glutoblop/smart-commit.git
cd smart-commit

# user-level (available in every project):
cp -r skills/smart-commit  ~/.claude/skills/
cp agents/smart-committer.md agents/commit-runner.md  ~/.claude/agents/
```

For a single project instead, copy into that repo's `.claude/skills/` and `.claude/agents/`.

## Requirements

- Claude Code with the `sonnet` and `haiku` models available to your account.
- A git repo with pending changes (the skill is a no-op on a clean tree).

## Repo layout

```
.claude-plugin/
  plugin.json         # plugin manifest (listed by the gluto marketplace)
skills/
  smart-commit/
    SKILL.md          # the /smart-commit orchestrator
agents/
  smart-committer.md  # sonnet — proposes the grouping
  commit-runner.md    # haiku  — runs commits + push
```

## Notes

- Commits never include a `Co-Authored-By` line or any mention of AI.
- The runner uses explicit file paths only — never `git add .` / `git add -A` — and never
  `--no-verify`, force push, or amend.
- Merge-to-main was intentionally left out; this plugin does commit → push only.

## License

See [LICENSE](LICENSE).
