# ai-policy.json

**Machine-readable AI agent permissions for websites.**

`ai-policy.json` is a simple JSON file that website owners deploy at `/.well-known/ai-policy.json` to declare what AI agents can and cannot do with their content. It's the only format designed to grow from content-usage permissions into action-level permissions for agentic AI.

## Quick Look

```json
{
  "version": "1.0",
  "domain": "example.com",
  "permissions": {
    "training": "block",
    "search": "allow",
    "ai_input": "allow"
  }
}
```

Three fields. One file. Any agent can parse it in a single request.

- **`training`**: Can AI models train on this content?
- **`search`**: Can AI search engines index and cite this content?
- **`ai_input`**: Can AI systems use this content at inference time to generate responses?

## Why This Exists

The current AI permissions landscape is fragmented across 8+ formats: robots.txt, Cloudflare Content Signals, TDMRep, ai.txt, llms.txt, Terms of Service, meta tags, HTTP headers. Research across 1,000,000+ domains found that approximately 90% have no machine-readable AI policy at all, and thousands have conflicts between different signals.

`ai-policy.json` consolidates these signals into one structured document and serves as the canonical source of intent. Tools that generate `ai-policy.json` also generate the corresponding robots.txt directives, Content Signals, TDMRep headers, and ai.txt, so your policy works everywhere, not just with agents that support this format.

## How It Works

**For website owners:** Use a tool like [Maango's Policy Builder](https://maango.io) to set your preferences. You'll get `ai-policy.json` plus all the other formats (robots.txt additions, Content Signals, TDMRep headers, ai.txt) generated automatically. Deploy them to your site. Done.

**For agent developers:** Fetch `/.well-known/ai-policy.json` from any domain to get its AI policy in structured JSON. Or use the [Maango API](https://maango.io/docs) to check any domain. It synthesizes every signal (robots.txt, Content Signals, TDMRep, ToS, and ai-policy.json) into one answer.

```python
# Direct fetch
import httpx
r = httpx.get("https://example.com/.well-known/ai-policy.json")
policy = r.json()
if policy["permissions"]["training"] == "block":
    # don't use this content for training

# Or via Maango API (covers domains without ai-policy.json too)
r = httpx.get(
    "https://api.maango.io/v1/domain/example.com",
    headers={"Authorization": "Bearer YOUR_KEY"}
)
```

## Vocabulary Alignment

The three permission fields align with existing standards:

| ai-policy.json | Cloudflare Content Signals | IETF AIPREF |
|----------------|---------------------------|-------------|
| `training` | `ai-train` | Foundation Model Production |
| `search` | `search` | Search |
| `ai_input` | `ai-input` | *(under discussion)* |

This is intentional. `ai-policy.json` is interoperable with the most widely deployed AI permissions infrastructure, not competing with it.

## Full Specification

📄 **[Read the full spec →](spec.md)**

The specification covers discovery, field definitions, permission semantics, bot directives, interoperability mappings to other formats, versioning, security considerations, and examples.

## Schema Validation

Validate your `ai-policy.json` against the JSON Schema:

```
schema/v1.0.json
```

## Examples

| File | Scenario |
|------|----------|
| [minimal.json](examples/minimal.json) | Smallest valid file, just the three required fields |
| [blog.json](examples/blog.json) | Blog that blocks training, allows search and AI input |
| [ecommerce.json](examples/ecommerce.json) | E-commerce site with bot-specific directives |
| [healthcare-block-all.json](examples/healthcare-block-all.json) | Regulated site that blocks everything |
| [open-source-allow-all.json](examples/open-source-allow-all.json) | Open source project that allows everything |

## What's Next

`ai-policy.json` v1.0 handles content-usage permissions, i.e. what AI can do with your content. Future versions will extend into action-level permissions: what AI agents can do on your site (browse, submit forms, transact). The `capabilities` field is reserved for this purpose.

## Contributing

We welcome contributions. See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose changes.

## License

This specification is released under [CC0 1.0 Universal](LICENSE) (public domain). You can implement, extend, and use it freely.

---

Built by [Maango](https://maango.io), the permissions layer for the agentic internet.
