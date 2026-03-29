# Failure Paths

## Deep Research Failure Paths

### Overview

This document lists all failure scenarios that may be encountered across all modes of the deep-research skill, along with their detection conditions, user notification messages, handling steps, and recovery paths. The purpose is to ensure every failure scenario has a clear handling strategy, preventing users from reaching a dead end.

---

### Failure Path Summary

| # | Failure Scenario | Affected Modes | Severity | Handling Strategy |
|---|---------|---------|---------|---------|
| F1 | RQ cannot converge | full, socratic | Medium | Narrow scope / provide candidate RQs |
| F2 | Insufficient literature | full, quick, lit-review | High | Expand search strategy |
| F3 | Methodology mismatch | full | High | Return to Phase 1 |
| F4 | Devil's Advocate CRITICAL | full | Critical | STOP + correct |
| F5 | Ethics BLOCKED | full, review | Critical | STOP + remediation path |
| F6 | Socratic dialogue does not converge | socratic | Medium | Switch to full mode |
| F7 | User abandons mid-process | all | Low | Save progress |
| F8 | Only Chinese-language literature available | full, lit-review | Medium | Switch search strategy |
| F9 | All source quality below threshold | full, fact-check | High | Downgrade or expand sources |
| F10 | Conclusions inconsistent with evidence | full | High | Return to Phase 3 |
| F11 | Revision loop exceeds limit | full | Medium | Force-complete + limitation list |
| F12 | Interdisciplinary bridging failure | full | Low | Revert to single discipline |

---

### Detailed Failure Paths

#### F1: Research Question Cannot Converge

**Affected Modes**: `full` (Phase 1), `socratic` (Layer 1)
**Severity**: Medium

**Trigger Conditions**:
- `full` mode: research_question_agent interaction exceeds 3 rounds, user still cannot determine the RQ
- `socratic` mode: Layer 1 exceeds 5 rounds, user repeatedly revises without a clear direction

**User Notification Message**:
> I notice we've been discussing for a while, but the research question hasn't converged to a clear direction yet. This is perfectly normal — sometimes the question itself is the hardest part. Let me offer a few possible directions to see which one is closest to your thinking.

**Handling Steps**:
1. Compile key topics discussed and user-expressed preferences
2. Produce 3 candidate RQs, each with a brief explanation and rough FINER assessment
3. Ask the user to select the closest one as a starting point
4. If the user still cannot choose → suggest doing a `lit-review` mode to explore the literature first, then return

**Recovery Paths**:
- Select a candidate RQ → continue the original workflow
- Do lit-review → restart RQ clarification after the literature review is complete
- User redescribes on their own → restart Phase 1 / Layer 1

---

#### F2: Insufficient Literature

**Affected Modes**: `full` (Phase 2), `quick`, `lit-review`
**Severity**: High

**Trigger Conditions**:
- bibliography_agent finds < 5 usable sources after standard search strategy
- After excluding quality-unqualified sources, < 3 remain

**User Notification Message**:
> With the current search strategy, I found only limited relevant literature. This could mean: (1) this is a very new research area; (2) the search keywords need adjustment; (3) the research question scope may need refinement. Let me try expanding the search strategy.

**Handling Steps**:
1. Expand search keywords (synonyms, broader terms, related concepts)
2. Expand database scope (add grey literature, policy reports, working papers)
3. Relax time range (from past 5 years to past 10 years)
4. Try keywords from adjacent disciplines
5. If still insufficient → suggest the user consider adjusting the RQ or accept this as an exploratory study

**Recovery Paths**:
- Expanded search yields sufficient literature → continue original workflow
- Accept as exploratory research → adjust report positioning, emphasize the study's pioneering nature
- Adjust RQ → return to Phase 1

---

#### F3: Methodology Mismatch

**Affected Modes**: `full` (Checkpoint 1)
**Severity**: High

**Trigger Conditions**:
- devils_advocate_agent at Checkpoint 1 determines that the methodology proposed by research_architect_agent cannot answer the RQ produced by research_question_agent
- There is a logical gap between the methodology and the RQ

**User Notification Message**:
> Devil's Advocate found an important issue in the methodology review: your research question asks "why," but your method design can only answer "whether." Let's go back and adjust — here are three possible directions...

**Handling Steps**:
1. Clearly state the gap between the RQ type (descriptive/comparative/causal/evaluative) and the method's capability
2. Provide 3 alternative method suggestions, each with pros and cons
3. Confirm whether the RQ needs adjustment to match a feasible method
4. Re-execute research_architect_agent

**Recovery Paths**:
- Select an alternative method → regenerate Methodology Blueprint → Checkpoint 1 re-review
- Adjust RQ → return to research_question_agent → redo Phase 1
- Maximum 2 retries; if still mismatched on the 3rd attempt → suggest the user consult their advisor

---

#### F4: Devil's Advocate CRITICAL

**Affected Modes**: `full` (any Checkpoint)
**Severity**: Critical

**Trigger Conditions**:
- devils_advocate_agent finds a Critical severity issue at any Checkpoint
- Includes: fatal logical flaws, core assumptions that cannot hold, evidence contradicting conclusions

**User Notification Message**:
> STOP — Devil's Advocate found a critical issue that must be resolved before continuing:
> [Specific issue description]
> This is not an issue that can be ignored, as it fundamentally affects the research's validity.

**Handling Steps**:
1. Fully present the Critical issue's description, impact, and suggested correction direction
2. Pause the workflow; do not allow advancement to the next Phase
3. Wait for user response or correction
4. After user correction → re-execute the Checkpoint
5. 2 consecutive CRITICALs → suggest the user fundamentally rethink the research direction

**Recovery Paths**:
- User corrects the issue → re-execute Checkpoint → continue after PASS
- User chooses to modify the RQ/method → return to the corresponding Phase
- User abandons the direction → enter F7 workflow

---

#### F5: Ethics BLOCKED

**Affected Modes**: `full` (Phase 5), `review`
**Severity**: Critical

**Trigger Conditions**:
- ethics_review_agent determines BLOCKED
- Includes: research involving non-consensual use of personal data, potentially discriminatory impact, dual-use risk

**User Notification Message**:
> Ethics Review has determined that this research has ethical issues requiring prior resolution:
> [Specific issue list]
> The research report cannot be delivered until these issues are resolved. Here are the suggested remediation paths...

**Handling Steps**:
1. List all BLOCKED reasons, each with specific remediation suggestions
2. Distinguish between remediable (e.g., add informed consent statement) and irremediable (e.g., research design inherently has ethical issues)
3. Remediable issues → provide modification suggestions → re-review after user confirmation
4. Irremediable issues → suggest fundamental redesign of the research

**Recovery Paths**:
- Fix ethical issues → re-execute ethics_review_agent → continue after CLEARED
- Modify research design → return to Phase 1
- Issues are irremediable → suggest abandoning this research direction, provide alternative direction suggestions

---

#### F6: Socratic Dialogue Does Not Converge

**Affected Modes**: `socratic`
**Severity**: Medium

**Trigger Conditions**:
- Dialogue exceeds 10 rounds, user still vacillating between different directions
- Extracted INSIGHTs < 3

**User Notification Message**:
> We've explored several directions, each with its own value. However, I notice we've been going back and forth between a few directions — this may mean you have multifaceted interests in this topic, but it can also make the research hard to focus. Would you like to:
> (A) Continue the Socratic dialogue, but focus on [the most promising direction] you just mentioned?
> (B) Switch to full mode, and let my team help you systematically explore and converge?
> (C) Take a pause, think it over, and come back later?

**Handling Steps**:
1. Compile currently extracted INSIGHTs
2. Identify the 1-2 directions with the most convergence potential
3. Provide 3 options (continue with focus / switch mode / pause)
4. If user chooses to continue but still hasn't converged by round 15 → auto-compile + end

**Recovery Paths**:
- Continue with focus → restrict discussion scope, converge within 5 rounds
- Switch to full mode → pass extracted INSIGHTs to research_question_agent
- Pause → save INSIGHT list; user can re-enter at any time

---

#### F7: User Abandons Mid-Process

**Affected Modes**: All modes
**Severity**: Low

**Trigger Conditions**:
- User explicitly states they don't want to continue ("never mind," "I quit," "too complicated")
- User abandons after a long period of non-response

**User Notification Message**:
> No problem. I've saved the current progress. Here's a summary of what we've accomplished so far:
> [List of completed stages and outputs]
> You can re-enter at any time by saying "continue the previous research," and we'll pick up where we left off.

**Handling Steps**:
1. Save all outputs from the current stage (RQ Brief, INSIGHTs, Bibliography, etc.)
2. Produce a progress summary
3. Provide instructions for re-entry

**Recovery Paths**:
- User says "continue the previous research" → load saved outputs, continue from where interrupted
- User starts over → entirely new workflow

---

#### F8: Only Chinese-Language Literature Available

**Affected Modes**: `full` (Phase 2), `lit-review`
**Severity**: Medium

**Trigger Conditions**:
- English academic database searches (Web of Science, Scopus, PubMed, etc.) yield empty or very few results
- The topic is strongly localized (e.g., Taiwan-specific policy, regulations, institutional systems)

**User Notification Message**:
> English-language literature on this topic is very limited, but Chinese-language literature resources are abundant. I will adjust the search strategy to include Chinese academic databases. Please note that citation conventions for Chinese-language literature in international publications may differ.

**Handling Steps**:
1. Switch search strategy to Chinese academic databases (Airiti Library, National Digital Library of Theses and Dissertations in Taiwan, CNKI)
2. Re-search using Chinese keywords
3. Note the language distribution of the literature in the report
4. If the user needs an English report → provide suggestions for English citation format of Chinese literature
5. If the user needs to publish internationally → suggest finding comparable international cases

**Recovery Paths**:
- Chinese literature is sufficient → continue workflow with clear language annotations
- User needs international publication → suggest adjusting RQ to add a comparative perspective

---

#### F9: All Source Quality Below Threshold

**Affected Modes**: `full` (Phase 2), `fact-check`
**Severity**: High

**Trigger Conditions**:
- source_verification_agent rates all found sources as Level V or below
- No peer-reviewed sources

**User Notification Message**:
> The overall quality of currently found sources is low, lacking high-quality peer-reviewed research. This may indicate an emerging field, or the search strategy may need adjustment. I suggest we consider...

**Handling Steps**:
1. Expand source types (add policy reports, white papers, official statistics)
2. Lower the threshold but clearly annotate quality levels
3. Reposition the report as "preliminary exploration" rather than "systematic review"
4. Add an "Evidence Quality Limitations" section to the report

**Recovery Paths**:
- Find sufficient alternative sources → continue workflow with clear quality annotations
- Cannot find qualified sources → suggest the user consider conducting primary research

---

#### F10: Conclusions Inconsistent with Evidence

**Affected Modes**: `full` (Phase 5, Checkpoint 3)
**Severity**: High

**Trigger Conditions**:
- editor_in_chief_agent or devils_advocate_agent finds in Phase 5 that report conclusions exceed the scope supported by the evidence

**User Notification Message**:
> The review found that some conclusions in the report go beyond what the evidence supports. Specifically:
> [List of issues]
> I will return for revision to ensure every conclusion has corresponding evidence support.

**Handling Steps**:
1. Flag all "over-inferred" conclusions
2. For each flag: (a) weaken the conclusion to match the evidence, or (b) supplement with additional evidence
3. Re-execute Checkpoint 3

**Recovery Paths**:
- Revision successful → complete Phase 6
- Issues remain after revision → 2nd revision round
- Issues remain after 2 revisions → convert issues to a "Research Limitations" section

---

#### F11: Revision Loop Exceeds Limit

**Affected Modes**: `full` (Phase 6)
**Severity**: Medium

**Trigger Conditions**:
- Phase 6 revision has been executed 2 times (maximum), with unresolved Major issues remaining

**User Notification Message**:
> After two rounds of revision, the following issues have been resolved: [resolved list]. However, the following issues remain unresolved due to inherent research limitations: [unresolved list]. These will be listed in the "Acknowledged Limitations" section. The report is now the best version achievable under current conditions.

**Handling Steps**:
1. Compile resolved and unresolved issues
2. Convert unresolved Major issues into the "Acknowledged Limitations" section
3. Deliver the final report

**Recovery Paths**:
- User accepts → deliver the report
- User does not accept → suggest redesigning the research from Phase 1

---

#### F12: Interdisciplinary Bridging Failure

**Affected Modes**: `full`
**Severity**: Low

**Trigger Conditions**:
- synthesis_agent attempts interdisciplinary integration but cannot find meaningful connections
- Conceptual frameworks from different disciplines cannot be reconciled

**User Notification Message**:
> I attempted to integrate perspectives from [Discipline A] and [Discipline B], but these two disciplines' understanding frameworks for this phenomenon differ substantially. Forcing integration may actually blur the focus. I suggest we center on the [primary discipline] framework, and mention other disciplines' perspectives in the discussion section as reference.

**Handling Steps**:
1. Select the primary disciplinary framework as the analytical foundation
2. Present other disciplinary perspectives in an "Alternative Perspectives" or "Interdisciplinary Insights" section
3. Do not force integration of irreconcilable frameworks

**Recovery Paths**:
- Focus on a single framework → continue workflow
- User insists on interdisciplinary → suggest switching to mixed-methods or narrative review

---

## Academic Paper Failure Paths

This document records the failure scenarios that the academic-paper skill may encounter at each stage, their trigger conditions, and handling strategies. All agents should refer to this guide when they detect a failure scenario.

---

### Failure Path Overview

| # | Failure Scenario | Trigger Condition | Severity | Handling Strategy |
|---|---------|---------|--------|---------|
| F1 | Insufficient research foundation | Plan mode Step 0 finds no RQ / no data | High | Recommend running `deep-research` first |
| F2 | Wrong paper structure selected | structure_architect finds RQ-structure mismatch | Medium | Return to Phase 2, suggest alternative structures |
| F3 | Severely over word count | Draft exceeds target word count by 30% or more | Medium | Identify sections to cut, suggest condensing |
| F4 | Severely under word count | Draft is 30% or more below target word count | Medium | Identify sections to expand, suggest additions |
| F5 | Citation format entirely wrong | citation_compliance finds > 50% format errors | High | Completely re-run citation phase |
| F6 | Poor bilingual abstract quality | Chinese and English abstracts have inconsistent logic | Medium | Re-run abstract_bilingual |
| F7 | Peer review rejection | peer_reviewer issues a Reject verdict | High | Analyze rejection reasons, recommend major revision or restructuring |
| F8 | Plan mode does not converge | > 15 rounds of dialogue without completing all chapters | Medium | Suggest switching to outline-only mode |
| F9 | Incomplete handoff materials | From deep-research but missing key materials | Low | List missing items, suggest supplementing or re-running |
| F10 | User abandons midway | Explicitly states unwillingness to continue | Low | Save completed Chapter Plan |
| F11 | Desk-reject | Journal editor rejects without sending to reviewers | High | Classify rejection cause, select recovery strategy |
| F12 | Conference-to-journal conversion failure | Conference paper expansion to journal article rejected | Medium | Ensure 30-50% new content + proper citation |

---

### Detailed Handling Strategies

#### F1: Insufficient Research Foundation

**Trigger Timing**: Plan mode Step 0 (Research Readiness Check) or Full mode Phase 0

**Detection Indicators**:
- User cannot describe their research question in one sentence
- No literature foundation
- No concept of research methods
- Topic is too broad and cannot be focused

**Handling Process**:
```
1. Affirm the user's research interest
2. Specifically explain what is currently missing
3. Recommend using deep-research (socratic mode)
4. Explain that they can come back to continue after deep-research is completed
5. If the user insists on continuing, switch to outline-only mode (low risk)
```

**Response Template**:
```
Your research topic is very interesting, but I notice that a clear research question
and literature foundation are still missing.

I recommend you first use the deep-research tool to:
1. Systematically search and organize relevant literature
2. Focus on a researchable question
3. Gain a preliminary understanding of possible research methods

Once completed, bring the materials back and we can produce a high-quality paper
more efficiently.
```

---

#### F2: Wrong Paper Structure Selected

**Trigger Timing**: Phase 2 (structure_architect_agent)

**Detection Indicators**:
- RQ is a causal question but a Literature Review structure was selected
- No data but IMRaD was selected
- Topic suits a Case Study but Policy Brief was selected
- Word count target and structure are mismatched (e.g., 3000-word IMRaD)

**Handling Process**:
```
1. Point out the mismatch between RQ and structure
2. Explain why they are mismatched
3. Suggest 1-2 alternative structures
4. Explain how the alternative structures better answer the RQ
5. Return to Phase 2 to let the user re-select
```

---

#### F3: Severely Over Word Count

**Trigger Timing**: Phase 4 (after draft_writer_agent completes)

**Detection Indicators**:
- Actual word count > target word count x 1.3

**Handling Process**:
```
1. List actual word count vs. target word count for each chapter
2. Identify the most over-count chapters
3. Suggest reduction strategies:
   a. Merge duplicate arguments
   b. Condense the literature review (keep core literature)
   c. Remove overly detailed method descriptions
   d. Compress repeated literature dialogue in Discussion
4. Do not proactively delete; let the user decide
```

---

#### F4: Severely Under Word Count

**Trigger Timing**: Phase 4 (after draft_writer_agent completes)

**Detection Indicators**:
- Actual word count < target word count x 0.7

**Handling Process**:
```
1. List actual word count vs. target word count for each chapter
2. Identify the most deficient chapters
3. Suggest expansion strategies:
   a. Increase the depth and breadth of the literature review
   b. Add more evidence and examples
   c. Expand Discussion (more literature dialogue)
   d. Add more detail to methodology descriptions
4. Provide specific expansion directions
```

---

#### F5: Citation Format Entirely Wrong

**Trigger Timing**: Phase 5a (citation_compliance_agent)

**Detection Indicators**:
- Citation format error rate > 50%
- Systematic errors (e.g., all missing DOIs, all using wrong format)

**Handling Process**:
```
1. Analyze error patterns (systematic vs. scattered)
2. If systematic errors:
   a. Identify root cause (user may have selected the wrong citation format)
   b. Confirm the correct citation format
   c. Completely re-run citation phase
3. If scattered errors:
   a. Fix one by one
   b. Produce a correction report
```

---

#### F6: Poor Bilingual Abstract Quality

**Trigger Timing**: Phase 5b (abstract_bilingual_agent)

**Detection Indicators**:
- Chinese and English abstracts cover different key points
- One language version omits important findings
- Keywords do not correspond between Chinese and English
- Word count seriously deviates from standards

**Handling Process**:
```
1. Compare the structure and coverage of Chinese and English abstracts
2. List inconsistencies
3. Rewrite based on the actual paper content as the standard
4. Ensure both versions are independently written but cover the same key points
```

---

#### F7: Peer Review Rejection

**Trigger Timing**: Phase 6 (peer_reviewer_agent issues Reject)

**Detection Indicators**:
- Two or more of the five dimensions scored below 60
- Fatal flaws exist (logical breakdowns, missing core evidence, serious methodology flaws)

**Handling Process**:
```
1. List all issues flagged as Critical
2. Classify the nature of the problems:
   a. Fixable (writing, formatting, minor logic issues) → Recommend Major Revision
   b. Structural issues (argument architecture needs reorganization) → Return to Phase 3 for restructuring
   c. Fundamental issues (RQ infeasible, insufficient data) → Return to Phase 0 for re-evaluation
3. Produce a revision roadmap
4. Execute revision after user confirmation
```

**Note**: If still Reject after 2 rounds of revision, recommend the user to:
- Consult domain experts
- Rethink the research design
- Consider switching target journals (lower the bar)

---

#### F8: Plan Mode Does Not Converge

**Trigger Timing**: Plan mode dialogue exceeds 15 rounds

**Detection Indicators**:
- User repeatedly modifies the direction of the same chapter
- Unable to make definitive decisions
- Discussion drifts off the paper topic

**Handling Process**:
```
1. Pause and summarize what has been determined so far
2. List completed and uncompleted chapters
3. Provide two options:
   a. Jump to outline-only mode (directly produce an outline)
   b. Continue dialogue (but narrow the scope of each discussion)
4. Save the completed Chapter Plan
```

---

#### F9: Incomplete Handoff Materials

**Trigger Timing**: intake_agent detects deep-research materials but they are incomplete

**Detection Indicators**:
- Has RQ but missing Annotated Bibliography
- Has Bibliography but missing Synthesis Report
- Has INSIGHT Collection but some INSIGHTs are incomplete

**Handling Process**:
```
1. List received and missing materials
2. Assess the impact of missing materials:
   a. Missing Bibliography → Need Phase 1 (literature_strategist)
   b. Missing Synthesis → Can continue, Phase 3 handles it additionally
   c. Missing Methodology Blueprint → Need Phase 0 supplementary questions
3. Recommend:
   a. Return to deep-research to complete the missing parts
   b. Or supplement within academic-paper (add Phase 0 interview questions)
```

---

#### F10: User Abandons Midway

**Trigger Timing**: User explicitly states unwillingness to continue

**Detection Indicators**:
- "Forget it" / "Not writing anymore" / "Too complicated" / "Let me think about it"
- Abandons after prolonged unresponsiveness

**Handling Process**:
```
1. Respect the user's decision
2. Save all completed outputs:
   - Paper Configuration Record
   - Chapter Plan (completed portions)
   - INSIGHT Collection
   - Any completed draft sections
3. Inform the user they can come back anytime with these materials to continue
4. Do not actively persuade them to continue (but encouragement is fine)
```

**Save Format**:
```markdown
## Academic Paper — Saved Record

**Topic**: {topic}
**Progress**: Phase {N} / Step {M}
**Completed**:
- [x] Paper Configuration Record
- [x/partial] Chapter Plan (completed {N}/{total} chapters)
- [ ] Draft
- [ ] Citation check
- [ ] Peer review

**How to Resume**: Bring this record and restart academic-paper; can continue from Phase {N}
```

---

### Relationships Between Failure Paths

```
F1 (Insufficient research foundation) → Recommend deep-research → May encounter F9 (incomplete materials) upon return
F2 (Wrong structure) → Return to Phase 2 → May cascading affect F3/F4 (word count issues)
F5 (All citations wrong) → May be a downstream effect of F2 (wrong format selected)
F7 (Rejection) → Analysis may require returning to F2 (structure) or F1 (foundation)
F8 (Non-convergence) → May evolve into F10 (abandonment)
```

#### F11: Desk-Reject Recovery

**Trigger**: Editor rejects the paper without sending to reviewers.

**Cause Classification & Recovery**:

| Cause | Diagnostic Signs | Recovery Strategy |
|-------|-----------------|-------------------|
| **Scope Mismatch** | Editor states "outside journal scope" or "not aligned with journal aims" | Re-analyze journal scope using `top_journals_by_field.md`; identify 3 alternative journals; may need to reframe the paper's contribution |
| **Insufficient Novelty** | "Incremental contribution" or "well-established findings" | Strengthen the novelty claim in introduction; consider additional analysis or a new dataset; reposition the paper's unique contribution |
| **Formatting Non-Compliance** | Immediate rejection for template/length/style violations | Review target journal's author guidelines; use `formatter_agent` to reformat; resubmit (often same journal accepts after formatting fix) |
| **Poor Opening** | No specific reason given; likely the abstract/introduction failed to hook | Rewrite abstract with the CARS model (Create A Research Space); lead with the gap, not the background; have `peer_reviewer_agent` evaluate the new opening |

**General Protocol**:
1. Do NOT take desk-reject personally — 30-50% of submissions to top journals are desk-rejected
2. Read the editor's email carefully for any specific feedback
3. Determine the cause category above
4. If Scope Mismatch: pivot journal, not paper
5. If Novelty/Opening: revise paper, then resubmit (different journal recommended)
6. Turnaround target: 2 weeks for reformatting, 4 weeks for substantive revision

---

#### F12: Conference-to-Journal Conversion Failure

**Trigger**: Attempt to expand a published conference paper into a journal article fails review.

**Common Rejection Reasons & Solutions**:

| Reason | Solution |
|--------|----------|
| **Insufficient Extension** (< 30% new content) | Journal expects 30-50% new material beyond the conference version. Add: extended related work, additional experiments/data, deeper analysis, new discussion sections |
| **Self-Plagiarism Flag** | Explicitly cite the conference version in the introduction: "This paper extends our previous work [conf-citation] with..." Use iThenticate to verify < 30% text overlap |
| **Stale Results** | If the conference paper is > 2 years old, results may be outdated. Update experiments with current data/baselines; acknowledge temporal limitations |
| **Missing Journal Standards** | Conference papers often lack: detailed methodology, reproducibility information, limitations section, broader impact discussion. Add all of these |

**Conversion Checklist**:
- [ ] Conference version explicitly cited in introduction
- [ ] 30-50% genuinely new content added (not just padding)
- [ ] Text overlap with conference version < 30% (verified by similarity tool)
- [ ] All reviewer expectations for a journal-length paper met
- [ ] Notation in cover letter: "This is an extended version of [conference paper]"
- [ ] Check journal policy: some journals prohibit conference-to-journal conversion

---

### Preventive Measures

| Failure Path | Preventive Measure |
|---------|---------|
| F1 | Phase 0 / Step 0 strictly checks research readiness |
| F2 | structure_architect cross-validates the match between RQ and structure |
| F3/F4 | draft_writer checks word count progress after completing each section |
| F5 | draft_writer uses the correct format during writing |
| F6 | abstract_bilingual writes independently based on the paper content as the standard |
| F7 | argument_builder stress-tests arguments in Phase 3 |
| F8 | socratic_mentor sets a dialogue cap per chapter |
| F9 | intake_agent performs a complete materials check when detecting a handoff |
| F10 | Maintain dialogue rhythm to avoid user fatigue |
| F11 | Phase 7 researches target journal scope when producing the cover letter; format_agent strictly follows formatting rules |
| F12 | intake_agent detects whether this is a conference paper expansion; calculate new content ratio early |
