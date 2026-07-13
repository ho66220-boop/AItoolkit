# 권장과목 기반 학과 군집화와 전문가 검증

*Clustering university departments by recommended high-school course profiles, validated against 14 admission consultants' card sort.*

고등학교 **권장과목 데이터만으로** 25개 학과의 유사 구조를 군집화하고, 그 결과를
현직 입시 컨설턴트 **14명의 카드소팅으로 정량 검증**한 탐색적 데이터 분석 프로젝트입니다.
완성된 추천 시스템이 아니라, 상담 현장의 직관을 데이터로 재현·보완할 수 있는지를 검증합니다.

## Key Results

- **거시 구조는 전문가와 일치.** 알고리즘 군집(k=3)은 전문가 합의와 ARI **0.84**로
  일치하며, 학과 쌍 수준에서도 내부 유사도와 전문가 동시분류율이 **r = 0.68**로 정렬됩니다.
- **IDF 가중의 이점은 세분 구조에 한정.** 상위 분할(k=4)에서는 binary가 더 높고
  (**0.59 vs 0.52**), IDF는 의약-보건을 더 일찍 분리하며(k=5 vs binary k=7)
  세분 구조(k=5–6)에서 전문가 합의를 더 잘 추적합니다.
- **불일치를 실무 인사이트로 번역.** 알고리즘-전문가 불일치를 Type 1/Type 2로
  구조화하여 "교차 계열 대안 제시"와 "과목 정합성 경고"라는 상담 활용안으로 연결했습니다.

![알고리즘 군집 vs 전문가 합의 ARI (rater 부트스트랩 95% CI)](reports/final/figures_v2/cardsort_agreement_bars.png)

![IDF 가중 hierarchical 군집 dendrogram (k=4 군집명 표기)](reports/final/figures_v2/idf_dendrogram.png)

## 문제 정의

입시 상담에서 "권장과목 관점에서 어떤 학과들이 비슷한가"는 컨설턴트의 경험적 직관에
의존해 왔습니다. 이 프로젝트는 그 직관을 학과별 권장과목 프로파일이라는 데이터로 재현할
수 있는지, 재현된 구조가 실제 전문가 판단과 어긋나는 지점은 어디인지를 묻습니다.
어긋나는 지점이야말로 데이터가 상담에 보탤 수 있는 정보이기 때문입니다.

분석 대상 25개 학과는 통계적 대표 표본이 아니라 **목적 표본**입니다. 권장과목 다양성,
상담 출현 빈도, 해석 가능성을 기준으로 선정했고, 특히 조선해양공학과·자동차공학과는
울산 주력 산업을 반영한 경계 사례로 포함해 "산업 맥락상 함께 추천되는 학과가
과목 관점에서도 가까운가"를 검증할 수 있게 설계했습니다.
(선정 논리: [docs/department_selection_rationale.md](docs/department_selection_rationale.md))

## 방법

1. 학과 과목 선택 가이드에서 **25×89 권장과목 binary matrix** 구성
2. 공통 과목(예: 확률과 통계, 21/25개 학과)의 변별력 저하를 **IDF 가중**으로 보정
   — `weight = ln(N/df)`, 손으로 정하는 모수 없음. 가중 강도는 α 민감도 스윕으로 검증
3. cosine similarity → **hierarchical clustering**(average linkage, 주 방법) + k-means 비교
4. 전문가 카드소팅 consensus와 **ARI/NMI**로 비교, rater/feature **부트스트랩**으로
   신뢰구간과 군집 안정성 정량화 (seed 고정, 재현 가능)

상세 설계: [docs/project_design.md](docs/project_design.md)

## 검증: 전문가 카드소팅

현직 입시 컨설턴트 14명이 25개 학과 카드를 그룹 수 제한 없이 자유 분류(open card sort)
했고, 응답을 25×25 동시분류 행렬로 집계해 consensus clustering을 도출한 뒤 알고리즘
군집과 비교했습니다. 거시 구조(k=3)는 IDF·binary 모두 ARI 0.84로 전문가와 일치하지만,
상위 k=4는 STEM-보건 대군집이 지배해 양쪽 모두 일치도가 낮고(IDF 0.52, binary 0.59)
IDF가 binary를 이기지 못합니다. IDF의 기여는 의약-보건을 더 일찍 분리하고(k=5 vs k=7)
세분 구조(k=5–6)에서 전문가를 더 잘 추적하는 데 있습니다 — 14명 소패널의 한계를
반영해 일치도의 크기는 부트스트랩 신뢰구간과 함께 방향성 위주로 해석합니다.

## 불일치 분석 → 상담 인사이트

이 프로젝트에서 가장 실무적인 산출물은 알고리즘과 전문가가 **어긋나는 지점**의 구조화입니다.

| 유형 | 패턴 | 상담 활용 |
|---|---|---|
| **Type 1** | 준비과목은 유사한데 전문가는 분리 | 상담이 놓치기 쉬운 **교차 계열 대안** 제시 |
| **Type 2** | 전문가는 함께 추천하는데 권장과목은 분기 | **과목 정합성 경고** — 보강 과목 안내 |

대표 사례: 조선해양공학과↔자동차공학과는 "울산 산업"으로 함께 추천되지만 권장과목은
서로 다른 방향으로 갈립니다(Type 2). 이 경우 자동차를 준비한 학생에게 조선해양을
추천할 때 재료·화학 계열 보강이 필요하다는 신호를 데이터가 먼저 줄 수 있습니다.

## 한계와 확장

- 25개 목적 표본이므로 전체 학과로 일반화할 수 없습니다.
- 수학·과학 공통 과목이 많아 STEM 내부의 세부 전공 차이를 충분히 변별하지 못합니다.
- 전문가 14명 소패널로 일치도 신뢰구간이 넓어, 크기 단정 대신 방향성으로 해석합니다.
- 확장 방향: 입결 데이터를 결합한 Type 2 위험 정량화, 25개 학과를 앵커로 한
  전 학과 준지도 확장. (요약: [reports/final/PORTFOLIO_SUMMARY.md](reports/final/PORTFOLIO_SUMMARY.md))

## 재현 방법

```bash
pip install -r requirements.txt

python src/build_matrix_from_subject_guide.py      # 1) binary matrix 구성
python src/build_idf_weighted_analysis.py          # 2) IDF 군집화 + 민감도 + 안정성
python src/build_card_sorting_analysis.py          # 3) 카드소팅 집계 + ARI + 부트스트랩
python src/build_portfolio_figures_v2.py           # 4) 그림 재생성 (계산 불변)
```

카드소팅 원자료는 응답자 개인정보를 포함하므로 저장소에 없습니다(.gitignore).
분석은 비식별 집계 산출물(`results/tables/keep_for_report/cardsort_*.csv`)로 재현됩니다.

```text
department-course-clustering/
├─ data/           # 원천·가공 데이터 (민감 원자료 제외)
├─ src/            # 재현 가능한 분석 파이프라인
├─ results/        # 그림·표 산출물
├─ reports/final/  # 소논문 v2, 개선 figure, 포트폴리오 문서
└─ docs/           # 설계 노트, 표본 선정 논리, 스키마
```

## 보고서

- 소논문(v2, IEEEtran): [reports/final/권장과목_학과군집화_소논문_v2.tex](reports/final/권장과목_학과군집화_소논문_v2.tex)
- 변경 이력: [reports/final/REVISION_NOTES.md](reports/final/REVISION_NOTES.md)
