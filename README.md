# Claude Fable Sonnet Hands

Claude Code skill for a safer orchestration workflow:

- **Fable** = orchestrator / head / decisions / specs
- **Sonnet** = hands: scout, executor, verifier, final review
- **Codex / Orca / Haiku** = disabled

The skill command is:

```text
/fable-sonnet-hands
```

## What it does

This skill keeps Fable focused on judgment and planning, while Sonnet subagents perform the operational work:

1. scout the repo and current diff;
2. write or update a self-contained spec;
3. dispatch a Sonnet executor;
4. verify with a fresh Sonnet verifier;
5. run a final Sonnet review;
6. report a short pipeline status.

It is useful when you use Claude Code as a VS Code extension and do not have Codex Companion, openai-codex plugin, Orca, Factory or external worker terminals configured.

## Install as a Claude Code marketplace plugin

In Claude Code:

```text
/plugins
```

Add marketplace:

```text
mioku50/claude-fable-sonnet-hands
```

Install the plugin, then start a new Claude Code session and run:

```text
/fable-sonnet-hands Continue the current task. Fable is the head; all hands are Sonnet. Do not use Codex, Orca or Haiku.
```

## Manual install into one project

From a local clone:

```bash
mkdir -p .claude/skills
cp -R skills/fable-sonnet-hands .claude/skills/
```

Or copy this folder into your project:

```text
.claude/skills/fable-sonnet-hands/SKILL.md
```

Then reload VS Code and call:

```text
/fable-sonnet-hands
```

## Recommended project setting

To force subagents to Sonnet inside a project, add:

```json
{
  "env": {
    "CLAUDE_CODE_SUBAGENT_MODEL": "sonnet"
  }
}
```

at:

```text
.claude/settings.local.json
```

## License

MIT
