# Collection Patterns

This file covers the two rules `SKILL.md` points here for before touching collection
creation or deletion: the find-or-create pattern (why you check `list_collections`
before calling `create_collection`) and the destructive-flag confirmation rule (when
`delete=True` and `delete_assets=True` are safe to pass). It also separates what's
actually confirmed about the connected Ideogram MCP's collection tools from what isn't,
per the standing rule against stating third-party API behavior as fact without a
verified source.

## Find-or-create: check before you create

`create_collection` does not enforce unique names. Nothing on the tool stops two calls
with the same `name` argument from succeeding and producing two separate collections
that both display as "Fizzwright." That's not a hypothetical edge case for this skill —
it's the default outcome of the obvious naive implementation, because the collections
a user asks about are almost always ones they expect to already exist: "add this to my
Fizzwright collection" said in a session next week assumes last week's "Fizzwright"
collection is still the only one. A skill that always calls `create_collection` without
checking first will silently fork that collection the second time the user mentions it,
and from then on the user's images are split across two collections with the same name
with no way to tell them apart except by opening each one.

The mitigation: before creating anything, always call `list_collections` and check the
results for a name match against what the user asked for.

- **Exactly one match** → reuse it. Don't create a new one.
- **Zero matches** → call `create_collection(name)`. This is the only case where
  creating is correct.
- **Multiple matches** (same name, or close variants that could plausibly be what the
  user meant) → don't guess which one they mean. Ask the user to disambiguate, and list
  the candidates with enough detail to actually distinguish them — creation date, item
  count, or whatever else the `list_collections` response provides for each candidate —
  so the user isn't asked to pick blind between two entries that look identical.

## Destructive-flag confirmation

Two flags in this tool surface are destructive in a way nothing else here is:
`delete=True` on `remove_images_from_collection` and `delete_assets=True` on
`delete_collection`. Per the tools' own descriptions, both permanently destroy the
underlying image assets rather than just unlinking them from the collection — this is
not reversible the way removing an image from a collection normally is. Without the
flag, both calls only touch the collection's membership; the images themselves are
untouched and can be found again via `get_recent_generations` or another collection.
With the flag, the images are gone.

The rule: never pass either flag without an explicit, action-specific confirmation
obtained in the *current* exchange. A general "yes, go ahead" given earlier in the
conversation about a different action does not satisfy this — confirmation has to be
tied to the specific act of permanently destroying these specific images, asked and
answered in the same exchange as the destructive call.

This matters because the two things a destructive flag can be attached to — "clean up a
collection" and "permanently delete images" — sound similar but aren't. A user asking to
tidy up their collections is not thereby asking to lose the underlying photos, and
treating an earlier unrelated "yes" as covering this specific irreversible action is how
a cleanup request turns into unrecoverable data loss.

**Sufficient confirmation** — the user's own words make the deletion of the assets
explicit, either up front or in response to being asked:
- "Delete the collection and the images too."
- Skill asks "Should I also permanently delete the images, or just remove the
  collection and keep them?" → user answers "Yes, delete the images."

**Not sufficient** — an earlier, differently-scoped "yes" gets stretched to cover this:
- Earlier in the conversation: "Should I organize these into a collection?" → "Yes."
  Later: "Clean up the old one" — with no mention of deleting images. Passing
  `delete_assets=True` here on the strength of the earlier "yes" is wrong; "clean up"
  doesn't specify whether the images should survive, and the earlier confirmation was
  about creating a collection, not destroying one. Ask specifically before deleting
  assets.

## What's confirmed vs. what to verify

**Confirmed (from direct inspection of the live connected MCP's tool schemas):**
`add_images_to_collection` and `get_images_by_collection_id` return `added_count`,
`failed_count`, `next_cursor`, and `cursor` as field names — these are real fields on
the live schema, not assumed from documentation. (Verified by reading the two tools'
own tool-schema description text on the connected Ideogram MCP directly: `add_images_to_collection`'s
description states "The response reports `added_count`, `failed_count`, and per-asset
failure reasons," and `get_images_by_collection_id`'s description states "Pass `cursor`
(taken from a previous response's `next_cursor`) to fetch the next page" — not carried
from the design spec's prose.)

**Unverified / unknown — don't assert these as fact:**
- Pagination page-size defaults (how many results `get_images_by_collection_id` returns
  per call before `next_cursor` appears) aren't documented anywhere findable.
- The precise shape of per-asset failure-reason strings in a partial
  `add_images_to_collection` failure isn't documented either — whether it's a fixed enum,
  free text, or something else.

Don't guess at either of these ahead of time. Read them from the actual response the
first time `add_images_to_collection` or `get_images_by_collection_id` is called in a
session, and report what was actually observed rather than a plausible-sounding assumed
value.

## Why this matters

Both rules above exist because the failure mode they prevent is silent and only shows up
later, when it's harder to fix. A duplicate "Fizzwright" collection doesn't announce
itself as a mistake — it just quietly exists, and the user only discovers it days later
when they go looking for an image they know they filed and can't find it, because it's
sitting in the collection's twin. Destroying image assets on a stretched confirmation is
worse: it's not just inconvenient to fix, it's often not fixable at all. Checking
`list_collections` first, and asking a specific question before passing a destructive
flag, cost one extra tool call or one extra question. Skipping either one costs the user
either a confusing mess to untangle or images they can't get back.
