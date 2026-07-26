# OpenCode Workflow Starter

This repo is now a small, reusable starter for **OpenCode** workflows.

It gives you:

- a primary `workflow` agent,
- four focused subagents,
- a few ready-to-use slash commands,
- a copy-pasteable setup you can drop into a new project.

The goal is simple:

> make OpenCode feel like a repeatable engineering workflow, not a one-off chat prompt.

## What Is In Here

These files are the starter:

- [opencode.jsonc](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/opencode.jsonc)
- [.opencode/WORKFLOW.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/WORKFLOW.md)
- [.opencode/prompts/workflow.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/prompts/workflow.md)
- [.opencode/prompts/feature-planner.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/prompts/feature-planner.md)
- [.opencode/prompts/feature-builder.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/prompts/feature-builder.md)
- [.opencode/prompts/feature-tester.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/prompts/feature-tester.md)
- [.opencode/prompts/feature-reviewer.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/prompts/feature-reviewer.md)
- [.opencode/commands/feature.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/commands/feature.md)
- [.opencode/commands/fix.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/commands/fix.md)
- [.opencode/commands/spec.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/commands/spec.md)
- [.opencode/commands/review-diff.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/commands/review-diff.md)
- [.opencode/commands/test-target.md](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode/commands/test-target.md)

## How The Workflow Works

The sequence is:

```text
/feature or /fix
  -> workflow
  -> feature-planner
  -> feature-builder
  -> feature-tester
  -> feature-reviewer
```

Each agent has one job:

| Agent | Role | Why it exists |
|---|---|---|
| `workflow` | Orchestrates the sequence | Keeps the process consistent |
| `feature-planner` | Makes a small plan | Prevents premature coding |
| `feature-builder` | Edits code | Keeps implementation focused |
| `feature-tester` | Runs targeted checks | Gives proof, not just opinion |
| `feature-reviewer` | Critiques the diff | Catches things the builder missed |

This is more useful than one giant prompt because the responsibilities are separated.

## How To Use This In A New Project

In any project where you want the workflow:

1. Copy [opencode.jsonc](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/opencode.jsonc) into the project root.
2. Copy the entire [.opencode](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/.opencode) folder into the project root.
3. Run `opencode models` to see the model IDs available in your setup.
4. Open [opencode.jsonc](/Users/rakeshcheekatimala/Desktop/Learnings/open-code-workflow/opencode.jsonc) and optionally uncomment the model overrides for planner, builder, tester, and reviewer.
5. Start OpenCode from that project root.

```bash
cd /path/to/your-project
opencode
```

Then run one of these:

```text
/feature Add email validation to the signup form. Reject empty emails and invalid email format.
/fix The settings page crashes when the API returns an empty list.
/spec Add pagination to the users table.
/review-diff
/test-target signup form validation
```

## What To Expect During A Feature Run

For `/feature Add email validation...`, a healthy run looks like this:

1. `workflow` asks `feature-planner` to inspect the repo.
2. `feature-planner` returns a goal, likely files, steps, tests, and risks.
3. `workflow` shrinks the work to one slice if needed.
4. `feature-builder` edits the smallest set of files.
5. `feature-tester` runs the smallest relevant test or tells you what manual check is needed.
6. `feature-reviewer` inspects the diff and calls out issues or says there are no major issues.
7. `workflow` summarizes what changed, how it was tested, and what to do next.

That is the whole workflow.

## How To Smoke-Test This In A Brand-New Project

Use a tiny toy project first. Do not start with a real production repo.

Example:

```bash
mkdir opencode-smoke-test
cd opencode-smoke-test
git init
printf 'function isValidEmail(email) { return true }\nmodule.exports = { isValidEmail }\n' > email.js
printf 'const { isValidEmail } = require(\"./email\")\nconsole.log(isValidEmail(process.argv[2]))\n' > index.js
```

Then copy in:

- `opencode.jsonc`
- `.opencode/`

Then start OpenCode:

```bash
opencode
```

Now run:

```text
/feature Update email.js so it rejects empty strings and invalid email formats. Keep it to one small slice.
```

You are looking for this behavior:

- planner identifies `email.js` as the main file,
- builder changes only `email.js` or the minimum needed files,
- tester suggests a focused check,
- reviewer comments on edge cases like whitespace or malformed input.

If that works, the starter is doing its job.

## How To Make It Reusable For Anyone Using OpenCode

The easiest reusable shape is:

1. keep `opencode.jsonc` small,
2. keep prompts in `.opencode/prompts/`,
3. keep commands in `.opencode/commands/`,
4. keep agent responsibilities narrow,
5. keep model routing optional.

That is why this starter does not hardcode model IDs by default. Different people will have different providers and model names available. The starter should run with a user's current OpenCode defaults, and then they can opt into model routing later.

A good reuse pattern is:

- use Claude or another strong reasoning model for `feature-planner`,
- use Codex or another coding-heavy model for `feature-builder`,
- use a cheaper or faster model for `feature-tester`,
- use Gemini, Claude, or another critical model for `feature-reviewer`.

## Small DX Improvements You Can Add Next

Once this starter feels good, add only a few more things:

- a `/refactor` command for small cleanup work,
- a `/ship-note` command for release summaries,
- a language-specific test command set if your team is mostly one stack,
- repo-specific instructions via `instructions`,
- tighter `bash` permissions if you want stronger guardrails.

Do not add more ceremony until this basic loop saves you time.

## Current Limitations

- The starter cannot guess your exact model IDs; check them with `opencode models`.
- Some repos will need stack-specific test commands added to the tester permissions.
- A huge feature request still needs human judgment to break it into slices.

## Verified Against Current OpenCode Docs

I wrote this starter from scratch and checked the current OpenCode docs for the config shape:

- [Config](https://opencode.ai/docs/config/)
- [Agents](https://opencode.ai/docs/agents/)
- [Commands](https://opencode.ai/docs/commands/)
- [Permissions](https://opencode.ai/docs/permissions/)
