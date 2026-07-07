# smart-commit

A [Claude Code](https://claude.com/claude-code) plugin that turns a pile of pending git changes
into clean, logically-grouped [Conventional Commits](https://www.conventionalcommits.org/) — with
you in control of every commit before it lands.

It splits the work across three models so the expensive reasoning only happens where it's needed:

| Piece | Model | Job |
|-------|-------|-----|
| `smart-commit` skill | main loop | orchestrates the flow, talks to you |
| `smart-committer` agent | **sonnet** (medium effort) | reads the diffs, proposes the grouping |
| `commit-runner` agent | **sonnet** | dumb executor — stages + commits the approved plan, runs the push |

## Flow

1. You run `/smart-commit` (or just say "commit everything").
2. **smart-committer** reads all pending diffs and proposes a numbered list of commits (message + files).
3. The skill shows you the proposal and asks what you want — commit all, drop some, reword, regroup.
4. On your OK, **commit-runner** stages and commits exactly the approved plan.
5. The skill asks "Push to remote?" — if yes, **commit-runner** runs the push.

Nothing is committed or pushed without your explicit go-ahead.

## Install

> **Recommended: vendor into the project, do NOT `/plugin install`.**
> Installing as a plugin namespaces everything as `plugin:name`. Because this plugin AND its skill are
> both called `smart-commit`, you get the doubled, ugly `/smart-commit:smart-commit` — and the sub-agents
> become `smart-commit:smart-committer` / `smart-commit:commit-runner`. Copying the files straight into a
> project's `.claude/` registers them as plain project skills/agents, so the names stay **bare**
> (`/smart-commit`, `smart-committer`, `commit-runner`) and they get committed + shared via the repo.

### Option A — vendor into your project (recommended, bare names)

From a clone of this repo, copy the pieces into the target repo's `.claude/`:

```bash
git clone https://github.com/Glutoblop/smart-commit.git
cd smart-commit
PROJ=/path/to/your/project

mkdir -p "$PROJ/.claude/skills" "$PROJ/.claude/agents"
cp -r skills/smart-commit                             "$PROJ/.claude/skills/"
cp agents/smart-committer.md agents/commit-runner.md  "$PROJ/.claude/agents/"
```

The skill refers to its agents by **bare** `subagent_type` (`smart-committer`, `commit-runner`), so no
path edits are needed — as project agents they resolve under exactly those names.

To make them travel with clones, ensure the project's `.gitignore` doesn't swallow `.claude/`. A common
pattern — ignore the dir but whitelist the shareable parts:

```gitignore
/.claude/*
!/.claude/skills/
!/.claude/agents/
!/.claude/commands/
!/.claude/hooks/
!/.claude/settings.json
```

Then `/smart-commit` is available **in that project**, bare, versioned. (Optional: to auto-suggest the
skill when you type "commit", add a `UserPromptSubmit` hook — see the project's `.claude/hooks/`.)

For **every** project instead of one, copy the same files into `~/.claude/skills/` and `~/.claude/agents/`
(user scope, unversioned) — still bare names, since user-scope skills aren't namespaced either.

---

### Plugin install (namespaces the names — not recommended)

> **Install from ONE source only.** Don't add both this repo and the `gluto` bundle — two
> marketplaces offering `smart-commit` make the name ambiguous, and Claude Code then forces the
> namespaced `/smart-commit:smart-commit` instead of bare `/smart-commit`. Pick A **or** B.

#### Marketplace B1 — from this repo (self-contained)

This repo ships its own `.claude-plugin/marketplace.json`, so it installs directly from its URL:

```
/plugin marketplace add Glutoblop/smart-commit
/plugin install smart-commit@smart-commit
```

Format is `<plugin>@<marketplace>`; here both are `smart-commit`.

#### Marketplace B2 — from the gluto bundle (several plugins in one marketplace)

```
/plugin marketplace add Glutoblop/gluto-claude
/plugin install smart-commit@gluto
```

Either way, restart Claude Code (or reload plugins). In practice Claude Code namespaces plugin content,
so the command commonly shows up as **`/smart-commit:smart-commit`** (plugin name + skill name) and the
agents as `smart-commit:smart-committer` / `smart-commit:commit-runner`. If you want bare names, use the
vendor option above instead.

## Requirements

- Claude Code with the `sonnet` model available to your account.
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
  commit-runner.md    # sonnet — runs commits + push
```

## Notes

- Commits never include a `Co-Authored-By` line or any mention of AI.
- The runner uses explicit file paths only — never `git add .` / `git add -A` — and never
  `--no-verify`, force push, or amend.
- Merge-to-main was intentionally left out; this plugin does commit → push only.

## License

See [LICENSE](LICENSE).
