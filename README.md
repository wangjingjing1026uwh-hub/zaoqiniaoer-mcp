# Zaoqiniaoer MCP — ask a real person who is actually there

An MCP server that lets an AI agent **pay a real person in China to go look at
something and say what they saw**.

Models are good at language and bad at being somewhere. There is no database that
holds "is that shop open right now". Someone has to walk over and look.

Two tiers, same pipeline:

| | What you get | Price | Time |
|---|---|---|---|
| **Ask someone there** | One person nearby goes and tells you what they saw | **¥5–30** (~$0.70–4) | hours |
| **Verify a place** | Three unrelated people each go, each judge independently, you get a report | **from ¥70** (~$10) | 24–48h |

Start with the cheap one. It answers most questions and costs less than a coffee.
The expensive one is for when you need something you can show to someone else.

- **Endpoint:** `https://zaoqiniaoerhannile.com/api/mcp` (Streamable HTTP)
- Operated by 芜湖早起鸟儿智能科技有限公司 (Wuhu, Anhui, China)
- Real end-to-end machine order ran 2026-08-15: ordered, charged, dispatched,
  report returned — no human in the loop on our side.

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

Four tools: **two ways to order, two ways to read the result.** Nothing else —
no `cancel`, no `refund`, no `change_price`. Those are irreversible and involve
money, so they stay with humans in the web console.

### `ask_someone_there` — start here

One person who happens to be nearby goes and looks. Hours, not days.

| Argument | Required | Meaning |
|---|---|---|
| `question` | yes | What you want to know, in one sentence. Specific beats vague: "is the Lanzhou noodle place opposite Golden Eagle still open" is far more useful than "how is that shop" |
| `place_hint` | yes | Where to go. An address, a shop name, a street corner — anything findable |
| `city` | no | e.g. `芜湖`. Helps us find someone close |
| `price_yuan` | no | ¥5–30, defaults to ¥5. Pay more, get picked up faster |

Returns an `ask_id`. **If nobody takes it before it expires, your balance is
automatically refunded** — you are not charged for a trip nobody made.

### `verify_place`

Three unrelated people go to an address and each report what is there.

| Argument | Required | Meaning |
|---|---|---|
| `name` | yes | The company or shop name, e.g. `某某精密机械有限公司` |
| `claim` | yes | The thing you want confirmed, e.g. "this address has a factory actually operating" |
| `address` | one of | Street address |
| `url` | one of | Their storefront or homepage |
| `budget_yuan` | no | Defaults to the minimum (¥70). Higher budget, faster pickup |

Returns an `order_id`. **This spends real money and sends real people outside**, so
the server instructs models to confirm with the user before calling it.

### `get_ask_answer` / `get_verification_result`

Look up progress and results, by `ask_id` and `order_id` respectively.

**Until someone has actually answered, `answer` is `null`.** That means nobody has
been there yet — not "no result". The tool description tells the model, in so many
words, not to invent an answer at that point.

## What this is not

- **Not a quality inspection.** We do not examine goods or test samples.
- **Not a factory audit.** No capacity, certification, or labour assessment.
- **Not a credit check.** We report what is at the address, not whether they pay debts.

It answers one question well: *what is actually there right now, according to
someone who went and looked.*

## Why three people

Fooling three strangers who have never met each other is hard. One person can be
mistaken, or talked around at the gate. The three never see each other's answers.

Every verification is permanently attached to that company's public record and cannot
be edited or deleted afterwards — including by us, and including by them.

## Idempotency

The server derives an idempotency key from `(client, arguments, 10-minute window)`.
Models retry. **A retry here means sending another human outside and charging you
again** — so identical calls inside that window return the existing order instead of
creating a new one. Over REST you pass your own `Idempotency-Key` header; it is
required, not optional.

## Rate limits

30 ordering calls per client per hour. Exceeding it returns a tool error, not a
protocol error, so the model can read it and back off.

## Docs

- Developer docs: <https://zaoqiniaoerhannile.com/developers>
- How we decide something is true: <https://zaoqiniaoerhannile.com/standard>
- Free self-check first: <https://zaoqiniaoerhannile.com/en/supplier-check>

## License

MIT for this repository (manifest, docs and examples). The hosted service itself is
proprietary.
