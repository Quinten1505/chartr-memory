# Knowledge lifecycle discussion

Discussion history for [Set the knowledge lifecycle and authority rules](../tickets/05-knowledge-lifecycle-contract.md). The ticket's Answer holds the canonical resolution, confirmed by the operator after four rounds and a final review.

## Round one: answered

The operator selected each recommended answer on 2026-09-26.

1. **Hard-rule authority:** Constraints and operator preferences remain proposed until the operator explicitly confirms them. A second provider may add evidence but cannot activate them. This reopens draft D7 because another provider can repeat retrieved content without establishing independent authority.
2. **Proposal visibility:** Ordinary bootstrap and recall exclude proposed decisions, constraints, and preferences. An explicit request may return them, clearly labeled as proposals. This resolves the draft's conflict between its “live” status table and `memory_recall`'s `include_proposed: false` default.
3. **Accepted-body changes:** A typo fix may update accepted knowledge only with an audited revision. A change of meaning, scope, or authority requires a new record linked by supersession. This sharpens the draft's ambiguous “cosmetic edit” exception.

## Round two: answered

The operator selected each recommended answer on 2026-09-26.

4. **Agent-derived knowledge:** Attributed agent-derived facts and lessons may enter ordinary retrieval. Decisions require operator acceptance; constraints and preferences still require explicit confirmation.
5. **Replacement lineage:** Each old record has at most one direct successor; one new record may replace several old records. A split explicitly closes the old record and links the new ones without naming a sole successor.
6. **Scoped-key conflicts:** An identical retry for the same type and slug in the same memory scope returns the existing record. Additional evidence is attached separately. A changed meaning conflicts and needs an explicit successor or a new slug.
7. **Publication lifecycle:** Sharing creates a fixed, reviewed snapshot. Changing its content or destination requires review again. Withdrawal stops future retrieval and preserves an audit trail.

## Round three: answered

The operator selected each recommended answer on 2026-09-26.

8. **Fact validity:** A fact receives an explicit validity date when the source supports one, with no invented default expiry. Expired facts leave ordinary retrieval. Conflicting current facts are flagged together for review rather than silently choosing one.
9. **Replacement states:** Every replaceable semantic type gains a terminal `superseded` state; its other lifecycle states remain. A split explicitly closes the old item with a reason and links the new items without claiming a sole successor.
10. **Ontology:** Node and edge types remain closed. An unfitted idea becomes a thought; a proposed new type requires a deliberate schema decision. No similarity threshold automatically creates schema proposals.
11. **Evidence independence:** A provider's repetition of a retrieved memory is derived evidence, not independent corroboration. Separately observed source work can corroborate a claim, but cannot activate a hard rule.

## Round four: answered

The operator selected each recommended answer on 2026-09-26.

12. **Other instruction-like types:** Agent-written conventions and skills start as proposals or drafts and require operator activation. Observed incidents may open automatically; closure requires source evidence or operator confirmation.
13. **Thought promotion:** Preserve the original thought, create a typed target under that type's normal authority rules, allow one current promotion target per thought, and require operator review before widening scope.
14. **Concurrent updates:** Status, body, scope, and successor changes use server revisions. Status and lineage edges commit atomically. Identical retried operations return their prior result; conflicting writes return a revision conflict for an explicit re-read and decision.
