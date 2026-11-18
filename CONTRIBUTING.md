# Contributing to Upshard Content

Thanks for helping make this better. This repo is the lesson and roadmap data behind [upshard.com](https://www.upshard.com) — see [README.md](README.md) for the file layout and schema before you start.

## Ways to contribute

- **A video pick isn't the best teacher for the concept**: open an issue using "Suggest a better video".
- **A lesson's explanation didn't click**: open an issue using "Report a confusing lesson".
- **Suggest a lesson or roadmap** that doesn't exist yet: open an issue using "Suggest a new roadmap" — don't jump straight to a PR for a whole new roadmap without discussing scope first, since roadmap structure is deliberate and we'd rather align before you write 30 lessons.
- **Fix something small directly**: a typo, a dead link, a wrong fact, a formula error. Go straight to a PR.
- **Add or improve a formula** in `formulas/`.

## Before you open a PR

1. **One lesson, roadmap, or formula per PR.** Small, reviewable PRs merge faster than one PR touching ten files.
2. **Match the existing schema exactly.** See the `_roadmap.yaml`, lesson frontmatter, and formula examples in [README.md](README.md). A lesson file must have:
   - `roadmap`, `node`, and `group` fields that match an existing `_roadmap.yaml` entry (`group` must be one of that roadmap's group slugs, `node` must appear in that group's `nodes` list).
   - A real YouTube `video.id` from a creator credited correctly in `creator`.
   - `takeaways` that are standalone, actionable sentences — not "you will learn about X."
   - `check_questions` that are answerable from the lesson body alone, not from the video.
3. **Pick videos the way we do.** One hand-picked, vetted video, 5–20 minutes, from someone who can actually teach the concept — not the first search result, not a channel trailer, not something behind a paywall or login.
4. **Write the body first-principles.** Explain why the thing exists and how it works under the hood, in your own words. Don't copy explanations, structure, or wording from another roadmap site, course, or docs page — see the originality note in `README.md`. It's fine (expected, even) to have read other roadmaps for topic coverage; it's not fine to reproduce their tree, descriptions, or link lists.
5. **If you're adding a new node to an existing roadmap**, also add it to the right `group.nodes` list and wire its `edges` (prerequisites) in `_roadmap.yaml`. Every edge must point forward — no cycles, no edge from a node to something earlier in its own prerequisite chain.
6. **If you're proposing a new roadmap**, open an issue first (see above). A new `_roadmap.yaml` needs `slug`, `title`, `description`, `level`, `status: draft` (we flip it to `published` after review), `tags`, `groups`, and `edges` — grouped so each group answers one clear question, with roughly 4–5 nodes per group.

## PR checklist

- [ ] Frontmatter fields match the schema in README.md exactly (no typos in field names, no missing required fields).
- [ ] `roadmap` / `node` / `group` in the lesson match `_roadmap.yaml`.
- [ ] Video is real, public, and credited to the actual creator.
- [ ] No content copied from another source — written in your own words, first-principles.
- [ ] `check_questions` are answerable from the lesson body alone.
- [ ] If you added a node: `_roadmap.yaml` groups and edges are updated to include it.

## Review

A maintainer will review for accuracy, originality, and fit with the surrounding roadmap. Expect feedback on explanation clarity even if the facts are correct — the bar here is "would this actually teach someone," not just "is this true."

## Code of conduct

Be respectful and assume good faith. Disagreements about content accuracy or pedagogy are fine and expected; personal attacks aren't.

## License

This repo is licensed under [CC BY-SA 4.0](LICENSE). By contributing, you agree your contribution is licensed under the same terms.
