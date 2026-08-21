# Invocation Contexts

Load this for a **warm** invocation (SKILL.md Phase 0). The method is one method; warm is a modifier on *where the question comes from* and *how much ceremony is warranted*, not a second workflow.

## Cold vs warm

- **Cold** — the user opens with an explicit external question at session start. Run the full method at the warranted tier.
- **Warm** — `ce-pov` is dropped into a live session ("weigh in", "give me your POV on this") and the question lives in the surrounding conversation, or is absent.

## What warm takes from the conversation: the question only

The conversation supplies the **question** and the **claims-to-verify** — *nothing else*. It is **not** grounding. The biggest failure here is **consensus laundering**: twenty turns of you and the agent mutually assuming "we must migrate off X" quietly becoming "grounding," producing a confident verdict that ratifies chat fiction.

So every input is labeled by provenance, and only verified buckets satisfy the gate (see `references/method.md`):

| Bucket | Counts as grounding? |
|---|---|
| Observed project facts (from a scout dossier or a host bounded read of the authoritative source) | Yes |
| Verified external facts (from a scout dossier or a host bounded read of the authoritative source) | Yes |
| Conversation claims | No — frame and hypotheses until a scout or a bounded inline read of the authoritative source corroborates |
| Unconfirmed assumptions | No — surfaced for the user to confirm or deny |

If the conversation says "we have 40 call-sites on X," the project-grounding scout — or the host's own bounded read, when the sites are already located — must confirm that against the codebase before it counts. **Warm adds no evidentiary weight** — it surfaces the question and hypotheses; the independent grounding is still done by scouts or bounded reads of the source, never by the conversation itself. Same invalidation rule, no warm exemption.

## Establishing the question (frame gate)

A warm invocation with **no explicit question**, or a materially ambiguous one, goes through the frame gate in `references/intake.md` — infer the decision from the conversation, propose/confirm it, then proceed. Rendering a confident POV on the wrong question is the warm-mode failure that gate prevents. **Skip the gate** when the user named the question ("ce-pov: should we use X?") — a mandatory confirm on every warm run is the bureaucratic ritual the skill avoids.

Short references are intentional: "on the approach," "these options," or "the three options presented" resolve from the active conversation when one referent fits. Ask once only when competing referents would materially change the POV. `oracle` and explicit peer names are recognized but no longer convene a panel — cross-model peer review is disabled in this fork (see FORK.md); deliver the solo POV with the disabled-panel note from SKILL.md Phase 3.

A warm summons naming an already-formed position — the host's prior POV or the user's own view — is a prior-opinion subject, not a revision prompt: that position is the subject ce-pov forms its own verdict on. With the panel disabled, that verdict is ce-pov's alone; say so rather than implying a second model weighed in.

## Be more adversarial than cold — operationalized

The conversation's momentum pulls toward agreement, and a second opinion that rubber-stamps is worthless. "More adversarial" is not an attitude; it is two concrete rules:

1. Run an **explicit disconfirming-evidence pass** on each load-bearing conversation claim — try to refute it from the grounded evidence (scout dossiers or bounded inline reads) before accepting it.
2. **Never upgrade a grade on conversation momentum alone** — if the only thing pushing toward Adopt is that the room already wants it, that is not grounding, and the grade does not move.

## Guest output contract

Warm is a guest, not a host:

- Never consult an external peer and never make a proactive panel offer: the cross-model panel is disabled in this fork.
- Output a **POV block only** — no reframing of the host session, no taking over the brainstorm.
- **Hand control back** after the POV.
- **Skip the capture offer** unless the user asks — a mid-session interjection should not push a durable-record decision.
