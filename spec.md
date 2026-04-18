# AI Policy Manifest Specification

**Version:** 1.0  
**Status:** Draft  
**Last Updated:** April 2026  
**License:** CC0 1.0 Universal (Public Domain Dedication)  
**Repository:** https://github.com/maango-io/ai-policy  
**Reference Implementation:** https://maango.io

---

## Abstract

This specification defines `ai-policy.json`, a machine-readable file format for website owners to declare permissions governing how AI agents may interact with their domain. The file is hosted at `/.well-known/ai-policy.json` and expresses purpose-level permissions (training, search, AI input), specific bot directives, and human-readable policy descriptions in a single structured document.

Existing web standards control what crawlers can access and what content can be used for training. `ai-policy.json` consolidates those signals and extends them toward controlling what agents can *do*. As AI agents evolve from passive crawlers to active participants on websites, the permissions model must evolve with them. This specification defines that model.

`ai-policy.json` is designed to complement, not replace, existing standards. It consolidates the signals currently scattered across robots.txt, ai.txt, TDMRep, HTTP headers, HTML meta tags, and Terms of Service into one authoritative file that any agent can parse in a single request. Tools generating `ai-policy.json` SHOULD also generate the corresponding signals in other formats to ensure that the domain owner's preferences are enforceable by agents that do not yet support this specification.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Conventions](#2-conventions)
3. [Discovery](#3-discovery)
4. [File Format](#4-file-format)
5. [Field Definitions](#5-field-definitions)
6. [Permission Semantics](#6-permission-semantics)
7. [Bot Directives](#7-bot-directives)
8. [Interoperability and Signal Generation](#8-interoperability-and-signal-generation)
9. [Versioning and Extensibility](#9-versioning-and-extensibility)
10. [Relationship to Existing Standards](#10-relationship-to-existing-standards)
11. [Security Considerations](#11-security-considerations)
12. [IANA Considerations](#12-iana-considerations)
13. [Examples](#13-examples)
14. [Future Extensions](#14-future-extensions)
15. [References](#15-references)

---

## 1. Introduction

### 1.1 Problem Statement

The current web permissions landscape for AI agents is fragmented across multiple standards, none of which individually provides a complete picture:

- **robots.txt** (RFC 9309) controls which bots can access which paths, but has no concept of access purpose. Blocking GPTBot prevents both training crawls and real-time user-initiated browsing.
- **Cloudflare Content Signals** introduce purpose-level categories (`search`, `ai-train`, `ai-input`) as extensions to robots.txt, but are limited to sites using Cloudflare's infrastructure and cannot express permissions beyond three content-usage categories.
- **TDMRep** (W3C) provides EU-specific text and data mining opt-outs via HTTP headers but has minimal adoption.
- **ai.txt** proposes purpose-level permissions but lacks structured formatting, bot-specific directives, and significant adoption.
- **llms.txt** provides content structure for LLM consumption but does not express permissions.
- **RSL** (Really Simple Licensing) provides XML-based licensing and payment terms but focuses on commercial licensing rather than permission declaration.
- **Terms of Service** express comprehensive permissions in natural language but are not machine-readable and cannot be parsed by agents at decision time.

No existing standard combines purpose-level permissions, bot-specific directives, and structured metadata in a single JSON document optimized for AI agent consumption.

### 1.2 Design Goals

1. **Machine-readable:** JSON format, parseable by any programming language with zero custom parsing logic.
2. **Purpose-aware:** Distinguishes between training, search, and AI input, the three primary AI content-usage categories, aligned with the Cloudflare Content Signals vocabulary and the IETF AIPREF working group's emerging terminology.
3. **Bot-specific:** Lists exactly which bots are blocked and which are allowed, eliminating ambiguity.
4. **Human-readable:** Includes a plain-language description suitable for LLM-based agents that interpret context.
5. **Complementary:** Does not replace robots.txt. Works alongside it. robots.txt handles path-level crawl access; `ai-policy.json` handles purpose-level permissions.
6. **Consolidating:** Serves as the canonical source of intent from which corresponding signals in other formats (robots.txt directives, Content Signals, TDMRep headers, ai.txt) can be derived.
7. **Simple to create:** A non-technical website owner can generate a valid file through a wizard interface without editing JSON manually.
8. **Extensible:** Future versions can add action-level permissions, conditional access, and agent identity requirements without breaking backward compatibility.

### 1.3 Scope

This specification defines:

- The file format and schema for version 1.0
- The discovery mechanism (well-known URI)
- The semantics of each permission field
- The interoperability mappings to existing standards
- The versioning and extensibility model

This specification does not define:

- Enforcement mechanisms (how agents or servers enforce the policy)
- Rate limiting (a runtime concern, not a static policy declaration)
- Authentication or payment protocols (RSL handles this)
- Path-level permissions (planned for future versions)
- Legal enforceability (which varies by jurisdiction)
- Action-level permissions (planned for future versions)

---

## 2. Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

---

## 3. Discovery

### 3.1 Primary Location

The AI Policy Manifest MUST be served at:

```
https://{domain}/.well-known/ai-policy.json
```

The file MUST be served with `Content-Type: application/json`.

The file MUST be accessible without authentication (no login, no API key, no cookies required).

The file SHOULD be accessible over HTTPS. HTTP-only hosting is permitted but not recommended.

### 3.2 Fallback Discovery

If the file is not found at the well-known location, agents MAY check for the following fallbacks, in order:

1. **HTTP response header:** `AI-Policy: /.well-known/ai-policy.json` (or a full URL) on any HTTP response from the domain.
2. **robots.txt directive:** A line reading `AI-Policy: /.well-known/ai-policy.json` within the domain's robots.txt file.
3. **Registry lookup:** A query to a public permissions registry (such as the Maango Registry API) that may have the domain's policy on file.

These fallback mechanisms are OPTIONAL for both publishers and consumers.

### 3.3 Caching

Agents SHOULD cache the file for a reasonable period (RECOMMENDED: 24 hours minimum, 7 days maximum) to avoid excessive requests to the domain. Agents SHOULD respect standard HTTP caching headers (`Cache-Control`, `ETag`, `Last-Modified`) if present.

---

## 4. File Format

The AI Policy Manifest is a JSON document conforming to RFC 8259. The document MUST be a single JSON object at the top level.

### 4.1 Minimal Valid File

The smallest valid `ai-policy.json` contains only the required fields:

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

### 4.2 Complete Example (Version 1.0)

```json
{
  "schema": "https://raw.githubusercontent.com/maango-io/ai-policy/main/schema/v1.0.json",
  "version": "1.0",
  "domain": "example.com",
  "updated": "2026-04-15T10:00:00Z",
  "site_type": ["ecommerce", "blog"],
  "description": "We allow AI agents to discover and reference our content for search and real-time responses. AI training on our content is not permitted.",
  "permissions": {
    "training": "block",
    "search": "allow",
    "ai_input": "allow"
  },
  "bots": {
    "blocked": [
      "AI2Bot",
      "Amazonbot",
      "anthropic-ai",
      "Applebot-Extended",
      "Bytespider",
      "CCBot",
      "ClaudeBot",
      "Cohere-ai",
      "Diffbot",
      "FacebookBot",
      "Google-Extended",
      "GPTBot",
      "img2dataset",
      "Meta-ExternalAgent",
      "Omgilibot",
      "Timpibot"
    ],
    "allowed": [
      "ChatGPT-User",
      "OAI-SearchBot",
      "PerplexityBot",
      "YouBot"
    ]
  },
  "contact": "ai-policy@example.com",
  "policy_url": "https://example.com/ai-policy",
  "registry": "maango.io",
  "verification": "maango_vf_abc123",
  "generated_by": "maango.io"
}
```

---

## 5. Field Definitions

### 5.1 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `version` | string | The version of this specification the file conforms to. MUST be `"1.0"` for this version. |
| `domain` | string | The fully qualified domain name this policy applies to, without protocol prefix (e.g., `"example.com"`, not `"https://example.com"`). |
| `permissions` | object | The three permission declarations. See Section 6. |

### 5.2 Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `schema` | string (URI) | URL pointing to the JSON Schema definition for validation. |
| `updated` | string (ISO 8601) | The date and time this policy was last created or modified. RECOMMENDED. |
| `site_type` | array of strings | One or more tags describing the type of website. Informational, not prescriptive. Suggested values: `"ecommerce"`, `"blog"`, `"saas"`, `"portfolio"`, `"education"`, `"regulated"`, `"community"`, `"personal"`. Custom values are permitted. |
| `description` | string | A plain-language summary of the policy. **This field is informational only and MUST NOT be relied upon for compliance determination.** Agents determining compliance MUST use the structured permission fields. Maximum RECOMMENDED length: 500 characters. |
| `bots` | object | Explicit lists of bot user-agent names that are blocked or allowed. See Section 7. |
| `contact` | string (email) | An email address for AI-related policy inquiries. Domain owners SHOULD use a role-based address (e.g., `ai-policy@example.com`) rather than a personal email. |
| `policy_url` | string (URI) | URL to a human-readable policy page. The URL need not be on the same domain as this file. |
| `registry` | string | The permissions registry service managing this domain's policy. Used by generating services for domain verification and policy management. Agents SHOULD ignore this field when determining permissions. |
| `verification` | string | An opaque verification token used by the generating service to match this file to a policy creation session. Agents SHOULD ignore this field when determining permissions. |
| `generated_by` | string | Informational. Identifies the tool or service that generated this file. Agents MUST NOT use this field for permission determination. |
| `capabilities` | object | Reserved for future versions. V1 processors MUST ignore this field if present. Future versions will use this field for action-level permissions and agent identity requirements. |

### 5.3 Unknown Fields

Agents encountering fields not defined in this specification MUST ignore them. This ensures forward compatibility as new fields are added in future versions.

Publishers MUST NOT add fields that contradict the semantics of defined fields. The `permissions` object is authoritative.

---

## 6. Permission Semantics

The `permissions` object contains three fields, each with a value of `"allow"` or `"block"`.

### 6.1 `training`

Declares whether AI model providers may use this domain's content as training data for machine learning models.

- **`"block"`**: The domain owner does not consent to their content being used for AI model training. Crawlers that collect data for training (such as GPTBot, CCBot, Google-Extended) SHOULD NOT access this domain for training purposes.
- **`"allow"`**: The domain owner permits use of their content for AI model training.

This field corresponds to Cloudflare Content Signals' `ai-train`, the IETF AIPREF vocabulary's "Foundation Model Production" category, and the TDMRep `TDM-Reservation` header. Under the EU AI Act (Article 53(1)(c)) and the DSM Copyright Directive (Article 4(3)), this field serves as a machine-readable reservation of rights when set to `"block"`.

### 6.2 `search`

Declares whether AI-powered search engines and assistants may index and reference this domain's content when answering user queries.

- **`"block"`**: AI search agents SHOULD NOT include this domain's content in search results or synthesized answers.
- **`"allow"`**: AI search agents may index, cite, summarize, and reference this domain's content in response to user queries. Search results SHOULD direct users back to the original source.

This field corresponds to Cloudflare Content Signals' `search` and the IETF AIPREF vocabulary's "Search" category. Search, as defined here, covers uses that direct users to the location of the original asset. It does not include AI-generated summaries that substitute for visiting the source; that usage falls under `ai_input`.

### 6.3 `ai_input`

Declares whether AI systems may use this domain's content as input at inference time to generate responses for users.

- **`"block"`**: AI systems SHOULD NOT fetch and use this domain's content for real-time response generation, retrieval-augmented generation (RAG), grounding, or other inference-time content consumption.
- **`"allow"`**: AI systems may access and reference this domain's content in real-time when generating responses.

This field corresponds to Cloudflare Content Signals' `ai-input`. The distinction between `search` and `ai_input` is: `search` covers uses that direct users back to the source; `ai_input` covers uses where content is consumed to generate a response and the user may never visit the source. A domain might allow search indexing but block AI input, or vice versa.

### 6.4 Precedence

When an agent's intended use case spans multiple permissions (e.g., a search engine that also uses crawled data for training), the most restrictive applicable permission applies. If `search` is `"allow"` but `training` is `"block"`, the agent may access the domain for search purposes but MUST NOT use the accessed content for training.

### 6.5 Default Behavior

Well-formed `ai-policy.json` files MUST declare all three permission fields (`training`, `search`, `ai_input`). The JSON Schema in `schema/v1.0.json` enforces this. Tools generating `ai-policy.json` MUST include all three fields so the domain owner's intent is explicit and unambiguous.

For robustness, if an agent encounters a non-conformant file where the `permissions` object is present but a specific field is missing, the agent SHOULD assume `"allow"` for the missing field.

This default aligns with the established conventions of the open web:

- **robots.txt** treats absence of a `Disallow` directive as implicit permission to crawl.
- **The EU DSM Copyright Directive (Article 4(3))** treats content as available for text and data mining unless rights are expressly reserved.
- **Cloudflare Content Signals** treats absent signals as neutral rather than as a reservation.

Default-allow also protects publishers from accidental self-harm. Silence defaulting to block would mean a malformed file could cause a publisher to disappear from AI search and real-time answer systems without their knowledge, cutting off discovery and traffic. Because tools generating `ai-policy.json` MUST include all three permissions explicitly, this default only applies to files that have been manually edited or corrupted, not to files produced by conformant tooling.

If no `ai-policy.json` file exists for a domain, agents SHOULD NOT infer any permissions from the absence of the file. The absence of a file is not an expression of preference.

---

## 7. Bot Directives

The optional `bots` object provides explicit lists of bot user-agent strings that are blocked or allowed. This consolidates the bot-specific information typically maintained in robots.txt into the same document as the purpose-level permissions.

### 7.1 Structure

```json
{
  "bots": {
    "blocked": ["Bytespider", "CCBot", "GPTBot"],
    "allowed": ["ChatGPT-User", "PerplexityBot"]
  }
}
```

Both `blocked` and `allowed` are arrays of strings. Each string is a bot user-agent name as it appears in the User-Agent HTTP header or as referenced in robots.txt.

Arrays SHOULD be sorted alphabetically (case-insensitive) for consistency and readability.

### 7.2 Semantics

The `bots` object provides explicit overrides for specific user agents. It does NOT function as a whitelist. The `permissions` object is the universal rule that applies to all bots; the `bots` lists fine-tune that rule for named exceptions.

The rules are:

1. **The `permissions` object is the baseline.** It governs every bot, known or unknown, based on the bot's intended use case (training, search, or ai_input).

2. **The `blocked` list is an explicit reinforcement.** A bot listed in `blocked` SHOULD NOT access this domain for any purpose, regardless of what the `permissions` object allows. This is useful for naming specific training crawlers (GPTBot, CCBot, etc.) even when `training` is already `"block"` in `permissions`, to provide redundancy against bots that might ignore purpose-level semantics but respect explicit user-agent blocks.

3. **The `allowed` list is an explicit exception.** A bot listed in `allowed` MAY access this domain even when the `permissions` object would block its use case. This is useful for naming specific user-facing agents (such as ChatGPT-User or PerplexityBot) that should always be able to access the content to serve real-time user queries. However, the `allowed` list does not grant blanket permission: a bot listed as `allowed` that intends to use content for a purpose that is `"block"` in `permissions` MUST NOT do so for that purpose. The `allowed` list grants access; the `permissions` object still constrains what can be done with that access, unless the specific use case is what the `allowed` listing is intended to grant.

4. **A bot not listed in either array falls back to the `permissions` object.** Absence from the `allowed` list does NOT imply blocking. This design ensures that new AI agents emerging after the policy is published are handled sensibly based on what they do, without requiring the policy to be updated every time a new bot appears.

### 7.3 Worked Example

Consider a blog that publishes the following policy:

```json
{
  "permissions": {
    "training": "block",
    "search": "allow",
    "ai_input": "allow"
  },
  "bots": {
    "blocked": ["CCBot", "Google-Extended", "GPTBot"],
    "allowed": ["ChatGPT-User", "PerplexityBot"]
  }
}
```

The following scenarios illustrate how different bots are handled:

- **GPTBot arrives to collect training data.** Explicitly blocked by name in `bots.blocked`. Does not access. (Also blocked by `training: "block"` in permissions, but the explicit listing provides redundancy.)

- **ChatGPT-User arrives to fetch a page because a ChatGPT user asked about it.** Explicitly allowed by name in `bots.allowed`. Accesses the content for AI input. This is exactly what the site owner intended.

- **Bytespider arrives for training.** Not listed in either array. Falls through to the `permissions` object. `training` is `"block"`, so Bytespider does not access for training purposes.

- **A new AI search agent called "NewSearchBot" launches in 2027 and visits the site.** Not listed in either array. Falls through to the `permissions` object. `search` is `"allow"`, so NewSearchBot may index the content for search. The policy does not need to be updated to accommodate this new bot.

- **An AI summarization agent called "SummaryBot" arrives to fetch the page in response to a user query.** Not listed in either array. Falls through to the `permissions` object. `ai_input` is `"allow"`, so SummaryBot may access the content for real-time response generation.

- **A malicious scraper that ignores all permissions signals arrives.** The specification cannot stop it. Enforcement of the policy requires additional technical measures (WAF rules, bot management, rate limiting) or legal action. `ai-policy.json` expresses intent; it does not enforce it.

### 7.4 When to Use Each Pattern

**To block all AI access universally:** Set all three permissions to `"block"` and omit the `bots` object. Every bot, known or unknown, is blocked for every purpose. This is the pattern used by the regulated healthcare example in Section 13.

**To allow all AI access universally:** Set all three permissions to `"allow"` and omit the `bots` object. Every bot may access for any purpose. This is the pattern used by the open source community example in Section 13.

**To allow some purposes and block others, applied to all bots:** Set the `permissions` fields accordingly and omit the `bots` object. For example, `training: "block"`, `search: "allow"`, `ai_input: "allow"` means all bots may access for search and AI input but none may use the content for training.

**To reinforce purpose-level rules with explicit bot names:** Set the `permissions` fields and add a `blocked` list naming the specific training crawlers you want to call out (GPTBot, CCBot, etc.). This is the most common pattern in practice and what the blog and ecommerce examples use.

**To grant explicit exceptions to user-facing agents:** Add an `allowed` list naming bots that serve real-time user queries (ChatGPT-User, PerplexityBot, OAI-SearchBot). These bots will access the site even if their specific use case might otherwise be ambiguous.

### 7.5 Conflict with robots.txt

If a bot is listed as `allowed` in `ai-policy.json` but Disallowed in robots.txt, the bot SHOULD respect robots.txt for crawl access (as robots.txt is the established access control mechanism) and use `ai-policy.json` for purpose-level permissions.

In practice: robots.txt controls whether the bot can visit at all; `ai-policy.json` controls what the bot may do with the content it accesses.

Domain owners are encouraged to keep their robots.txt and `ai-policy.json` consistent. Tools that generate `ai-policy.json` SHOULD also generate corresponding robots.txt directives.

---

## 8. Interoperability and Signal Generation

`ai-policy.json` is designed as the **canonical source of intent** from which corresponding signals in other formats are derived. Tools generating `ai-policy.json` SHOULD also generate the corresponding signals in other formats as defined in this section, to ensure that the domain owner's preferences are enforceable by agents that do not yet support this specification.

### 8.1 Mapping Table

| ai-policy.json | robots.txt | Content-Signal | TDMRep | ai.txt |
|----------------|-----------|---------------|--------|--------|
| `training: "block"` | `Disallow: /` for GPTBot, CCBot, Google-Extended, Bytespider, anthropic-ai, ClaudeBot, Cohere-ai, Diffbot, FacebookBot, img2dataset, Meta-ExternalAgent, Omgilibot, Timpibot, AI2Bot, Amazonbot, Applebot-Extended | `ai-train=no` | `TDM-Reservation: 1` | `Disallow: Training` |
| `training: "allow"` | No training-specific Disallow directives | `ai-train=yes` | *(no header, or `TDM-Reservation: 0`)* | `Allow: Training` |
| `search: "allow"` | `Allow: /` for ChatGPT-User, OAI-SearchBot, PerplexityBot, YouBot | `search=yes` | *(no equivalent)* | `Allow: Search` |
| `search: "block"` | `Disallow: /` for search-specific bots | `search=no` | *(no equivalent)* | `Disallow: Search` |
| `ai_input: "allow"` | *(no direct robots.txt equivalent)* | `ai-input=yes` | *(no equivalent)* | `Allow: Inference` |
| `ai_input: "block"` | *(no direct robots.txt equivalent)* | `ai-input=no` | *(no equivalent)* | `Disallow: Inference` |

The specific bot user-agent names in the `robots.txt` column are illustrative of the bots known to be in use as of April 2026. This list is not exhaustive and will change over time as AI providers launch new crawlers, rename existing ones, or retire old ones. Tools generating `ai-policy.json` SHOULD maintain an up-to-date list of publicly declared AI crawler user-agent strings rather than treating the names in this table as canonical.

### 8.2 Consistency Model

When signals conflict across formats, the following precedence applies:

1. **robots.txt** is authoritative for path-level access (can the bot visit this URL?).
2. **ai-policy.json** is authoritative for purpose-level permissions (what can the bot do with the content?).
3. **Content Signals / TDMRep** are authoritative for regulatory TDM opt-out (legal signal under the EU DSM Directive).

Tools that generate `ai-policy.json` SHOULD detect inconsistencies between the generated file and existing signals on the domain and alert the domain owner to conflicts.

### 8.3 Relationship to Cloudflare Content Signals Vocabulary

The three permission fields in this specification correspond directly to the Cloudflare Content Signals vocabulary:

| ai-policy.json | Content Signals | Description |
|----------------|----------------|-------------|
| `training` | `ai-train` | Training or fine-tuning AI models |
| `search` | `search` | Building a search index and providing search results |
| `ai_input` | `ai-input` | Inputting content into AI models for real-time generation |

This alignment is intentional. Cloudflare Content Signals have the broadest deployment of any AI-specific permissions mechanism (3.8M+ domains as of April 2026). By aligning vocabulary, `ai-policy.json` ensures interoperability with the existing ecosystem while providing a richer, more structured format.

### 8.4 Signal Translation

When generating `ai-policy.json` from signals in other formats (Content Signals, robots.txt directives, TDMRep headers, ai.txt entries), tools face a semantic gap: other formats often support a three-state model (allow / block / unspecified) while `ai-policy.json` requires an explicit binary choice for each permission.

Tools MUST NOT fabricate permission values for signals not present in the source. Specifically:

- If a domain has `Content-Signal: ai-train=no` but does not specify `search` or `ai-input`, a tool MUST NOT generate an `ai-policy.json` with arbitrary values for `search` and `ai_input`. The domain owner has not expressed a preference on those categories.
- Tools generating `ai-policy.json` SHOULD require explicit declaration of all three permission fields from the domain owner through a policy builder or wizard interface, rather than inferring them from partial signals.
- Tools MAY publish partial `ai-policy.json`-like data in internal systems (such as a permissions registry API) where the distinction between "unspecified" and "allow" or "block" is preserved, but the published file at `/.well-known/ai-policy.json` MUST contain explicit values for all three permission fields.

When reading `ai-policy.json` and translating to another format, omit signals for permissions the source file does not declare; do not translate defaults.

---

## 9. Versioning and Extensibility

### 9.1 Version Field

The `version` field is a string using semantic versioning (`"major.minor"`). This specification defines version `"1.0"`.

- **Minor version increments** (e.g., `"1.1"`) add optional fields. Agents that understand `"1.0"` can safely read `"1.1"` files by ignoring unknown fields.
- **Major version increments** (e.g., `"2.0"`) may change the semantics of existing fields. Agents that only understand `"1.x"` SHOULD read the `permissions` block (which is guaranteed to be present and backward-compatible in all versions) and ignore everything else.

### 9.2 Forward Compatibility

All future versions of this specification are expected to include the `version`, `domain`, and `permissions` fields with the same semantics as version 1.0. This ensures that any agent implementing version 1.0 can read any future version's file and extract a usable (if incomplete) permission set.

New capabilities (action-level permissions, conditional access, agent identity requirements) will be added as new top-level fields that version 1.0 agents ignore. The `capabilities` field is reserved for this purpose.

---

## 10. Relationship to Existing Standards

### 10.1 robots.txt (RFC 9309)

`ai-policy.json` complements robots.txt. robots.txt controls path-level access per bot. `ai-policy.json` controls purpose-level permissions for all access. Both should be present and consistent.

A domain that blocks GPTBot in robots.txt AND sets `training: "block"` in `ai-policy.json` has consistent enforcement across both standards.

A domain that blocks GPTBot in robots.txt but sets `search: "allow"` in `ai-policy.json` is expressing: "I don't want GPTBot specifically, but I do allow AI search in general." This is a valid and common configuration because GPTBot is used for both training and search, and robots.txt cannot distinguish between them.

### 10.2 Cloudflare Content Signals

Content Signals express purpose-level preferences as extensions to robots.txt using the `Content-Signal` directive. `ai-policy.json` provides the same information in structured JSON with additional metadata (bot lists, descriptions, site types, contact information). Both can coexist; tools that generate `ai-policy.json` SHOULD also generate a corresponding `Content-Signal` line.

### 10.3 TDMRep (W3C)

TDMRep provides text and data mining reservation signals via HTTP headers, primarily for EU AI Act compliance. `ai-policy.json` subsumes TDMRep's training signal. Domains using both should ensure consistency: `training: "block"` in `ai-policy.json` should correspond to `TDM-Reservation: 1` in TDMRep headers.

### 10.4 ai.txt

ai.txt expresses similar purpose-level permissions in a text key-value format. `ai-policy.json` provides the same information in structured JSON with additional metadata. Both can coexist; tools that generate `ai-policy.json` SHOULD also generate a corresponding ai.txt for maximum compatibility.

### 10.5 llms.txt

llms.txt is a content structure document that describes what content is available on a domain, formatted for LLM consumption. It is an invitation file, not a permission file. `ai-policy.json` and llms.txt serve different purposes and can coexist without conflict.

### 10.6 RSL (Really Simple Licensing)

RSL provides machine-readable licensing and payment terms for AI content usage. `ai-policy.json` expresses permissions (what is allowed); RSL expresses licensing (under what terms). The two are complementary. A domain might use `ai-policy.json` to declare `training: "block"` while using RSL to define licensing terms for training access should the AI provider wish to negotiate.

### 10.7 Terms of Service

Terms of Service are natural-language documents with legal weight. `ai-policy.json` is a machine-readable expression of the domain owner's intent. The two should be consistent but serve different audiences: ToS for legal proceedings, `ai-policy.json` for automated agent decision-making.

### 10.8 IETF AIPREF Working Group

The IETF AI Preferences Working Group is developing a formal vocabulary for AI usage preferences and mechanisms for expressing those preferences via HTTP headers and robots.txt. `ai-policy.json` aligns with the AIPREF vocabulary where standardized terms exist and will adopt additional terms as the working group finalizes them.

---

## 11. Security Considerations

### 11.1 Authenticity

Version 1.0 does not include a cryptographic signature mechanism. Agents accessing `ai-policy.json` over HTTPS can trust that the file was served by the domain's server (via TLS certificate verification). This provides the same level of trust as robots.txt.

### 11.2 Unauthorized Modification

A file hosted at `/.well-known/ai-policy.json` can be modified by anyone with write access to the web server. This is the same trust model as robots.txt, security.txt, and other well-known files. Domain owners should restrict write access to authorized personnel.

### 11.3 Spoofing

An attacker who controls DNS or performs a man-in-the-middle attack could serve a modified `ai-policy.json`. HTTPS mitigates this risk. Publishers SHOULD serve the file over HTTPS.

### 11.4 Denial of Service

Agents that frequently fetch the file could create excessive load. The caching recommendations in Section 3.3 mitigate this risk. Publishers MAY also use standard HTTP rate limiting.

### 11.5 Sensitive Information

The `contact` field exposes an email address publicly. Domain owners should use a role-based address (e.g., `ai-policy@example.com`) rather than a personal email. The `contact` field is optional; domain owners who do not wish to expose any contact information may omit it.

---

## 12. IANA Considerations

This specification proposes registration of the following well-known URI:

- **URI suffix:** `ai-policy.json`
- **Change controller:** Maango (https://maango.io)
- **Specification document:** This document
- **Status:** Provisional
- **Related information:** The file is a JSON document (`Content-Type: application/json`) containing AI agent permission declarations for the hosting domain.

---

## 13. Examples

### 13.1 Blog That Blocks Training, Allows Search

```json
{
  "version": "1.0",
  "domain": "myblog.com",
  "updated": "2026-04-01T00:00:00Z",
  "site_type": ["blog"],
  "description": "We allow AI agents to discover and cite our articles. AI training on our content is not permitted.",
  "permissions": {
    "training": "block",
    "search": "allow",
    "ai_input": "allow"
  },
  "bots": {
    "blocked": ["CCBot", "Google-Extended", "GPTBot"],
    "allowed": ["ChatGPT-User", "PerplexityBot"]
  },
  "contact": "editor@myblog.com"
}
```

### 13.2 E-commerce Site

```json
{
  "version": "1.0",
  "domain": "shop.example.com",
  "updated": "2026-04-01T00:00:00Z",
  "site_type": ["ecommerce"],
  "description": "AI agents may discover and recommend our products. Training on our product catalog is not permitted.",
  "permissions": {
    "training": "block",
    "search": "allow",
    "ai_input": "allow"
  },
  "bots": {
    "blocked": ["Bytespider", "CCBot", "Google-Extended", "GPTBot", "img2dataset"],
    "allowed": ["ChatGPT-User", "OAI-SearchBot", "PerplexityBot"]
  },
  "contact": "webmaster@shop.example.com"
}
```

### 13.3 Regulated Healthcare Site (Block All)

```json
{
  "version": "1.0",
  "domain": "healthcare.example.com",
  "updated": "2026-04-01T00:00:00Z",
  "site_type": ["regulated"],
  "description": "This site does not permit AI agent access. All forms of AI crawling, training, search indexing, and content consumption are prohibited.",
  "permissions": {
    "training": "block",
    "search": "block",
    "ai_input": "block"
  },
  "contact": "compliance@healthcare.example.com"
}
```

### 13.4 Open Source Community (Allow All)

```json
{
  "version": "1.0",
  "domain": "opensourceproject.org",
  "updated": "2026-04-01T00:00:00Z",
  "site_type": ["community"],
  "description": "This site welcomes AI agents. Content may be used for training, search, and AI input.",
  "permissions": {
    "training": "allow",
    "search": "allow",
    "ai_input": "allow"
  }
}
```

### 13.5 Minimal Valid File

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

---

## 14. Future Extensions

This section is non-normative and describes planned directions for future versions.

### 14.1 Conditional Permissions (Version 1.1)

Version 1.1 is expected to introduce per-permission conditions using a closed vocabulary of machine-verifiable qualifiers. Conditions would be boolean or numeric to ensure deterministic verification. Free-text qualifiers are explicitly excluded from consideration because they cannot be reliably evaluated by automated systems.

### 14.2 Action-Level Permissions and Agent Identity (Version 2.0)

Version 2.0 is expected to introduce trust tiers (such as observe, interact, act) that align with agent infrastructure frameworks emerging in the ecosystem, including projects such as AGNTCY. This would enable domain owners to set differentiated permissions based on agent identity and declared skills, supporting the transition from content-usage permissions to behavior-authorization for agentic AI. Specific vocabulary and schema design will depend on the state of agent-side standards at the time of publication.

These extensions will be additive. A version 1.0 agent reading a version 2.0 file will see the `permissions` block and ignore the rest. The file remains useful at every compatibility level.

---

## 15. References

### Normative References

- **RFC 2119**: Key words for use in RFCs to Indicate Requirement Levels
- **RFC 8259**: The JavaScript Object Notation (JSON) Data Interchange Format
- **RFC 8615**: Well-Known Uniform Resource Identifiers (URIs)
- **RFC 9309**: Robots Exclusion Protocol (robots.txt)

### Informative References

- Cloudflare Content Signals (https://contentsignals.org)
- IETF AIPREF Working Group: AI Usage Preferences Vocabulary (https://datatracker.ietf.org/wg/aipref/about/)
- RSL: Really Simple Licensing 1.0 (https://rslstandard.org/rsl)
- TDMRep: W3C Text and Data Mining Reservation Protocol
- ai.txt specification (https://ai-txt.org)
- llms.txt specification (https://llmstxt.org)
- EU AI Act: Regulation (EU) 2024/1689
- DSM Copyright Directive: Directive (EU) 2019/790
- GPAI Code of Practice: General-Purpose AI Code of Practice (July 2025)
- AGNTCY: Linux Foundation Internet of Agents (https://agntcy.org)
- Google A2A: Agent-to-Agent Protocol (https://google.github.io/A2A)

---

## Changelog

### Version 1.0 (April 2026)

- Initial specification
- Three permission fields: `training`, `search`, `ai_input`
- Bot directive lists (`blocked`/`allowed`)
- Discovery via `/.well-known/ai-policy.json`
- Interoperability mappings to robots.txt, Content Signals, TDMRep, ai.txt
- Versioning and forward compatibility model
- Reserved `capabilities` field for future action-level permissions
