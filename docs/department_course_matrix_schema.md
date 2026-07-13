# Department-Course Matrix Schema

This note defines the cleaned data structure for the simplified project scope. The goal is to build a department-course matrix for 25 academic departments and use the course-feature columns for cosine similarity, hierarchical clustering, and k-means clustering.

The template file is a blank structural template. It is not a clustering-ready matrix until the course-feature values are populated from recommended-course evidence.

## Final CSV Schema

Recommended template file:

```text
templates/data_collection/department_course_matrix_template.csv
```

Required metadata columns:

| Column | Role | Use in clustering |
| --- | --- | --- |
| `department_id` | Stable department identifier | No |
| `department_name` | Department name | No |
| `broad_field` | Broad academic field label | No |
| `selected_reason` | Short rationale for inclusion in the 25-department purposive sample | No |

Recommended course-feature columns (illustrative):

| Column | Role | Use in clustering |
| --- | --- | --- |
| `calculus` | Calculus or advanced calculus preparation | Yes |
| `geometry` | Geometry preparation | Yes |
| `probability_statistics` | Probability and statistics preparation | Yes |
| `physics` | Physics preparation | Yes |
| `chemistry` | Chemistry preparation | Yes |
| `biology` | Biology or life science preparation | Yes |
| `earth_science` | Earth science preparation | Yes |
| `information` | Information, computing, or AI-related preparation | Yes |
| `economics` | Economics preparation | Yes |
| `ethics` | Ethics or philosophy-related preparation | Yes |
| `other_relevant_courses` | Other relevant recommended courses not captured above | Yes |

Only the course-feature columns should be used as clustering features. Metadata columns must be excluded before computing cosine similarity or fitting clustering models.

The matrix keeps all course-feature columns. Rather than deleting broad common subjects (which the specific-elective subject guide does not actually contain as columns), the main analysis down-weights widely shared courses at analysis time using inverse document frequency (`weight = ln(N / df)`), so that courses appearing across many departments contribute less to the similarity computation.

## Coding Rule

Each department-course cell should use the following binary coding rule:

| Value | Meaning |
| ---: | --- |
| `1` | Listed as a related high-school elective subject in `학과 과목 선택 가이드.xlsx` |
| `0` | Not listed |

If the same department-course pair appears multiple times, keep the value as `1`. Repeated mentions should not be summed, because the matrix represents whether a related elective subject is listed in the guide, not the intensity of recommendation.

## Clustering Input

For cosine similarity, hierarchical clustering, and k-means clustering, the feature matrix should exclude:

- `department_id`
- `department_name`
- `broad_field`
- `selected_reason`

The clustering input should include only numeric course-feature columns (IDF weighting is applied to these at analysis time). It should not include metadata columns or any non-feature variables such as expert consensus / co-grouping scores, admission score feasibility, or candidate-generation variables.

## Validation Expectations

A clustering-ready matrix should satisfy the following checks:

- exactly 25 department rows
- no duplicate `department_id`
- no duplicate `department_name`
- non-empty `broad_field`
- non-empty or documented `selected_reason`
- course-feature values limited to `0` and `1`
- no missing values in course-feature columns
- no department row with all course-feature values equal to `0`
- no course-feature column entirely equal to `0` unless intentionally retained and documented
- no expert consensus, admission score, or candidate-generation columns

The blank template may fail the all-zero row and all-zero column checks by design. Those checks must pass before the matrix is used for cosine similarity or clustering.

## Actual Data Files (moved from README)

The populated data objects in this repository:

- `data/raw/departments_raw.xlsx`: selected department list
- `data/raw/recommended_courses_raw.xlsx`: raw recommended-course evidence
- `data/processed/course_coding_evidence.csv`: cleaned evidence for department-course coding
- `data/processed/department_course_matrix_binary.csv`: baseline binary matrix (25 departments x 89 course features)
- `data/processed/department_course_matrix_idf_weighted.csv`: IDF-weighted matrix (main analysis input)
- `results/tables/keep_for_report/course_idf_weights.csv`: per-course document frequency and IDF weight

Binary coding rule for the main matrix:

| Value | Meaning |
| ---: | --- |
| 1 | Listed as a related high-school elective subject in the subject selection guide (`학과 과목 선택 가이드.xlsx`) |
| 0 | Not listed |

The baseline vector uses raw binary presence, so a near-ubiquitous course such as
확률과 통계 (21 of 25 departments) is as influential as a highly department-specific one.
The main analysis re-weights each course by inverse document frequency,
`weight = ln(N / df)`, which down-weights widely shared courses with no hand-tuned
parameter. The chosen weighting is supported by a sensitivity sweep in
`results/tables/keep_for_report/weighting_sensitivity.csv`.
