# 홍수 확률 예측 모델

## 1. 프로젝트 개요 & 동기

강수, 지형, 하천 관리, 도시화, 기후 변화, 인프라 상태 등 다양한 환경·사회적 요인을 활용한 홍수 발생 확률 예측 프로젝트

단순한 예측값 산출이 아니라, 회귀 모델을 통해 홍수 확률을 예측하고, 추가적으로 위험 탐지 관점의 이진 분류 모델까지 비교한 분석

**핵심 질문**

> 다양한 환경·인프라 지표를 활용해 홍수 발생 확률을 얼마나 정확하게 예측할 수 있는가?
> 홍수 위험 예측에서는 정확도보다 위험 탐지율을 더 중요하게 볼 수 있는가?

## 2. 데이터 출처 & 전처리 과정

### 데이터 구성

* 홍수 예측 학습 데이터
* 주요 변수

  * MonsoonIntensity
  * TopographyDrainage
  * RiverManagement
  * Deforestation
  * Urbanization
  * ClimateChange
  * DamsQuality
  * Siltation
  * DrainageSystems
  * PopulationScore
  * WetlandLoss
  * InadequatePlanning
  * PoliticalFactors 등
* 목표 변수

  * FloodProbability

### 전처리 과정

1. 데이터 구조 및 기초 통계량 확인
2. 변수별 분포 시각화
3. `id` 컬럼 제거
4. 입력 변수 X와 목표 변수 y 분리
5. `PopulationScore + Urbanization` 기반 `Society_Scale` 파생변수 생성
6. 학습 데이터와 검증 데이터 8:2 분리
7. StandardScaler 기반 수치형 변수 스케일링
8. 일부 실험에서 K-Means 기반 군집 변수 추가

## 3. 분석 방법 & 선택 이유

홍수 확률은 연속형 값이므로 회귀 모델을 중심으로 분석 진행
추가적으로 홍수 위험 여부를 판단하기 위해 확률값을 기준으로 이진 분류 실험 수행

| 분석 방법                   | 설명                              |
| ----------------------- | ------------------------------- |
| Linear Regression       | 기본 선형 회귀 모델 기준 성능 확인            |
| Ridge Regression        | 규제를 적용하여 과적합 가능성 완화             |
| Decision Tree Regressor | 비선형 패턴 반영 가능성 확인                |
| LightGBM                | 대용량 데이터 기반 부스팅 모델 성능 비교         |
| XGBoost                 | 비선형 관계와 변수 상호작용 반영 가능성 확인       |
| Voting Ensemble         | Ridge, LightGBM, XGBoost 예측값 결합 |
| RandomForest Classifier | 홍수 위험 여부 이진 분류 실험               |
| K-Means Feature 추가      | 비지도학습 기반 군집 정보의 성능 개선 여부 확인     |

## 4. 핵심 결과

### 1) 선형 모델의 높은 설명력 확인

Linear Regression 모델에서 R² 0.8449 수준의 높은 설명력 확인
Decision Tree Regressor는 R² 0.2648로 낮은 성능 기록

| 모델                      |     R² |
| ----------------------- | -----: |
| Linear Regression       | 0.8449 |
| Decision Tree Regressor | 0.2648 |

홍수 확률 예측에서 복잡한 비선형 모델보다 선형 모델이 더 안정적인 성능을 보이는 구조 확인

### 2) 5-Fold 교차검증 기반 모델 안정성 확인

5-Fold 교차검증 결과 평균 RMSE 0.0201 수준
Fold별 RMSE 차이가 거의 없어 특정 데이터 분할에만 의존하지 않는 안정적인 성능 확인

| 지표   |     결과 |
| ---- | -----: |
| MAE  | 0.0158 |
| MSE  | 0.0004 |
| RMSE | 0.0201 |
| R²   | 0.8449 |

### 3) Ridge Regression 튜닝 결과

GridSearchCV를 통해 Ridge Regression의 최적 alpha 탐색
최적 alpha 값은 1.0, 튜닝 후 RMSE는 0.0201 수준

규제를 적용해도 기존 선형 회귀와 성능 차이가 크지 않아, 데이터 구조상 선형 관계가 강하게 반영된 것으로 해석

### 4) LightGBM, XGBoost, Ensemble 비교

Ridge, LightGBM, XGBoost 및 가중 앙상블 모델 성능 비교 결과, 단일 Ridge 모델의 성능이 가장 우수

| 모델       |             MAE |            RMSE |              R² |
| -------- | --------------: | --------------: | --------------: |
| Ridge    |         0.01579 |         0.02008 |         0.84488 |
| LightGBM |         0.01739 |         0.02157 |         0.82096 |
| XGBoost  |         0.01775 |         0.02190 |         0.81546 |
| Ensemble | 0.01593~0.01630 | 0.02012~0.02042 | 0.83962~0.84432 |

최종 회귀 모델은 가장 낮은 RMSE와 가장 높은 R²를 기록한 Ridge Regression으로 선정

### 5) 홍수 위험 이진 분류 실험

FloodProbability가 0.5를 초과하는 경우를 홍수 위험으로 정의하고 RandomForest 기반 이진 분류 실험 진행
Threshold 0.30 적용 시 Recall 0.9711 기록

| 지표        |     결과 |
| --------- | -----: |
| Accuracy  | 0.6462 |
| Precision | 0.5933 |
| Recall    | 0.9711 |
| F1-score  | 0.7366 |

홍수 위험 예측에서는 위험 사례를 놓치지 않는 것이 중요하므로 Recall 중심 해석 가능성 확인

### 6) K-Means 군집 변수 추가 결과

K-Means를 활용해 군집 변수를 추가했으나, 기존 선형 회귀 모델 대비 성능 개선 효과는 거의 없음

| 구분           |   RMSE |     R² |
| ------------ | -----: | -----: |
| K-Means 적용 전 | 0.0201 | 0.8449 |
| K-Means 적용 후 | 0.0201 | 0.8449 |

비지도학습 기반 파생변수가 항상 예측 성능 개선으로 이어지지는 않는다는 점 확인

## 5. 시각화

* 변수별 분포 히스토그램
* 실제 홍수 확률과 예측 홍수 확률 비교 산점도
* 잔차 분석 그래프
* 오차가 큰 사례 추출
* 모델별 RMSE, R² 성능 비교
* Threshold 조정에 따른 분류 성능 비교

## 6. 한계점 & 향후 개선 방향

### 한계점

* 데이터 내 변수들이 실제 관측값이 아닌 점수화된 지표 중심
* 홍수 확률 예측값의 실제 재난 발생 여부와의 직접 연결 한계
* Ridge 모델 성능이 가장 높아 복잡한 모델의 추가적 장점 제한
* K-Means 군집 변수의 성능 개선 효과 미미
* 이진 분류 기준인 0.5 threshold의 정책적 근거 보완 필요

### 향후 개선 방향

* 실제 강수량, 지형 고도, 하천 수위 등 외부 데이터 결합
* 지역 단위 공간 정보 추가
* 시계열 강수 데이터 기반 예측 모델 확장
* SHAP 등 변수 중요도 해석 기법 적용
* Precision-Recall Curve 기반 위험 판단 임계값 설정
* 홍수 위험 등급화 모델로 확장

## 7. 배운 점

회귀 문제에서는 복잡한 모델이 항상 더 좋은 성능을 내는 것은 아니라는 점을 확인하였다.

홍수 확률 예측에서는 RMSE와 R²를 함께 확인해야 하며, 예측 오차의 크기와 설명력을 동시에 고려할 필요성 학습하였으며,
위험 예측 문제에서는 Accuracy보다 Recall, Precision, F1-score 등 목적에 맞는 지표를 선택해야한다.

비지도학습 기반 군집 feature 추가가 항상 성능 개선으로 이어지지 않으며, 추가 변수의 유효성 검증 과정이 필요하다

## 8. 사용 기술

* Python
* Pandas
* NumPy
* Scikit-learn
* LightGBM
* XGBoost
* Matplotlib
* Jupyter Notebook

## 9. 저장소 구조

```bash
flood-probability-prediction/
├── README.md
├── notebooks/
│   └── flood_probability_prediction.ipynb
├── data/
│   └── train.csv
├── outputs/
│   └── final_predictions.csv
├── requirements.txt
└── .gitignore
```

## 10. 주의사항

학습 및 포트폴리오 목적의 프로젝트

데이터 파일 용량이 큰 경우 원본 데이터 대신 샘플 데이터 또는 데이터 출처만 제공
