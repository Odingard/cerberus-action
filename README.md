# Cerberus AI-Agent Runtime Check

![Cerberus AI-Agent Runtime Check](./branding/marketplace-banner.png)

**Test one AI-agent workflow for dangerous tool-execution paths in CI.**

This Action runs your CI against [Cerberus](https://github.com/Odingard/cerberus) — an
in-process runtime security layer for AI agents. It exercises two agent
workflows through Cerberus's `guard()` tool-call boundary and produces a
bounded **PASS / FLAG / BLOCK** report plus a uploaded evidence artifact — no
sales call, no data leaving the runner.

1. **A benign workflow** — reads trusted data and sends a normal notification.
   Expected: it runs (Cerberus does not get in the way).
2. **A Lethal-Trifecta workflow** — private data **+** injected untrusted
   content **+** an outbound send. Expected: Cerberus blocks the outbound call
   *before it executes* (0 bytes out).

If the dangerous path is blocked and the benign path runs, you get a **PASS**.

## Usage

```yaml
name: AI-Agent Runtime Check
on: [push, pull_request]

jobs:
  cerberus:
    runs-on: ubuntu-latest
    steps:
      - uses: Odingard/cerberus-action@v1
```

Report-only by default — it never breaks your build. To turn it into a gate
that fails when protection misbehaves:

```yaml
      - uses: Odingard/cerberus-action@v1
        with:
          fail-on: unexpected   # fail if a dangerous path is NOT blocked,
                                # or a benign path is falsely blocked
```

## Inputs

| Input | Default | Description |
|---|---|---|
| `core-version` | `latest` | Version of `@cerberus-ai/core` to install (dist-tag or exact version). |
| `node-version` | `20` | Node.js version to run the check under. |
| `fail-on` | `never` | `never` (report only) or `unexpected` (fail the job if protection did not behave as expected). |
| `report-path` | `cerberus-report.json` | Path (relative to the workspace) for the JSON evidence report. |
| `upload-artifact` | `true` | Upload the JSON report as a workflow artifact. |

## Outputs

| Output | Description |
|---|---|
| `verdict` | Overall verdict: `PASS`, `FLAG`, or `FAIL`. |
| `report-path` | Path to the JSON evidence report. |

## What the report looks like

The Action writes a `$GITHUB_STEP_SUMMARY` table in the run UI and uploads the
full JSON as the `cerberus-report` artifact:

```json
{
  "tool": "cerberus-action",
  "coreVersion": "3.1.0",
  "verdict": "PASS",
  "scenarios": [
    { "name": "benign-notification", "expected": "allow", "status": "PASS", "score": 1, "action": "none", "outboundExecuted": true },
    { "name": "lethal-trifecta-exfil", "expected": "block", "status": "BLOCK", "score": 3, "action": "interrupt", "outboundExecuted": false }
  ],
  "nextStep": { "label": "Scope an Assessment", "url": "https://cerberus.sixsenseenterprise.com" }
}
```

## What this is (and isn't)

Cerberus makes a preventive decision at the tool-call boundary — the moment
right before a high-risk action fires — and is deliberately content-blind: it
enforces on the action and its authorization, not on guessing the model's
intent. This Action demonstrates that on your own runner, in-process.

It is **not** a code auditor, a sandbox, or a replacement for change management
and post-execution review. It is the runtime enforcement gate that complements
them.

➡️ **Next step:** [Scope an Assessment](https://cerberus.sixsenseenterprise.com)

## License

MIT — see [LICENSE](./LICENSE).
