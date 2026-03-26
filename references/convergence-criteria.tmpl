# Convergence Criteria

## Inner Loop (Experiments)

The inner loop converges when ALL of the following are true:

1. **Every hypothesis has a verdict**: Each hypothesis is marked as `validated`, `partially_supported`, or `revised`
   - `validated`: experiment results clearly support the hypothesis (e.g., p < 0.05 with meaningful effect size)
   - `partially_supported`: mixed results — some evidence supports, some doesn't. The hypothesis is refined to match what the evidence actually shows.
   - `revised`: evidence contradicts original hypothesis. The hypothesis is rewritten to match the evidence. This is a legitimate scientific outcome, not a failure.

2. **No hypothesis is `pending`**: All experiments have run and produced usable results

3. **Claim-evidence mapping is complete**: Every hypothesis points to specific experiment run(s) with specific metrics

### Inner Loop Does NOT Converge When:
- Any experiment crashed without a successful retry
- Any hypothesis has no experiment results at all
- Results are ambiguous and no interpretation has been committed to

### Safety Cap: 10 Experiment Runs
After 10 total experiment runs (across all hypotheses), force-converge:
- Take best results for each hypothesis
- Mark unresolved hypotheses as `inconclusive` with explanation
- Log warning in process log
- Proceed to WRITE stage

## Outer Loop (Review-Revise)

The outer loop converges when ANY of the following are true:

1. **Accept decision**: Reviewer returns "Accept" (no revision needed)
2. **Minor with no critical issues**: Reviewer returns "Minor Revision" and the convergence-judge determines all issues are non-structural
3. **Safety cap reached**: 5 rounds exhausted — force-finalize

### Escalation Trigger:
Round 3 + decision is still "Major Revision" or "Reject" -> escalate to inner loop re-entry (max 1 escalation)

### Post-Escalation Convergence:
After escalation, the outer loop resumes with a fresh round count starting at the round where escalation was triggered (round 3). The remaining cap is rounds 4-5.
