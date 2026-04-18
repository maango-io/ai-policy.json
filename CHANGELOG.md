# Changelog

All notable changes to the `ai-policy.json` specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.0] - 2026-04-15

### Added

- Initial specification for `ai-policy.json`
- Three permission fields: `training`, `search`, `ai_input`, aligned with Cloudflare Content Signals vocabulary
- Bot directive lists (`blocked`/`allowed`) for user-agent-specific access control
- Discovery via `/.well-known/ai-policy.json`
- Interoperability mapping tables to robots.txt, Content Signals, TDMRep, and ai.txt
- Consistency model defining precedence when signals conflict across formats
- JSON Schema for validation (`schema/v1.0.json`)
- Five example files covering common deployment scenarios
- Reserved `capabilities` field for future action-level permissions
- Forward compatibility guarantee: all future versions will include `version`, `domain`, and `permissions`
