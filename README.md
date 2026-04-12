# OpenCode Multi-Agent Workflow: A Simple Example

This is about **OpenCode**: https://opencode.ai/

The easiest way to understand OpenCode is this:

> OpenCode lets you describe your AI engineering workflow in config, then use different agents for different jobs.

Instead of saying:

> "Claude, understand the problem, write the plan, edit the code, test it, review it, and tell me if it is safe."

You can say:

> "Planner, understand the problem. Builder, make the change. Reviewer, criticize the diff."

That is the whole idea.

## The Example

Imagine this task:

> Add email validation to a signup form.

A normal one-agent flow looks like this:

```text
User -> One AI model -> Plan + Code + Test + Review
```

The problem is that one model is doing everything in the same context. It may rush into code, forget the original goal, skip tests, or review its own work too kindly.

An OpenCode multi-agent flow can look like this:

```text
User
  -> workflow agent decides the sequence
  -> planner agent reads and plans, but cannot edit
  -> builder agent edits the code
  -> reviewer agent checks the diff, but cannot edit
```

This is useful because each agent has a smaller responsibility.

## The Three-Agent Version

Start with only three agents:

| Agent | Job | Allowed to edit? | Good model type |
|---|---|---:|---|
| `planner` | Understand the request and make a small plan | No | Claude/reasoning model |
| `builder` | Implement only the approved plan | Yes | Codex/coding model |
| `reviewer` | Review the final diff and find risks | No | Gemini/Codex/Claude review model |

Do not start with seven agents. Seven stages are useful later. Three agents are enough to learn the concept.

## Minimal OpenCode Config

Create this file in your project root:

```text
opencode.jsonc
```

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "workflow",

  "agent": {
    "workflow": {
      "description": "Coordinates a simple plan -> build -> review workflow.",
      "mode": "primary",
      "model": "anthropic/<your-claude-model>",
      "prompt": "You coordinate work. For every feature request: first ask planner to create a small plan, then ask builder to implement that plan, then ask reviewer to review the diff. Keep the task small.",
      "permission": {
        "task": {
          "*": "deny",
          "planner": "allow",
          "builder": "ask",
          "reviewer": "allow"
        },
        "edit": "deny",
        "bash": "ask"
      }
    },

    "planner": {
      "description": "Reads the codebase and writes a small implementation plan. It never edits files.",
      "mode": "subagent",
      "model": "anthropic/<your-claude-model>",
      "prompt": "You are the planner. Understand the user's request and the existing code. Return: goal, files likely involved, small implementation steps, and tests to run. Do not edit files.",
      "tools": {
        "write": false,
        "edit": false,
        "bash": true
      },
      "permission": {
        "bash": {
          "*": "ask",
          "rg *": "allow",
          "ls *": "allow",
          "git status *": "allow",
          "git diff *": "allow"
        }
      }
    },

    "builder": {
      "description": "Implements the approved plan. It can edit files.",
      "mode": "subagent",
      "model": "openai/<your-coding-model>",
      "prompt": "You are the builder. Implement only the plan you are given. Keep the change small. Follow the existing code style. Do not refactor unrelated code. After editing, summarize what changed.",
      "tools": {
        "write": true,
        "edit": true,
        "bash": true
      },
      "permission": {
        "edit": "ask",
        "bash": {
          "*": "ask",
          "rg *": "allow",
          "git diff *": "allow",
          "npm test *": "allow",
          "pnpm test *": "allow"
        }
      }
    },

    "reviewer": {
      "description": "Reviews the diff for bugs, missing tests, and unnecessary complexity. It never edits files.",
      "mode": "subagent",
      "model": "google/<your-gemini-or-review-model>",
      "prompt": "You are the reviewer. Review the current diff only. Look for correctness issues, edge cases, missing tests, and unnecessary complexity. Do not edit files. If the change is good, say so clearly.",
      "tools": {
        "write": false,
        "edit": false,
        "bash": true
      },
      "permission": {
        "bash": {
          "*": "ask",
          "git diff *": "allow",
          "git status *": "allow",
          "rg *": "allow"
        }
      }
    }
  },

  "command": {
    "feature": {
      "description": "Run a small feature through planner, builder, and reviewer.",
      "agent": "workflow",
      "template": "Run this through planner, builder, and reviewer. Keep the change small: $ARGUMENTS"
    }
  }
}
```

Replace the model placeholders with the model names from your OpenCode setup. The important part is the shape:

- `workflow` is the main agent you talk to.
- `planner` is a read-only thinking agent.
- `builder` is the only agent that should edit code.
- `reviewer` is a read-only critic.
- `permission.task` controls which subagents the workflow agent can call.
- `tools` and `permission` control what each agent can do.

## How You Would Use It

Open your project:

```bash
opencode
```

Then run:

```text
/feature Add email validation to the signup form. Reject empty emails and invalid email format.
```

What should happen conceptually:

```text
1. workflow receives the request
2. workflow asks planner to inspect the code and produce a plan
3. workflow asks builder before editing
4. builder changes the smallest needed code
5. builder runs focused tests if available
6. workflow asks reviewer to inspect the diff
7. reviewer reports issues or says the change is safe enough
```

## Why This Is Better Than One Big Prompt

The benefit is not that "many agents are smarter." That is the wrong mental model.

The benefit is that the workflow has **separation of duties**:

- The planner cannot accidentally edit files.
- The builder has a narrow implementation target.
- The reviewer is not the same role that wrote the code.
- The workflow agent controls the sequence.
- The config makes the process repeatable for the team.

For a senior engineer, this means fewer messy patches and clearer checkpoints.

For an engineering manager, this means the team can describe the process:

```text
We plan before code.
We keep implementation small.
We review before merge.
We restrict which agents can edit.
We can swap models without changing the workflow.
```

## When This Is Worth It

Use this workflow for:

- features that touch multiple files,
- refactors,
- bug fixes where correctness matters,
- work that needs review discipline,
- teams that want repeatable AI coding practices.

Do not use it for:

- renaming one variable,
- changing one string,
- quick throwaway prototypes,
- tasks where the agent handoff costs more than the code change.

The practical rule:

> If a bad patch would waste more time than the workflow costs, use the workflow.

## How To Extend To Seven Stages

Once the three-agent version makes sense, expand it like this:

| Stage | Agent | Purpose |
|---|---|---|
| Define | `definer` | Clarify the request and success criteria |
| Plan | `planner` | Break the work into small steps |
| Build | `builder` | Make the code change |
| Test | `tester` | Run tests and explain failures |
| Review | `reviewer` | Critique the diff |
| Simplify | `simplifier` | Remove accidental complexity |
| Ship | `shipper` | Summarize what changed and remaining risks |

But start with three. The point is to learn the workflow, not to create ceremony.

## The Cleanest Explanation

OpenCode is useful because it turns this:

```text
One model does everything.
```

into this:

```text
One workflow coordinates multiple focused agents.
```

That is the concept you want to get your hands dirty with locally.

## References Used For Config Shape

This example is written from scratch. I only checked the official OpenCode docs for current config concepts:

- https://opencode.ai/docs/config/
- https://opencode.ai/docs/agents/
