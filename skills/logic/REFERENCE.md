# Logic Reference

Read only the section needed for the current task. Each section is self-contained.

## 1. Valid deductive forms

- **Modus ponens:** If p then q. p. Therefore q.
- **Modus tollens:** If p then q. Not q. Therefore not p.
- **Hypothetical syllogism:** If p then q. If q then r. Therefore if p then r.
- **Disjunctive syllogism:** p or q. Not p. Therefore q.
- **Categorical syllogism:** All M are P. All S are M. Therefore all S are P.

## 2. Common formal fallacies

| Fallacy | Form | Example | Repair |
|---|---|---|---|
| Affirming the consequent | If p then q. q. Therefore p. | If it rains, the street is wet. The street is wet. Therefore it rained. | Require p or rule out other causes of q. |
| Denying the antecedent | If p then q. Not p. Therefore not q. | If you study, you pass. You did not study. Therefore you fail. | Show that p is the only way to get q. |
| Undistributed middle | All A are C. All B are C. Therefore all A are B. | All dogs are mammals. All cats are mammals. Therefore all dogs are cats. | Restate with a shared middle term that actually distributes. |
| Quantifier shift | Every person has a dream. Therefore there is one dream everyone shares. | contests the scope of every and there is. | Keep quantifier order explicit with forall and exists. |

## 3. Common informal fallacies

| Fallacy | Signal | Example | Repair |
|---|---|---|---|
| Ad hominem | Attacks the person | You are biased, so your data is wrong. | Evaluate the claim and evidence, not the speaker. |
| Straw man | Restates opponent weakly | Critics of transit just hate freedom. | Quote the strongest version of the opposing view. |
| False cause | Correlation as causation | Sales rose after the logo change, so the logo caused it. | Control for confounders and show mechanism. |
| Appeal to authority | Authority outside domain | A celebrity chef endorses this crypto fund. | Cite domain experts plus primary evidence. |
| Appeal to popularity | Many believe it | Most people agree, so it must be true. | Replace vote count with reasons and data. |
| False dilemma | Only two options | Either full surveillance or total chaos. | Add the missing middle options. |
| Slippery slope | Unshown chain | This policy will inevitably lead to collapse. | Prove each step or bound the claim. |
| Begging the question | Conclusion in premises | This law is just because just laws must be obeyed. | Provide independent support for the premise. |
| Equivocation | Term shifts meaning | Banks hold money, rivers have banks, so rivers hold money. | Fix one definition per term for the whole argument. |
| Hasty generalization | Small or biased sample | Three users loved it, so all users will. | Report sample size, selection, and counterexamples. |

## 4. Formalization mini-guide

Use propositional variables p, q, r and connectives: not `~`, and `/\`, or `\/`, implies `->`.

- Translate: "If the test is flaky, CI is slow" becomes `p -> q`.
- First-order: "All services log errors" becomes `forall x. Service(x) -> LogsErrors(x)`.
- Truth tables decide tautology, contradiction, or contingency for propositional forms. Build one when validity is disputed and the form has at most 3 variables.
- Keep quantifier order. `forall x. exists y.` is not the same as `exists y. forall x.`.

## 5. Inductive strength checklist

- Sample size and selection method stated.
- Representativeness addressed.
- Rival generalizations considered.
- Confidence stated as probable, not certain.
- One counterexample that would weaken the claim named.

## 6. Abductive comparison template

For observation O and rival hypotheses H1, H2:

- Which explains more of O with fewer extra assumptions.
- Which predicts something checkable next.
- What evidence would change the ranking.
