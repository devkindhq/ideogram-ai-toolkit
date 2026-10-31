# Post-generation verification checklist

The anti-slop gate scores the prompt. This scores the image that came back. They are
different failure surfaces, and the gate cannot see this one by construction: a board
scored 5/5/4/5/5 going in and came back with six panels instead of nine.

Run this before writing any board up as usable, however well its prompt scored.

## 1. Panel-count integrity

Count the panels the image actually shows against the number asked for. A 3x3 board that
renders as six merged panels has failed structurally, however good the survivors look.
Mark the whole board discard, or usable-with-caveats only if what collapsed was decoration
(borders, captions) and every intended panel is still individually present and readable as
its own thing.

## 2. Ground-colour fidelity

Compare the rendered ground against the specified hex by eye, not from memory of what you
asked for. Light warm neutrals are a known-fragile category and drift twice out of two.
Worth double-checking even when the composition otherwise reads correctly, because a wrong
ground is easy to miss when everything else looks right.

## 3. Subject fidelity per panel

For every panel that specified a subject (a material, a person, an object, a scene), check
that the specified subject is what rendered, not something adjacent. The frequent miss is
not absence, it is the wrong version of the right idea: a person present, but the wrong
kind of person for the claim the brand is making.

## 4. Illustration-versus-photography substitution

If a panel asked for a documentary or photographic register, check whether it rendered as
a photograph or silently became a painterly illustration. The tell is visible brushwork,
painterly light, or an absence of photographic grain where the prompt asked for available
light. This is easy to miss because an illustrated version of the right scene still looks
right at a glance. A board that substitutes illustration for photography has not delivered
what was asked for, even where the mood is close. See the stop-generating rule in
`SKILL.md`: it may be a strong brief for a human photographer rather than a delivered
asset.

## 5. Text and caption legibility

Confirm, do not assume, that caption text is garbled, and read the board by panel position
rather than by what a caption appears to say. If a caption did render legibly, note it as
an unusual success, not as something to rely on.

## Recording the result

Write the outcome into the saved file next to each image, in the same discard / usable /
strongest-of-set language the rest of this family already uses for prompt-side judgments.
This is not a separate report. It is the same per-image verdict, informed by looking at
the actual pixels rather than only at the prompt.
