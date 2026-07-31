# Repository understanding index

This material is a small semantic navigation aid for repeated work in the Warp
client repository. It is not an authoritative description: current code, build
selection, and tests remain authoritative.

## Scope and revision

- **Analysis mode:** Mode A, general repository map.
- **Scope:** the Rust workspace at repository root, with the OSS GUI and headless
  TUI as the concrete default runtimes and internal release channels noted where
  their selection changes navigation.
- **Exclusions:** detailed feature flows, server-side implementation, deployment
  operations, exhaustive crate/test inventories, and historical design rationale.
- **Reflected revision:** `0acfd019d6cd91547f49b4a98ef02935f22873b5`
  on branch `work`, verified 2026-07-31.
- **Working-tree context:** pre-existing changes under
  `crates/warp_graphql_schema/` (`yarn.lock`, `.yarn/`, and `.yarnrc.yml`) were
  present but were not inspected as runtime evidence or included in this analysis.

## Available analyses

- [Repository map](01-repository-map.md) — runtime selection, composition,
  boundaries, concepts, and a task-oriented navigation table.
- No subsystem or task-investigation documents currently exist in this pack.

Known gaps are the closed-source/internal channel-configuration generator,
Warp/Oz server implementations, and runtime validation of platform-specific GUI
paths. The repository establishes client requests and configuration boundaries,
not server routing, retention, billing, entitlement, or deployment behavior.

## How to use and maintain this material

Read the repository map before broad exploration, then load only the subsystem or
investigation documents relevant to the task. Treat these documents as navigation
aids, not as authority. Verify task-relevant claims against current implementation,
build configuration, and tests. Ignore unrelated analyses rather than loading the
whole pack into context.

When documentation conflicts with current code, follow the current code and
correct or record the discrepancy. Update repository-understanding documentation
when a change materially alters architecture, runtime flow, core behavior,
integration boundaries, or the recommended places to inspect. Do not update it
for ordinary implementation details.
