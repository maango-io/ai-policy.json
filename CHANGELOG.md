# Changelog

All notable changes to the `ai-policy.json` specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.0.1] - 2026-04-17

### Fixed

- Schema `$id` URL corrected to match the repository path at `maango-io/ai-policy.json`
- `site_type` field now uses a closed vocabulary with `x-*` extension namespace for custom values (previously accepted any string)
- Section 7.2 `allowed` list semantics simplified; removed ambiguous escape clause that had no corresponding syntax
- HTTP header fallback discovery (Section 3.2) now specifies same-origin requirement
- Added negative caching guidance for 404 responses (Section 3.3)

### Added

- User-Agent matching rules (new Section 7.2) based on RFC 9309 conventions
- Domain Verification security requirement (new Section 11.6) to prevent spoofing attacks
- Extension fields namespace (`x-*`) formally specified in new Section 5.4
- Example `site_type` values expanded: added `news`, `forum`, `media`, `government`, `nonprofit`

### Changed

- Maango-specific fields `registry` and `verification` moved from core schema to `x-maango-registry` and `x-maango-verification` under the new extension fields namespace. This is a breaking change for files using these field names; update Maango-generated files accordingly.
- Section 7 subsection numbering shifted down by one to accommodate the new UA Matching Rules subsection

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
