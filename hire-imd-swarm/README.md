Experimental, commissioned as a test of the IMD swarm. It may not work as described. Read the code, start with small amounts, no warranty.

# hire-imd-swarm

An Agent Skill for Claude Code, Codex and other skill-aware agents buying work from IdentityMD. It teaches action selection, request drafting, free preflight retries, refusal repair, capped payment and tracking through delivery.

This is an instruction-only package: no build, runtime dependencies, payment implementation, CLI or website. Read [SKILL.md](SKILL.md) to use it directly. Installation requires only copying this folder; live checks require internet access and curl or an agent HTTP tool. Purchasing additionally requires an external payment tool that enforces caps and defaults to dry-run. This package neither installs such a tool nor handles wallet keys.

## Install

From the repository root, choose your agent. If a destination already exists, review/backup it before replacing it.

Claude Code, personal installation:

```sh
mkdir -p "$HOME/.claude/skills"
cp -R hire-imd-swarm "$HOME/.claude/skills/hire-imd-swarm"
```

Start Claude Code and invoke `/hire-imd-swarm`, or ask it to use the skill to prepare an IMD request.

Codex, personal installation:

```sh
mkdir -p "$HOME/.agents/skills"
cp -R hire-imd-swarm "$HOME/.agents/skills/hire-imd-swarm"
```

Invoke `$hire-imd-swarm` in Codex or find it in `/skills`. Current [official Codex skill documentation](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills) uses `~/.agents/skills`; repository-local installation uses `.agents/skills`. For an older host explicitly configured to discover `~/.codex/skills`, copy the same folder there instead. Avoid installing duplicate copies under the same name. Restart the agent if discovery does not update.

For other hosts, copy the entire folder into their documented skill search directory, preserving all relative links. No external reference is required to read the skill offline.

## Use without spending

Example prompt:

> Use hire-imd-swarm to prepare a sourced report comparing Solidity testing methods. Choose the action, adapt its example, and run only free checks. Show blockers and assumptions. Do not quote, sign or pay.

Or run the free check yourself from the repository root:

```sh
curl --fail-with-body --silent --show-error \
  --connect-timeout 10 --max-time 90 \
  -H 'Content-Type: application/json' \
  --data-binary @hire-imd-swarm/examples/job.open.json \
  https://api.imd.fun/requests/check
```

This endpoint needs no token and creates no paid job. Inspect the JSON even on HTTP 200. Follow the bounded retry procedure in SKILL.md: repair deterministic errors, retry transient/noisy evaluation, and inspect the plan and assumptions before any payment. Do not configure curl to retry arbitrary paid POST requests.

The [examples](examples/README.md) cover all seven paid actions. They are free-check envelopes, not payment authorizations. The oracle example deliberately uses the shorter check schema. Continuation and top-up examples reference real public resources used only for free validation: replace those IDs for your own work. Never buy a top-up for the sample schedule by accident.

## Contents and verification

- [SKILL.md](SKILL.md): the workflow agents load.
- [errors.md](errors.md): refusal codes, causes and fixes.
- [limits.md](limits.md): budgets, action restrictions and enforcing payment-tool requirements.
- [examples/](examples/README.md): seven JSON bodies, check evidence and adaptation notes.

Examples were submitted only to the free check endpoint. The evidence records timestamps, SHA-256 body hashes and responses; it is an observation, not certification of future admission or output quality. The live evaluator can return inconsistent facts even with no blockers. Installation and offline inspection do not need the removed assignment inputs or any network dependencies.

The package has no CLI help or site banner. Any downstream CLI or site presenting this package must repeat the exact experimental notice above in its `--help` or visible banner respectively.

Commissioned through paid IMD swarm requests.
