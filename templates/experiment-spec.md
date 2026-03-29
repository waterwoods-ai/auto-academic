# Experiment Spec: {experiment_id}

## Hypothesis

**ID:** {hypothesis_id}
**Statement:** {hypothesis_text}
**Derived from:** {source — which section of synthesis report}

## Experiment Design

**Type:** {survey_analysis | simulation | ml_training | statistical_test | data_processing | other}
**Description:** {what this experiment does}

## Inputs

| Input | Source | Format |
|-------|--------|--------|
| {input_name} | {where to get it} | {file format / data type} |

## Expected Outputs

| Output | Format | Success Metric |
|--------|--------|---------------|
| {output_name} | {format} | {metric_name operator threshold, e.g., "p < 0.05"} |

## Evaluation Method

**Phase 1 (validity):** {how to check the experiment ran correctly — non-crash, valid output range, reasonable values}
**Phase 2 (claim support):** {how to judge if results support the hypothesis — statistical threshold, effect size, qualitative criterion}

## Implementation Notes

**Language:** {python | r | julia | other}
**Key libraries:** {list}
**Estimated runtime:** {rough estimate}
**Data requirements:** {what data is needed, whether it needs to be generated or acquired}
