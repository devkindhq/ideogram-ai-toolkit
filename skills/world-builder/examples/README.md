# Examples

This directory is intentionally empty at ship time. The pipeline this skill documents
was developed manually, over several sessions, for StoreAlert's mascot ("Bell-Boy") —
but that work happened directly in a project vault, not through this generalized skill,
so there is no worked example here yet that was actually produced by running this skill
end to end. Fabricating one would risk being mistaken for a real result later.

## Adding the first real worked example

The next time this skill is run against the live Ideogram MCP for a real world-building
job, save the result here as `<character-slug>-world.md`, following the two-artifact
pattern the sibling skills use (see `skills/character-model-sheet/examples/` for the
target level of detail):

1. **The locked palette** — paper/primary/secondary/ink-trim/accent hex values, as
   defined per `references/palette-lock.md` before step 1.
2. **Per-step record** — for each of the six pipeline steps actually run: the prompts
   used, whether `custom_model_uri` was set, the `request_id`/`job_id`(s), and the
   `visual_review_status` per `references/batch-tracking.md`.
3. **Contamination-check result**, if step 6 was run — the honest pass/fail read per
   `references/contamination-check.md`, not just "ran the check."
4. **A "what worked" closing note** — same spirit as the other skills' worked examples:
   what held up, what needed a re-prompt, and any family-resemblance calls made per
   `references/character-batch-discipline.md`.

Do not invent a fictional worked example to fill this file in the meantime. Leave this
README as the only file in this directory until a real job has actually been run.
