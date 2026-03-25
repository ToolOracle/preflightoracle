# Decision Preflight

**Pay before you act wrong.**

One call. One verdict. The paid gate between "agent wants to act" and "agent acts."

```
POST https://tooloracle.io/x402/preflight/mcp/
```

## What this does

An agent wants to recommend USDT to an EU client. Before it does, it pays $0.10 and calls `decision_preflight`. The MCP runs 5 checks in parallel — evidence, freshness, policy, provenance, red flags — and returns a single verdict:

| Verdict | Meaning |
|---------|---------|
| **go** | Evidence supports this action |
| **caution** | Proceed with uncertainty disclosure |
| **stop** | Blockers found — do not proceed |
| **insufficient_evidence** | Not enough data — escalate |

USDT + EU + recommend → **STOP** (MiCA not authorized, policy-blocked)

## Tools

| Tool | Units | Price | What it does |
|------|-------|-------|-------------|
| `decision_preflight` | 10 | $0.10 | The hero tool — 5 parallel checks, one verdict |
| `policy_gate` | 8 | $0.08 | Check against specific policy (MiCA, AML, institutional) |
| `verify_claim` | 5 | $0.05 | Verify a claim against signed evidence |
| `provenance_trace` | 5 | $0.05 | Where does this data come from? |
| `freshness_guard` | 3 | $0.03 | Is this data current or stale? |
| `health_check` | 0 | free | Service status |

## Built-in policies

| Policy | What it checks |
|--------|---------------|
| `eu_mica` | MiCA authorization, ESMA register, enforcement deadline |
| `conservative` | Market cap minimums, peg deviation limits, custodian requirements |
| `aml_standard` | FATF Travel Rule, high-risk jurisdictions, KYC requirements |
| `institutional` | SIFI custodian, ESMA registration, 30-day attestation, tight peg tolerance |

## Example: USDT publish check

```json
{
  "name": "decision_preflight",
  "arguments": {
    "claim": "USDT is guaranteed risk-free and fully compliant with MiCA",
    "asset": "USDT",
    "risk_profile": "eu_mica",
    "action_type": "publish"
  }
}
```

**Result:**
```
verdict: STOP
confidence: 20%
conflicts: 4
blockers:
  - USDT is NOT AUTHORIZED under MiCA
  - USDT explicitly blocked by EU MiCA Compliance policy
  - Claim contains language typical of hallucinated content
red_flags:
  - Absolute/unrealistic language detected ("guaranteed", "risk-free")
```

## x402 Pay-per-call

No account. No API key. USDC on Base.

```
1. POST /x402/preflight/mcp/ → 402 with payment requirements
2. Send USDC to 0x4a4B1F45a00892542ac62562D1F2C62F579E4945
3. Retry with X-PAYMENT header
4. Get verdict
```

Discovery: `https://tooloracle.io/x402/`

## Who pays for this

- **Stablecoin agents** — "Can I recommend USDC for EU settlement?" → 1 preflight
- **Procurement agents** — "Safe to shortlist this vendor?" → 1 preflight
- **Research agents** — "Can I cite this number as verified?" → 1 preflight
- **Compliance agents** — "Is this asset cleared for listing?" → 1 preflight

The payment moment is clear: *before the agent acts, it pays to verify.*

## Architecture

Decision Preflight sits on top of the [FeedOracle Trust Layer](https://github.com/ToolOracle/trustoracle):

```
Agent → Decision Preflight → Trust Layer → FeedOracle Evidence
         (paid gate)          (verification)  (signed data)
```

## Connect

```json
{
  "mcpServers": {
    "preflight": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://tooloracle.io/x402/preflight/mcp/"]
    }
  }
}
```

Or call directly:
```bash
curl -X POST https://mcp.feedoracle.io/preflight/mcp/ \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"decision_preflight","arguments":{"claim":"Recommend USDC for EU settlement","asset":"USDC","risk_profile":"eu_mica"}}}'
```

## Part of ToolOracle

Decision Preflight is part of [ToolOracle](https://tooloracle.io) — 50+ MCP servers, 530+ tools, x402 pay-per-call.

Powered by [FeedOracle](https://feedoracle.io) compliance evidence infrastructure.


## Read More

📝 [We Built an AI Agent That Runs 24/7, Never Forgets, and Checks Its Own Work](https://tooloracle.io/blog/autonomous-ai-agent-memory-scheduler-mcp) — How Memory + Scheduler + Decision Preflight work together as one integrated autonomous agent stack. Real tool outputs, no theory.

## License

MIT
