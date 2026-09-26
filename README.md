# Zaoqiniaoer MCP — send real people to check a real place

An MCP server that lets an AI agent **dispatch three unrelated human beings** to a
physical address in China, and get back what they actually saw.

Models are good at language and bad at being somewhere. This closes that gap:
one tool call, three strangers go to the address, photograph the entrance, walk in
and ask, and each writes down what they saw — independently.

- **Endpoint:** `https://zaoqiniaoerhannile.com/api/mcp` (Streamable HTTP)
- **Report:** 24–48 hours
- **Price:** from ¥700 (about $99) per address
- Operated by 芜湖早起鸟儿智能科技有限公司 (Wuhu, Anhui, China)

## Install

Add to your MCP client config:

```json
{
  "mcpServers": {
    "zaoqiniaoer": {
      "type": "http",
      "url": "https://zaoqiniaoerhannile.com/api/mcp",
      "headers": {
        "Authorization": "Bearer sma_live_YOUR_KEY"
      }
    }
  }
}
```

Get a key at <https://zaoqiniaoerhannile.com/developers/register> — self-serve,
no sales call.

## Tools

### `verify_place`

Send three unrelated people to an address and have them report what is there.

| Argument | Required | Meaning |
|---|---|---|
| `name` | yes | The company or shop name, e.g. `某某精密机械有限公司` |
| `claim` | yes | The thing you want confirmed, e.g. "this address has a factory actually operating" |
| `address` | one of | Street address |
| `url` | one of | Their storefront or homepage |
| `budget_yuan` | no | Defaults to the minimum. Higher budget, faster pickup. |

Returns an `order_id`. **This spends real money and sends real people outside**, so
the server instructs models to confirm with the user before calling it.

### `get_verification_result`

Look up progress, and the three independent accounts once the report is ready.

| Argument | Required | Meaning |
|---|---|---|
| `order_id` | yes | From `verify_place` |

That is the whole surface. There is deliberately no `cancel`, no `refund`, no
`change_price` tool — those are irreversible and involve money, so they stay with
humans in the web console.

## What this is not

- **Not a quality inspection.** We do not examine goods or test samples.
- **Not a factory audit.** No capacity, certification, or labour assessment.
- **Not a credit check.** We report what is at the address, not whether they pay debts.

It answers one question well: *what is actually at that address, right now, according
to three people who went and looked.*

## Why three people

Fooling three strangers who have never met each other is hard. One person can be
mistaken, or talked around at the gate. The three never see each other's answers.

Every verification is permanently attached to that company's public record and cannot
be edited or deleted afterwards — including by us, and including by them.

## Idempotency

The server derives an idempotency key from `(client, arguments, 10-minute window)`.
Models retry; a retry here would mean sending three more people and charging again.
Identical calls inside that window return the same order rather than creating a new one.

## Rate limits

30 `verify_place` calls per client per hour. Exceeding it returns a tool error, not a
protocol error, so the model can read it and back off.

## Docs

- Developer docs: <https://zaoqiniaoerhannile.com/developers>
- How we decide something is true: <https://zaoqiniaoerhannile.com/standard>
- Free self-check first: <https://zaoqiniaoerhannile.com/en/supplier-check>

## License

MIT for this repository (manifest, docs and examples). The hosted service itself is
proprietary.
