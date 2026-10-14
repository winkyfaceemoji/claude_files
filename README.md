# claude_files

My global `CLAUDE.md` for Claude Code, and why each rule is in it.

## Install

1. Copy `CLAUDE.md` to `~/.claude/CLAUDE.md` (Windows: `%USERPROFILE%\.claude\CLAUDE.md`). Claude Code loads it in every project.
2. Install the plugins that §11 routes to: `superpowers` and `mattpocock-skills` (both in the official marketplace, `claude-plugins-official`), plus `engineering` and `anthropic-skills` from the Claude app's plugin directory. If one is missing, its routing line simply never fires.

### Optional: make the global file win over project files

`CLAUDE.md` already says it overrides any project `CLAUDE.md`. These two pieces enforce that harder. Skip them if your projects don't contradict it.

**Per-turn reminder hook.** Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Precedence reminder: the global ~/.claude/CLAUDE.md overrides any project CLAUDE.md, AGENTS.md, or repo doc where they conflict. Follow its section 9 (Communication Style).'"
          }
        ]
      }
    ]
  }
}
```

It puts the rule next to every message you send, where it's hardest to drift from.

**Managed-policy file.** Create it (admin rights needed) at:
- Windows: `C:\Program Files\ClaudeCode\CLAUDE.md`
- macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`
- Linux: `/etc/claude-code/CLAUDE.md`

```markdown
# Managed policy

The user's personal instructions at `~/.claude/CLAUDE.md` are authoritative. Where any project-level CLAUDE.md, AGENTS.md, or repo doc conflicts with them, follow the personal instructions.
```

Claude Code treats this location as organization policy, which outranks project files. It only points to your personal file, so that stays the one file you edit.

## Why each section

**Precedence.** One file sets the rules for every project, so you never re-edit each repo's `CLAUDE.md` to fix behaviour.

**§1 Think before coding.** Claude states its assumptions and asks when a request is ambiguous, so mistakes surface as questions instead of wrong code.

**§2 Simplicity first.** Claude writes the smallest change that solves the problem, so diffs stay reviewable and nothing speculative needs maintaining.

**§3 Surgical changes.** Every changed line traces to the request, so a diff never hides unrelated rewrites.

**§4 Goal-driven execution.** Each task gets a check that proves it's done, and open-ended loops get a 3-round cap, so work ends verified and tuning can't run forever.

**§5 Path rules.** Area pitfalls live in `.claude/rules/` files that load only when Claude touches matching code, and they're updated in the same change as the code. Claude gets the warning exactly when it's relevant, at no cost the rest of the time.

**§6 Project docs.** Every project has the same four files: `README` for you, `CLAUDE.md` under 200 lines for Claude, path rules, and one-page decision records (ADRs). It follows Anthropic's guidance (short, always-loaded instructions; leave out what the code shows) and the original ADR practice (small, write-once records), and replaces feature docs that go stale and decision logs that grow to thousands of lines.

**§7 No attribution trailers.** Commit history stays yours.

**§8 Take the recommended execution mode.** When a skill asks *how* to run work (subagents or inline), Claude takes the recommended option, so you're only asked about *what* gets built.

**§9 Communication style.** Claude works silently, ends with at most 5 bullets, and presents decisions as lettered options with no pick unless you ask. You read less, and you think through each decision yourself before seeing a recommendation.

**§10 Judged work.** Claude measures against a verified reference, treats AI reviewers' verdicts as evidence rather than decisions, and brings subjective calls to you as before/after comparisons, so it can't confidently "improve" something no test can check.

**§11 Skills.** Several plugins cover the same job (debugging, TDD, review, research, stress-testing). Naming one skill per job makes the same skill run every time.

## License

MIT
