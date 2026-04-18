# Contributing to ai-policy.json

Thanks for your interest in improving `ai-policy.json`. This specification is open (CC0) and we welcome contributions from the community.

## How to Propose Changes

### For Discussion

Open a [GitHub Discussion](https://github.com/maango-io/ai-policy/discussions) to float ideas, ask questions, or explore directions before writing a formal proposal. This is the right place for questions like "should we add field X?" or "how should this interact with standard Y?"

### For Specific Changes

1. Open an [Issue](https://github.com/maango-io/ai-policy/issues) describing the problem and proposed solution.
2. If the change involves modifying the spec, fork the repo and submit a Pull Request with the proposed edits to `spec.md`.
3. Include a rationale section in your PR description explaining why the change is needed and how it interacts with existing fields.

### For New Examples

If you have a real-world use case that isn't covered by the existing examples, submit a PR adding a new `.json` file to the `examples/` directory with a brief description.

## What We're Looking For

- Bug fixes or clarifications in the spec language
- New examples covering underrepresented site types
- Improvements to the JSON Schema
- Interoperability mapping corrections (if a mapping to robots.txt, Content Signals, TDMRep, or ai.txt is incorrect or incomplete)
- Proposals for future version features (open as Discussions first)

## What We're Not Looking For Right Now

- Changes to the three core permission fields (`training`, `search`, `ai_input`). These are aligned with the Cloudflare Content Signals vocabulary and the IETF AIPREF working group's emerging terminology. Changes here require ecosystem-wide coordination.
- Implementation code (SDKs, parsers, validators). These belong in separate repositories.

## Style

- The spec uses RFC 2119 keywords (MUST, SHOULD, MAY, etc.) deliberately. Don't change their casing or usage without understanding the implications.
- JSON examples should use 2-space indentation.
- Bot lists in examples should be sorted alphabetically.

## License

By contributing, you agree that your contributions will be released under the CC0 1.0 Universal license.
