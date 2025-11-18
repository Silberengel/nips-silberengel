# NKBIP-08 Review and Suggestions

## Critical Issues

### 1. Format Inconsistency (Lines 7, 13, 19, 40+)
**Issue**: The document shows three different formats:
- Line 7: `#bookstr` (with hash)
- Line 13: `book:bible:genesis 2:4-9 | KJV` (single colon after "book")
- Line 19: `book::<book_name>...` (double colon)
- Examples: `[[book::bible:genesis...]]` (double colon inside brackets)

**Suggestion**: Standardize on `book::` (double colon) everywhere. Update:
- Line 7: "a `bookstr` wikilink macro" or "a `book::` wikilink macro"
- Line 13: Change to `[[book::bible:genesis 2:4-9 | KJV]]` to match actual usage

### 2. Missing Link Format Specification (Line 19)
**Issue**: The tag structure description doesn't explain the full format clearly. Examples show `bible:genesis` but it's unclear if "bible" is a namespace/prefix or part of the book name.

**Suggestion**: Add a clear BNF or formal specification:
```
book_wikilink = "[[" "book::" book_reference [ "|" version_list ] "]]"
book_reference = [ namespace ":" ] book_name [ " " chapter_spec [ ":" section_spec ] ] [ "," book_reference ]
chapter_spec = chapter_name_or_number
section_spec = section_name_or_number [ "-" section_name_or_number ]
version_list = version [ "," version ]
```

Clarify that:
- The `book::` prefix is required
- Optional namespace before book name (e.g., `bible:`)
- Multiple references can be comma-separated
- Multiple versions can be comma-separated after the pipe

### 3. Normalization Rules Conflict (Line 21)
**Issue**: States "tags MUST contain lowercase ASCII characters, separated by hyphens" but examples contain spaces, colons, and mixed case.

**Suggestion**: Clarify that:
- The wikilink format (what users write) is human-readable and can contain spaces, colons, mixed case
- Normalization applies to the `d` tags in Nostr events (following NIP-54 rules)
- Clients MUST normalize when searching: convert to lowercase, replace non-letter characters with hyphens
- Add a "Normalization" subsection explaining how `bible:genesis 2` maps to `d` tags like `bible-genesis-2`

### 4. Hierarchical Structure Description (Line 21)
**Issue**: "MUST be only two `30040` levels" is confusing. The hierarchy should be:
- Kind 30040: Book (top level)
- Kind 30040: Chapter (nested within book)
- Kind 30041: Section (within chapter)

**Suggestion**: Rewrite as:
"The structure supports a book (kind `30040`) containing chapters (kind `30040` nested within the book) and sections (kind `30041` within chapters). The book wikilink references this hierarchical structure."

### 5. Missing Event Resolution Specification
**Issue**: The document doesn't explain how clients should resolve `book::bible:genesis` to actual Nostr events.

**Suggestion**: Add a "Resolution Algorithm" section:
1. Parse the wikilink format
2. Normalize book/chapter/section names per NIP-54 rules
3. Construct `d` tag search patterns (e.g., for `bible:genesis 2:4`, search for events with `d` tags matching normalized patterns)
4. Query relays for kind `30040` (books/chapters) and `30041` (sections)
5. Filter by version tag if specified
6. Use `a` tags from book index to find chapters, then sections

Reference NKBIP-01's structure: books have `a` tags listing their chapters, chapters have `a` tags listing their sections.

### 6. Missing Error Handling
**Issue**: No guidance on what to do when:
- Book/chapter/section not found
- Invalid format
- Ambiguous matches
- Multiple matches for same reference

**Suggestion**: Add "Error Handling" section:
- Invalid format: Client SHOULD ignore malformed links
- Not found: Client MAY display link as-is or show error indicator
- Multiple matches: Client SHOULD use web-of-trust, reaction counts, or user preferences to select
- Version not found: Client MAY fall back to another version or show all available versions

### 7. Multiple References Not Explained (Lines 78-96)
**Issue**: Comma-separated multiple sections and versions introduced without explanation in Tag Structure section.

**Suggestion**: Update Tag Structure section (line 19) to mention:
- Multiple book references can be comma-separated
- Multiple versions can be comma-separated after the pipe separator
- When multiple versions specified, client SHOULD return Cartesian product of all references × all versions

### 8. Version Field Format (Line 26)
**Issue**: Includes `npub1...` as example version but doesn't explain why/how this is used.

**Suggestion**: Clarify:
- Version can be any identifier: edition name, translation abbreviation, or pubkey
- If pubkey provided, client SHOULD prioritize content from that author
- Add example: "Version can identify a specific publisher/author via their pubkey (npub...) to differentiate between editions from different sources"

### 9. Typo (Line 11)
**Issue**: "Biblication" should be "Biblical"

**Suggestion**: Fix: "The structure is based upon a combination of Asciidoc `image::` links and Biblical citations..."

### 10. Grammar Error (Line 21)
**Issue**: "If is omitted" should be "If it is omitted"

**Suggestion**: Fix

### 11. Ambiguous Example (Line 82)
**Issue**: `bible:gen 2:4-9, romans 1:1-10` uses "gen" as abbreviation but this isn't explained.

**Suggestion**: Either:
- Use full form in examples: `bible:genesis 2:4-9, bible:romans 1:1-10`
- Or add note: "Abbreviations MAY be used; clients SHOULD handle common abbreviations (e.g., 'gen' for 'genesis')"

### 12. Missing Normalization Examples
**Issue**: No examples showing how human-readable format maps to normalized `d` tags.

**Suggestion**: Add subsection showing:
- `[[book::bible:genesis 2:4 | KJV]]` → searches for `d` tag: `bible-genesis-2` (kind 30040) and `bible-genesis-2-4` (kind 30041)
- `[[book::The Odin Codex:Introduction]]` → searches for `d` tag: `the-odin-codex-introduction`

### 13. Relationship to NIP-54 Unclear (Line 7)
**Issue**: References NIP-54 but doesn't explain how it extends it.

**Suggestion**: Clarify:
"This NKBIP extends NIP-54 wikilinks with a specialized `book::` prefix for referencing hierarchical publications. Unlike standard wikilinks that reference kind `30818` articles by `d` tag, book wikilinks reference kind `30040` (publication indices) and kind `30041` (publication content) events, following the publication structure defined in NKBIP-01."

### 14. Range Resolution Ambiguity (Line 32)
**Issue**: Says client MAY infer ranges from integers if chapter not available, but doesn't specify behavior when chapter IS available.

**Suggestion**: Clarify priority:
1. If chapter event available: use `a` tag list to determine range
2. If chapter event not available and range is numeric: MAY infer integer sequence
3. If range contains non-numeric sections: MUST use chapter's `a` tag list

### 15. Missing Implementation Notes
**Suggestion**: Add section covering:
- Client caching strategies
- Relay query optimization (single query vs multiple)
- Handling updates when publication structure changes
- Backward compatibility with existing wikilink implementations

### 16. Display Format Not Specified
**Issue**: Examples show what to search for, but not how links should be displayed.

**Suggestion**: Add note:
- Standard wikilink display: `[[book::bible:genesis 2:4 | KJV]]` displays as "Genesis 2:4 (KJV)" or user-configurable format
- If version omitted, display without version indicator
- Client MAY customize display format while preserving search semantics

## Minor Improvements

1. **Line 9**: Consider breaking into two sentences for clarity
2. **Line 28**: "preceeding" → "preceding"
3. **Line 30**: "preceeding" → "preceding"
4. **Line 40**: Example uses "DRB" but explanation says "Douay-Rheims Bible" - consider adding full form in first mention
5. **Line 55-58**: Consider making one option the default/recommended behavior
6. **Line 96**: Consider ending document with a "See Also" section referencing NKBIP-01 and NIP-54

## Suggested Structure Reorganization

Consider reorganizing as:
1. Abstract
2. Specification
   - Link Format
   - Normalization Rules
   - Event Resolution Algorithm
   - Multiple References and Versions
3. Examples
4. Error Handling
5. Implementation Notes
6. Display Format

This would make the specification more systematic and easier to implement.

