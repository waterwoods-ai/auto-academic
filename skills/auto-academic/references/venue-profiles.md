# Venue Profiles

Reference database for adapting paper format, review rigor, experiment standards, and citation style based on target venue.

## How to Use

When the user provides a venue name:
1. Check if it matches a known profile below
2. If yes, apply the profile settings
3. If no, use WebSearch to research the venue's requirements, then apply the closest profile category

## Profile Categories

### Tier 1: Top-Tier CS/AI Conferences

**Examples:** NeurIPS, ICML, ICLR, ACL, CVPR, AAAI
**Citation style:** Author-year (NeurIPS style) or numbered
**Paper format:** Conference template (typically LaTeX)
**Page limit:** 8-10 pages + unlimited appendix
**Review rigor:** Very high — novelty, significance, clarity, reproducibility
**Experiment expectations:**
- Reproducible code required
- Ablation studies expected
- Statistical significance tests (multiple seeds)
- Comparison against SOTA baselines
- Computational cost reporting

### Tier 2: Top-Tier Journals (Science/Nature family)

**Examples:** Nature, Science, Nature Machine Intelligence, Nature Human Behaviour
**Citation style:** Numbered (Nature style)
**Paper format:** Journal template
**Page limit:** Varies (typically 3000-5000 words + methods)
**Review rigor:** Extremely high — broad significance, methodological rigor
**Experiment expectations:**
- Pre-registration encouraged
- Effect sizes and confidence intervals
- Replication or robustness checks
- Data availability statement

### Tier 3: SSCI/SCI Journals

**Examples:** Computers & Education, Internet and Higher Education, Studies in Higher Education, Higher Education
**Citation style:** APA 7.0 (most common)
**Paper format:** Journal-specific (often Word/LaTeX)
**Page limit:** 6000-10000 words
**Review rigor:** High — theoretical contribution, methodological soundness
**Experiment expectations:**
- Mixed methods welcome
- Survey validation (Cronbach's alpha, CFA)
- Effect sizes (Cohen's d, eta-squared)
- IRB/ethics approval mentioned
- Participant demographics

### Tier 4: Regional/Specialized Journals

**Examples:** Taiwan Journal of TESOL, Journal of Research in Education Sciences (JRES)
**Citation style:** APA 7.0 or journal-specific
**Paper format:** Journal template
**Page limit:** Varies
**Review rigor:** Moderate to high
**Experiment expectations:**
- Context-appropriate methodology
- Local relevance emphasized

### Tier 5: Conference Proceedings

**Examples:** AERA, HERDSA, CHI (short papers), workshop papers
**Citation style:** Varies (APA, ACM, IEEE)
**Paper format:** Conference template
**Page limit:** 4-8 pages
**Review rigor:** Moderate
**Experiment expectations:**
- Preliminary results acceptable
- Work-in-progress framing

## Default Fallback

If venue is unknown and cannot be WebSearched:
- Citation style: APA 7.0
- Format: Markdown + LaTeX
- Review rigor: Tier 3 (SSCI standard)
- Experiment expectations: Tier 3 standard
