# LCH Wiki Integration

Use this reference whenever the user's private `lch-llm-wiki` repository is available. The wiki is the preferred personal context source for this skill.

## Repository

Target repo: `lch99310/lch-llm-wiki` (private).

Resolve wiki context in this order:

1. **GitHub connector / GitHub repo access** to `lch99310/lch-llm-wiki`.
2. **Local repository fallback**, only if GitHub access is unavailable, not installed, unauthorized, or fails.

When GitHub access works, use it as the source of truth. Fetch the same paths named below from the private repo. Treat `LCH_WIKI_DIR` as the logical wiki root; if a local checkout is needed for `rg` or file-based lookup, clone the private repo into the current workspace under `work/repos/lch-llm-wiki` for read-only use. Do not modify, commit, push, publish, deploy, or expose the wiki.

When GitHub access does **not** work, resolve a local repo in this order:

1. `LCH_LLM_WIKI_DIR` environment variable, if set.
2. `${HOME}/Desktop/SideProjects/lch-llm-wiki`, if it exists.
3. A local folder named `lch-llm-wiki` in or above the current workspace.
4. A local clone under `work/repos/lch-llm-wiki`.

If no source is available, fall back to `persona.md` and `memory.md`, and mention in the final META line that wiki context was unavailable.

## Query Entry Points

When wiki context is available:

1. Read `<LCH_WIKI_DIR>/AGENTS.md` first and obey its private-first rules.
2. Start lookup from `<LCH_WIKI_DIR>/wiki/index.md`.
3. For any personal fact, verify against `<LCH_WIKI_DIR>/wiki/profile/source-of-truth.md`.
4. For writing voice or analysis style, use:
   - `wiki/writing/voice-profile.md`
   - `wiki/writing/key-essays.md`
   - `wiki/product-thinking/product-analysis-process.md`
   - `wiki/product-thinking/rca.md`
   - `wiki/product-thinking/ai-native-product-design.md`
5. For career or positioning questions, use:
   - `wiki/profile/source-of-truth.md`
   - `wiki/profile/persona-summary.md`
   - `wiki/profile/career-strategy.md`
   - relevant pages under `wiki/career/`

Do not load the whole repo. Use `rg` and the index pages to pull only the pages relevant to the user's question.

## Disclosure Gate

The wiki uses disclosure levels in `wiki/profile/source-of-truth.md`:

- `P` = public-safe.
- `R` = resume/interview only.
- `C` = confidential; never external.

Personal lens may use `P`, `R`, and `C` internally to understand fit, constraints, and tradeoffs. User-facing output must obey these rules:

- `P` may be stated directly when relevant.
- `R` may be summarized for private decision support, but do not turn it into public-ready copy.
- `C` must not be surfaced as a specific fact. Only abstract it into constraints, such as "there is a confidential employer-side constraint" or "private career timing affects this decision."
- Never output raw contact details, addresses, IDs, salary, credentials, family details, visa/work-rights specifics, internal employer metrics, internal project names, or private third-party names unless the user explicitly asks to use that exact detail.

## Citation Discipline

For Personal lens output, cite the wiki page path used, not raw private values. Example:

`wiki/profile/source-of-truth.md` -> supports career timeline and disclosure level.

If a claim is marked `Needs confirmation`, label it unconfirmed. Do not smooth conflicts.
