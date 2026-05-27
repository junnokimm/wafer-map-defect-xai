# Wafer Map Defect Pattern Classification with Explainable AI

## 0. 문서 목적

이 문서는 `Wafer Map Defect Pattern Classification with Explainable AI` 프로젝트의 전체 방향, 데이터 관리 방식, 모델링 전략, 현재까지의 실험 결과, 앞으로의 작업 계획을 정리한 프로젝트 기준 문서이다.

README는 GitHub 방문자에게 프로젝트를 간단히 소개하는 외부 공개용 문서라면, 이 문서는 앞으로 전처리, 모델링, 성능 분석, XAI 적용, 보고서 작성, 발표 자료 제작 시 참고하기 위한 내부 작업 기준서 역할을 한다.

---

## 1. 프로젝트 한 줄 요약

본 프로젝트는 공개 웨이퍼 맵 데이터셋인 WM-811K / LSWMD를 활용하여 반도체 웨이퍼 결함 패턴을 CNN 기반 모델로 분류하고, Grad-CAM 기반 Explainable AI 기법을 통해 모델이 결함 판단 시 주목한 영역을 시각화하는 것을 목표로 한다.

---

## 2. 프로젝트 주제

### 2.1 최종 주제명

**Wafer Map Defect Pattern Classification with Explainable AI**

### 2.2 한국어 주제명

**첨단 반도체 수율 관리를 위한 웨이퍼 맵 결함 패턴 분류 및 Grad-CAM 기반 해석 가능성 분석**

또는 조금 더 산업적 맥락을 강조하면 다음과 같이 표현할 수 있다.

**HBM 수율 개선을 위한 웨이퍼 맵 결함 패턴 분류 및 해석 가능한 AI 분석**

다만 실제 HBM 제조 데이터가 공개되어 있지 않기 때문에, 보고서나 README에서는 “HBM 데이터로 직접 실험했다”는 표현은 피하고, “HBM과 같은 첨단 반도체 제조 환경에서 중요해지는 수율 관리 문제를 배경으로 한다”는 방식으로 표현한다.

---

## 3. 프로젝트 배경

반도체 제조 공정에서는 하나의 웨이퍼 위에 수많은 die가 형성된다. 제조 및 검사 과정이 끝난 뒤 각 die는 정상 또는 불량으로 판정되며, 이 결과는 웨이퍼 맵 형태로 표현된다.

웨이퍼 맵은 단순히 불량 위치를 보여주는 이미지가 아니다. 불량 die가 웨이퍼의 중앙부에 몰려 있는지, 가장자리를 따라 나타나는지, 특정 국소 영역에 집중되는지, 혹은 긁힘처럼 선형으로 나타나는지에 따라 서로 다른 공정 이상 가능성을 암시할 수 있다.

최근 AI 반도체와 HBM(High Bandwidth Memory)에 대한 수요가 증가하면서, 반도체 제조 공정에서 수율 관리와 결함 분석의 중요성은 더욱 커지고 있다. HBM과 같은 첨단 반도체 제품은 공정 및 패키징 난이도가 높기 때문에, 제조 과정에서 발생하는 결함을 빠르게 탐지하고 원인을 분석하는 기술이 중요하다.

하지만 실제 HBM 제조 데이터는 공개되어 있지 않다. 따라서 본 프로젝트에서는 공개 데이터셋인 WM-811K / LSWMD를 활용하여 웨이퍼 맵 기반 결함 패턴 분류 모델을 구현하고, 클래스 불균형 문제와 모델 해석 가능성을 함께 분석한다.

---

## 4. 문제 정의

본 프로젝트의 핵심 문제는 다음과 같다.

> 웨이퍼 맵 이미지에 나타난 불량 die의 공간적 분포를 기반으로 결함 유형을 자동 분류하고, 모델이 실제로 결함과 관련된 영역을 보고 판단했는지 해석 가능한 방식으로 확인할 수 있는가?

이 문제는 단순한 이미지 분류 문제가 아니라 다음 세 가지 하위 문제를 포함한다.

1. **분류 문제**
   - 웨이퍼 맵 이미지를 입력으로 받아 결함 유형을 예측한다.
   - 결함 유형은 `Center`, `Donut`, `Edge-Loc`, `Edge-Ring`, `Loc`, `Near-full`, `Random`, `Scratch`, `none`으로 구성된다.

2. **불균형 데이터 문제**
   - `none` 클래스가 매우 많고, `Scratch`, `Donut`, `Near-full`과 같은 일부 결함 클래스는 상대적으로 적다.
   - 이로 인해 전체 accuracy는 높게 나와도 소수 클래스 성능은 낮을 수 있다.

3. **모델 해석 가능성 문제**
   - CNN 모델이 높은 정확도를 보이더라도, 실제 결함 영역을 보고 판단했는지는 별도로 확인해야 한다.
   - Grad-CAM을 활용하여 모델이 특정 클래스를 예측할 때 주목한 영역을 시각화한다.

---

## 5. 연구 질문

본 프로젝트의 주요 연구 질문은 다음과 같다.

1. 웨이퍼 맵의 불량 분포 패턴을 기반으로 결함 유형을 자동 분류할 수 있는가?
2. 클래스 불균형이 심한 웨이퍼 맵 데이터에서 단순 accuracy가 아닌 클래스별 성능을 어떻게 평가할 수 있는가?
3. Class weight, downsampling, augmentation 등의 불균형 처리 기법이 소수 결함 클래스의 분류 성능을 개선하는가?
4. 모델은 `Scratch`, `Loc`, `Edge-Loc`처럼 구분이 어려운 결함 클래스를 어떤 방식으로 혼동하는가?
5. Grad-CAM을 활용했을 때 모델이 실제 결함 영역을 근거로 판단하는지 시각적으로 확인할 수 있는가?
6. 웨이퍼 맵 기반 결함 분류와 XAI 분석이 첨단 반도체 수율 관리 관점에서 어떤 의미를 가질 수 있는가?

---

## 6. 사용 데이터셋

### 6.1 데이터셋 이름

**WM-811K Wafer Map Dataset / LSWMD**

본 프로젝트에서는 Kaggle에 공개된 WM-811K Wafer Map Dataset을 사용한다. 실제 작업 파일명은 `LSWMD.pkl`이다.

### 6.2 데이터셋 특징

WM-811K / LSWMD 데이터셋은 반도체 웨이퍼 맵 데이터를 포함하며, 각 웨이퍼 맵은 die 단위의 정상/불량 정보를 2차원 배열 형태로 나타낸다.

일반적으로 웨이퍼 맵의 값은 다음과 같은 의미를 가진다.

| 값 | 의미 |
|---:|---|
| 0 | 웨이퍼 영역 밖 또는 사용하지 않는 영역 |
| 1 | 정상 die |
| 2 | 불량 die |

본 프로젝트에서는 이 2차원 배열을 이미지 형태로 변환하여 CNN 모델의 입력으로 사용한다.

### 6.3 분류 대상 클래스

본 프로젝트에서 사용하는 결함 클래스는 총 9개이다.

| 클래스 | 설명 |
|---|---|
| `none` | 특별한 결함 패턴이 없는 정상 또는 무결함 웨이퍼 |
| `Center` | 웨이퍼 중앙부에 결함이 집중된 패턴 |
| `Donut` | 도넛 형태로 결함이 분포하는 패턴 |
| `Edge-Loc` | 웨이퍼 가장자리 특정 위치에 결함이 집중된 패턴 |
| `Edge-Ring` | 웨이퍼 가장자리를 따라 링 형태로 결함이 나타나는 패턴 |
| `Loc` | 특정 국소 영역에 결함이 집중된 패턴 |
| `Random` | 결함이 무작위로 분포하는 패턴 |
| `Scratch` | 긁힘처럼 선형으로 결함이 나타나는 패턴 |
| `Near-full` | 웨이퍼 대부분이 불량에 가까운 패턴 |

### 6.4 데이터 관리 방식

데이터 파일은 용량이 크고 라이선스 및 재배포 이슈가 있을 수 있으므로 GitHub 저장소에는 포함하지 않는다.

데이터는 Google Drive에 저장하고, Colab에서 해당 경로를 마운트하여 사용한다.

예상 Google Drive 구조는 다음과 같다.

```text
MyDrive/
└── wafer-map-defect-xai/
    ├── data/
    │   ├── raw/
    │   │   └── LSWMD.pkl
    │   └── processed/
    │       ├── X_train.npy
    │       ├── X_val.npy
    │       ├── X_test.npy
    │       ├── y_train.npy
    │       ├── y_val.npy
    │       └── y_test.npy
    └── outputs/
        ├── figures/
        ├── models/
        └── reports/
```

---

## 7. 프로젝트 폴더 구조

GitHub 저장소의 기본 구조는 다음과 같이 관리한다.

```text
wafer-map-defect-xai/
├── README.md
├── docs/
│   └── project_overview.md
├── notebooks/
│   └── 01_data_eda_preprocessing.ipynb
├── data/
│   ├── raw/
│   │   └── .gitkeep
│   └── processed/
│       └── .gitkeep
├── outputs/
│   ├── figures/
│   ├── models/
│   └── reports/
│       └── cnn_baseline_result.md
├── src/
│   ├── data/
│   ├── models/
│   ├── visualization/
│   └── xai/
└── requirements.txt
```

### 7.1 GitHub에 올리는 파일

GitHub에 올릴 수 있는 파일은 다음과 같다.

- README.md
- docs/project_overview.md
- notebooks
- src 코드
- requirements.txt
- 실험 결과 요약 markdown
- confusion matrix 등 주요 결과 이미지
- 작은 크기의 샘플 시각화 이미지

### 7.2 GitHub에 올리지 않는 파일

다음 파일은 GitHub에 직접 올리지 않는다.

- `LSWMD.pkl`
- `X_train.npy`, `X_val.npy`, `X_test.npy`
- `y_train.npy`, `y_val.npy`, `y_test.npy`
- `.h5`, `.keras` 등 큰 모델 파일
- 용량이 큰 중간 산출물

이 파일들은 Google Drive에서 관리하고, README나 문서에는 데이터 위치와 재현 방법만 적는다.

---

## 8. 현재까지 진행 상황

### 8.1 데이터 로드 및 경로 문제 해결

초기 작업에서는 Google Drive에 저장된 `LSWMD.pkl` 파일을 Colab에서 불러오는 과정에서 경로 문제가 있었다. 특히 `RAW_PATH exists: False`가 출력되며 파일이 존재하지 않는 것처럼 보이는 상황이 발생했다.

이후 Google Drive 마운트 경로와 실제 폴더 구조를 확인하면서 데이터 경로를 다시 점검했다.

Colab 기준 기본 경로는 다음과 같이 사용한다.

```python
BASE_DIR = "/content/drive/MyDrive/wafer-map-defect-xai"
RAW_PATH = f"{BASE_DIR}/data/raw/LSWMD.pkl"
PROCESSED_DIR = f"{BASE_DIR}/data/processed"
```

### 8.2 전처리 데이터 재생성

기존에는 `X_train.npy` 파일이 누락되어 CNN 학습을 바로 진행할 수 없는 문제가 있었다.

이를 해결하기 위해 전체 라벨 데이터를 기준으로 train / validation / test 데이터를 다시 생성하였다.

최종 입력 이미지는 모두 `64x64` 크기로 통일했으며, 모델 입력 형태는 다음과 같다.

```text
Input shape: (64, 64, 1)
```

### 8.3 최종 데이터 분할

현재 CNN baseline 실험에서 사용한 데이터 분할은 다음과 같다.

```text
Train: 121,065
Validation: 25,942
Test: 25,943
```

### 8.4 클래스 구성

분류 대상 클래스는 총 9개이다.

```text
Center
Donut
Edge-Loc
Edge-Ring
Loc
Near-full
Random
Scratch
none
```

데이터셋은 `none` 클래스가 매우 많은 불균형 구조를 가진다. 따라서 1차 CNN baseline 실험에서는 `class_weight`를 적용하였다.

---

## 9. CNN Baseline 실험

### 9.1 실험 목적

CNN baseline 실험의 목적은 다음과 같다.

1. 웨이퍼 맵 결함 패턴 분류가 CNN으로 가능한지 확인한다.
2. 향후 개선 모델과 비교하기 위한 기준 성능을 확보한다.
3. 클래스 불균형 상황에서 accuracy만으로 성능을 판단하기 어렵다는 점을 확인한다.
4. 클래스별 precision, recall, f1-score를 통해 어떤 결함 유형이 특히 어려운지 분석한다.

### 9.2 모델 구조

현재 baseline 모델은 기본 CNN 구조를 사용한다.

```text
Input
Conv2D + BatchNormalization + MaxPooling
Conv2D + BatchNormalization + MaxPooling
Conv2D + BatchNormalization + MaxPooling
Conv2D + BatchNormalization + MaxPooling
Flatten
Dense
Dropout
Dense Softmax
```

### 9.3 학습 설정

학습 설정은 다음과 같다.

```text
Optimizer: Adam
Loss: Sparse Categorical Crossentropy
Batch size: 128
Class weight: Applied
Early stopping: Applied
Model checkpoint: Applied
```

가장 좋은 검증 성능은 epoch 17에서 나타났고, 이후 early stopping에 의해 best weight가 복원되었다.

```text
Best Validation Accuracy: 0.9313
Best Validation Loss: 0.2080
```

### 9.4 테스트 결과

최종 test set 평가 결과는 다음과 같다.

```text
Test Loss: 0.2087
Test Accuracy: 0.9340
```

전체 정확도는 약 93.4%로 높게 나타났다.

---

## 10. 클래스별 성능 분석

### 10.1 Classification Report

CNN baseline 모델의 클래스별 성능은 다음과 같다.

```text
              precision    recall  f1-score   support

Center          0.83      0.95      0.88       644
Donut           0.77      0.89      0.83        83
Edge-Loc        0.59      0.78      0.67       779
Edge-Ring       0.97      0.96      0.97      1452
Loc             0.61      0.67      0.64       539
Near-full       0.91      0.91      0.91        22
Random          0.68      0.92      0.78       130
Scratch         0.14      0.57      0.22       179
none            0.99      0.95      0.97      22115

accuracy                            0.93      25943
macro avg       0.72      0.84      0.76      25943
weighted avg    0.96      0.93      0.94      25943
```

### 10.2 주요 해석

전체 accuracy는 93.4%로 높게 나타났지만, 데이터셋의 클래스 불균형을 고려하면 accuracy만으로 모델 성능을 판단하기는 어렵다.

특히 `none` 클래스의 support가 22,115개로 매우 많기 때문에, `none`을 잘 맞추는 것만으로도 전체 accuracy와 weighted average가 높게 나타날 수 있다.

따라서 본 프로젝트에서는 다음 지표를 함께 확인해야 한다.

- Accuracy
- Macro F1-score
- Weighted F1-score
- 클래스별 Precision
- 클래스별 Recall
- 클래스별 F1-score
- Confusion Matrix

### 10.3 성능이 좋은 클래스

현재 모델은 다음 클래스에서 좋은 성능을 보였다.

```text
none f1-score: 0.97
Edge-Ring f1-score: 0.97
Center f1-score: 0.88
```

이 클래스들은 패턴이 비교적 명확하거나 데이터 수가 충분하여 CNN이 공간적 특징을 잘 학습한 것으로 볼 수 있다.

### 10.4 성능이 낮은 클래스

반면 다음 클래스에서는 상대적으로 낮은 성능을 보였다.

```text
Scratch f1-score: 0.22
Loc f1-score: 0.64
Edge-Loc f1-score: 0.67
```

특히 `Scratch` 클래스는 precision이 0.14로 매우 낮았다. 이는 모델이 실제 Scratch가 아닌 샘플까지 Scratch로 많이 예측했다는 의미이다.

즉, 현재 모델은 실제 Scratch 샘플 중 일부를 탐지하는 능력은 있지만, Scratch가 아닌 샘플도 Scratch로 과하게 예측하는 문제가 있다.

---

## 11. 현재 모델의 한계

### 11.1 Accuracy 중심 해석의 한계

전체 accuracy가 93.4%라는 점만 보면 모델 성능이 매우 좋아 보일 수 있다. 그러나 `none` 클래스가 압도적으로 많기 때문에 accuracy는 다수 클래스 성능에 크게 영향을 받는다.

따라서 이 프로젝트에서는 accuracy보다 다음 지표를 더 중요하게 해석해야 한다.

- Macro F1-score
- 소수 클래스 F1-score
- 소수 클래스 recall
- 소수 클래스 precision
- 클래스별 confusion matrix

### 11.2 Scratch 클래스 precision 문제

`Scratch` 클래스는 recall이 0.57이지만 precision은 0.14이다.

이는 다음과 같이 해석할 수 있다.

```text
실제 Scratch 중 일부는 맞추고 있지만,
모델이 Scratch라고 예측한 샘플 중 상당수는 실제 Scratch가 아니다.
```

즉, Scratch 클래스에서 false positive가 많이 발생하고 있다.

### 11.3 Loc / Edge-Loc 혼동 가능성

`Loc`와 `Edge-Loc`는 모두 특정 위치에 결함이 집중되는 패턴이다. 차이는 결함이 웨이퍼 내부의 국소 영역에 있는지, 가장자리 근처에 있는지에 있다.

따라서 웨이퍼 가장자리와 내부의 경계가 애매하거나 결함 위치가 복합적인 샘플에서는 모델이 두 클래스를 혼동할 가능성이 있다.

### 11.4 Class weight 적용의 부작용 가능성

현재 baseline 모델은 class weight를 적용하였다. class weight는 소수 클래스의 학습 비중을 높여 recall 개선에는 도움이 될 수 있지만, 특정 소수 클래스를 과하게 예측하여 precision을 낮출 수도 있다.

Scratch precision이 매우 낮게 나타난 것은 class weight 적용에 따른 부작용일 가능성도 있으므로, 이후에는 class weight를 적용하지 않은 모델과 비교해야 한다.

---

## 12. 바로 다음 작업

현재 단계에서 가장 먼저 해야 할 작업은 새로운 모델을 만드는 것이 아니라, baseline 모델의 오분류 패턴을 더 자세히 분석하는 것이다.

### 12.1 Normalized Confusion Matrix 생성

기존 confusion matrix는 개수 기반이므로 `none` 클래스의 데이터 수가 너무 커서 다른 클래스의 오분류가 잘 보이지 않을 수 있다.

따라서 각 true class 기준으로 정규화된 confusion matrix를 추가로 생성한다.

목적은 다음과 같다.

- 실제 Scratch 중 몇 %가 Scratch로 분류되었는지 확인
- 실제 Scratch가 어떤 클래스로 오분류되는지 확인
- Loc와 Edge-Loc가 서로 얼마나 혼동되는지 확인
- none이 Scratch로 얼마나 잘못 예측되는지 확인

### 12.2 오분류 Top 20 표 생성

전체 confusion matrix에서 true label과 predicted label이 다른 경우만 추출하여, 가장 많이 발생한 오분류 조합을 확인한다.

예상 분석 질문은 다음과 같다.

- 어떤 클래스가 가장 자주 다른 클래스로 잘못 예측되는가?
- 어떤 실제 클래스가 Scratch로 많이 잘못 예측되는가?
- Loc와 Edge-Loc는 서로 혼동되는가?
- none이 특정 결함 클래스로 많이 잘못 예측되는가?

### 12.3 오분류 이미지 직접 확인

특히 다음 클래스의 오분류 샘플을 직접 시각화한다.

```text
Scratch
Loc
Edge-Loc
```

확인할 질문은 다음과 같다.

- Scratch가 너무 얇거나 부분적으로 나타나서 none으로 예측되는가?
- Scratch가 아닌 샘플이 왜 Scratch로 예측되는가?
- Loc와 Edge-Loc의 결함 위치가 시각적으로 애매한가?
- Edge-Ring과 Edge-Loc가 일부 샘플에서 비슷하게 보이는가?

### 12.4 결과 markdown 업데이트

`outputs/reports/cnn_baseline_result.md`에 다음 섹션을 추가한다.

```md
## 8. 오분류 분석

Confusion Matrix를 확인한 결과, 모델은 Scratch, Loc, Edge-Loc 클래스에서 상대적으로 많은 오분류를 보였다.

Scratch 클래스는 recall은 0.57로 일부 실제 Scratch 샘플을 탐지했지만, precision이 0.14로 매우 낮았다. 이는 모델이 Scratch가 아닌 샘플을 Scratch로 과하게 예측하고 있음을 의미한다.

Loc와 Edge-Loc 클래스는 결함의 위치와 형태가 부분적으로 유사하여 서로 혼동될 가능성이 있다. 따라서 이후 실험에서는 단순 accuracy보다 클래스별 precision, recall, f1-score와 오분류 패턴을 함께 분석해야 한다.
```

---

## 13. 이후 개선 실험 계획

### 13.1 실험 A: Class weight 비교

현재 모델은 class weight를 적용한 상태이다. 다음 실험에서는 class weight를 적용하지 않은 CNN과 비교한다.

비교 대상은 다음과 같다.

```text
CNN baseline without class_weight
CNN baseline with class_weight
```

비교 기준은 다음과 같다.

- Test accuracy
- Macro F1-score
- Weighted F1-score
- Scratch precision / recall / f1-score
- Loc f1-score
- Edge-Loc f1-score

목표는 class weight가 소수 클래스 성능을 실제로 개선했는지, 혹은 precision 저하를 유발했는지 확인하는 것이다.

### 13.2 실험 B: none downsampling

현재 데이터셋은 `none` 클래스가 압도적으로 많다. 따라서 `none` 클래스를 일부만 샘플링하여 결함 클래스와의 비율을 완화하는 실험을 진행할 수 있다.

예상 방식은 다음과 같다.

```text
defect class: 가능한 많이 유지
none class: 일정 비율로 downsampling
```

이 실험의 목적은 모델이 `none` 클래스에 지나치게 지배되지 않도록 하고, 결함 클래스의 분류 성능을 개선할 수 있는지 확인하는 것이다.

### 13.3 실험 C: Data augmentation

소수 클래스에 대해 데이터 증강을 적용할 수 있다.

후보 증강 방식은 다음과 같다.

- Rotation
- Horizontal flip
- Vertical flip
- Small shift

다만 웨이퍼 맵은 결함의 위치와 형태가 중요한 데이터이므로, 모든 증강이 항상 적절한 것은 아니다.

특히 `Edge-Loc`, `Edge-Ring`처럼 위치 정보가 중요한 클래스는 과도한 이동이나 확대/축소가 클래스 의미를 흐릴 수 있으므로 조심해야 한다.

### 13.4 실험 D: Focal Loss

Focal Loss는 쉬운 샘플보다 어려운 샘플에 더 집중하도록 만든 손실 함수이다. 불균형 분류 문제에서 사용할 수 있는 개선 방법이다.

다만 구현과 튜닝이 추가로 필요하므로, class weight 비교와 downsampling 실험 이후에 적용하는 것이 좋다.

### 13.5 실험 E: Grad-CAM 기반 XAI 분석

모델 개선 실험 이후에는 Grad-CAM을 적용한다.

Grad-CAM의 목적은 다음과 같다.

- 모델이 `Edge-Ring`을 예측할 때 실제로 웨이퍼 가장자리를 보는지 확인
- 모델이 `Center`를 예측할 때 실제로 중앙 결함 영역을 보는지 확인
- 모델이 `Scratch`를 예측할 때 선형 결함 영역을 보는지 확인
- 오분류 샘플에서 모델이 엉뚱한 영역을 보고 판단했는지 확인

---

## 14. Grad-CAM 분석 계획

### 14.1 Grad-CAM을 적용할 샘플

Grad-CAM은 전체 데이터에 무작정 적용하기보다 대표 샘플을 선별해서 적용한다.

추천 샘플은 다음과 같다.

1. 모델이 맞춘 샘플
   - `Center`
   - `Edge-Ring`
   - `none`
   - `Scratch`

2. 모델이 틀린 샘플
   - True Scratch / Pred none
   - True none / Pred Scratch
   - True Loc / Pred Edge-Loc
   - True Edge-Loc / Pred Loc

3. 신뢰도가 높은 샘플
   - softmax confidence가 높은 correct prediction

4. 신뢰도가 낮거나 애매한 샘플
   - softmax confidence가 낮은 prediction
   - top-1과 top-2 확률 차이가 작은 prediction

### 14.2 Grad-CAM 해석 질문

Grad-CAM 결과를 볼 때는 다음 질문을 기준으로 해석한다.

- 모델이 실제 불량 die가 분포한 영역을 보고 있는가?
- 모델이 웨이퍼의 배경이나 외곽 형태만 보고 판단하는가?
- `Edge-Ring` 예측에서 가장자리 영역이 강조되는가?
- `Center` 예측에서 중앙부가 강조되는가?
- `Scratch` 예측에서 선형 결함 영역이 강조되는가?
- 오분류 샘플에서는 결함과 무관한 영역에 주목했는가?

### 14.3 Grad-CAM 결과 문서화 방식

Grad-CAM 결과는 다음 형식으로 정리한다.

| Case | True Label | Predicted Label | Confidence | Grad-CAM 해석 |
|---|---|---|---:|---|
| Correct 1 | Edge-Ring | Edge-Ring | 0.98 | 가장자리 링 영역을 중심으로 활성화됨 |
| Correct 2 | Center | Center | 0.95 | 중앙 결함 영역에 주목함 |
| Error 1 | Scratch | none | 0.72 | Scratch 선형 영역이 약하게 반영됨 |
| Error 2 | Loc | Edge-Loc | 0.81 | 결함 위치가 가장자리 근처로 해석됨 |

---

## 15. pseudo-yield 분석 계획

### 15.1 pseudo-yield 정의

실제 HBM 수율 데이터는 공개되어 있지 않기 때문에, 본 프로젝트에서는 웨이퍼 맵 내 정상 die 비율을 기반으로 pseudo-yield를 정의할 수 있다.

```text
pseudo-yield = 정상 die 수 / 전체 유효 die 수
```

웨이퍼 맵 값 기준으로 계산하면 다음과 같다.

```text
정상 die 수 = 값이 1인 die 개수
불량 die 수 = 값이 2인 die 개수
전체 유효 die 수 = 값이 1 또는 2인 die 개수
pseudo-yield = 정상 die 수 / (정상 die 수 + 불량 die 수)
```

### 15.2 pseudo-yield 분석 목적

pseudo-yield 분석은 실제 기업 수율을 대체하는 것이 아니라, 웨이퍼 맵 데이터 안에서 결함 비율과 결함 유형 간의 관계를 탐색하기 위한 분석용 지표이다.

분석 질문은 다음과 같다.

- 결함 클래스별 pseudo-yield 분포는 어떻게 다른가?
- `Near-full` 클래스는 pseudo-yield가 낮게 나타나는가?
- `none` 클래스는 pseudo-yield가 높게 나타나는가?
- `Scratch`, `Loc`, `Edge-Loc` 등의 결함 유형은 pseudo-yield와 어떤 관계를 보이는가?

### 15.3 주의할 점

pseudo-yield는 실제 제조사의 수율 데이터가 아니므로 보고서에서는 반드시 다음과 같이 명시한다.

```text
본 프로젝트의 pseudo-yield는 실제 HBM 제조 수율이 아니라, 공개 웨이퍼 맵 데이터에서 정상 die와 불량 die의 비율을 기반으로 정의한 분석용 대체 지표이다.
```

---

## 16. 최종 산출물

프로젝트 최종 산출물은 다음과 같다.

### 16.1 필수 산출물

1. 클래스별 데이터 분포 그래프
2. 결함 유형별 웨이퍼 맵 샘플 이미지
3. CNN baseline 성능 결과
4. Classification report
5. Confusion matrix
6. Normalized confusion matrix
7. 오분류 Top 20 분석표
8. 오분류 샘플 이미지 시각화
9. 개선 실험 결과 비교표
10. Grad-CAM 시각화 결과
11. 프로젝트 README
12. 최종 보고서 또는 발표 자료

### 16.2 선택 산출물

1. pseudo-yield 분석 그래프
2. EfficientNet 또는 ResNet 비교 실험
3. Focal Loss 실험
4. SHAP, Integrated Gradients 등 다른 XAI 기법 비교
5. 웹 데모 또는 Streamlit 시각화 페이지

---

## 17. 보고서 목차 초안

최종 보고서는 다음 구조로 작성할 수 있다.

```text
1. 서론
   1.1 연구 배경
   1.2 문제 정의
   1.3 연구 목적

2. 데이터셋
   2.1 WM-811K / LSWMD 데이터셋
   2.2 웨이퍼 맵 구조
   2.3 결함 클래스
   2.4 클래스 불균형 문제

3. 방법론
   3.1 데이터 전처리
   3.2 CNN baseline 모델
   3.3 클래스 불균형 처리
   3.4 성능 평가 지표
   3.5 Grad-CAM 기반 XAI 분석

4. 실험 결과
   4.1 데이터 분포
   4.2 CNN baseline 결과
   4.3 클래스별 성능 분석
   4.4 Confusion matrix 분석
   4.5 개선 실험 비교
   4.6 Grad-CAM 결과

5. 논의
   5.1 Accuracy 중심 평가의 한계
   5.2 소수 결함 클래스 분류의 어려움
   5.3 Grad-CAM 해석 결과의 의미
   5.4 첨단 반도체 수율 관리 관점에서의 활용 가능성
   5.5 실제 제조 데이터 부재의 한계

6. 결론
   6.1 연구 요약
   6.2 주요 기여
   6.3 향후 연구 방향
```

---

## 18. 핵심 개념 정리

### 18.1 Wafer

반도체 칩을 만들기 위한 원판이다. 하나의 웨이퍼 위에 수많은 개별 칩 단위인 die가 형성된다.

### 18.2 Die

웨이퍼 위에 만들어진 개별 칩 단위이다. 검사 후 정상 die와 불량 die로 나뉜다.

### 18.3 Wafer Map

웨이퍼 위 die들의 정상/불량 상태를 위치별로 나타낸 지도이다. 본 프로젝트의 핵심 입력 데이터이다.

### 18.4 Yield

전체 die 중 정상 die의 비율이다. 일반적으로 수율이 높을수록 제조 효율이 좋다고 볼 수 있다.

### 18.5 Defect Pattern

불량 die가 어떤 형태로 분포하는지 나타내는 패턴이다. Center, Edge-Ring, Scratch, Random 등이 대표적이다.

### 18.6 CNN

Convolutional Neural Network의 약자이다. 이미지의 공간적 패턴을 학습하는 데 강점이 있는 딥러닝 모델이다.

### 18.7 Class Imbalance

클래스별 데이터 수가 크게 차이 나는 문제이다. 본 프로젝트에서는 `none` 클래스가 매우 많고 일부 결함 클래스가 적기 때문에 중요한 문제이다.

### 18.8 Macro F1-score

각 클래스의 F1-score를 동일한 비중으로 평균낸 지표이다. 클래스 불균형이 심한 데이터에서 모델이 소수 클래스도 잘 분류하는지 확인하는 데 유용하다.

### 18.9 Confusion Matrix

모델이 어떤 클래스를 어떤 클래스로 예측했는지 보여주는 표이다. 오분류 패턴을 분석할 때 중요하다.

### 18.10 Grad-CAM

Gradient-weighted Class Activation Mapping의 약자이다. CNN 모델이 특정 클래스를 예측할 때 이미지의 어느 영역을 중요하게 보았는지 heatmap으로 시각화하는 XAI 기법이다.

---

## 19. 프로젝트 차별화 포인트

본 프로젝트의 차별화 포인트는 다음과 같다.

### 19.1 첨단 반도체 수율 관리 문제와 연결

단순한 이미지 분류 과제가 아니라, HBM과 같은 첨단 반도체 제조 환경에서 중요해지는 수율 관리와 결함 분석 문제를 배경으로 한다.

단, 실제 HBM 데이터가 아니라 공개 웨이퍼 맵 데이터 기반의 가능성 탐색이라는 점을 명확히 밝힌다.

### 19.2 클래스 불균형 분석 중심

단순히 accuracy가 높은 모델을 만드는 것에 그치지 않고, 소수 결함 클래스의 precision, recall, f1-score를 함께 분석한다.

특히 `Scratch`, `Loc`, `Edge-Loc`처럼 성능이 낮은 클래스의 오분류 패턴을 중심으로 모델의 한계를 분석한다.

### 19.3 Grad-CAM 기반 해석 가능성

모델이 결함을 맞혔는지뿐만 아니라, 실제 결함 영역을 보고 판단했는지를 Grad-CAM으로 확인한다.

이는 제조 현장에서 AI 모델을 신뢰하고 활용하기 위해 중요한 요소이다.

### 19.4 pseudo-yield 분석 가능성

실제 제조 수율 데이터가 없다는 한계를 보완하기 위해, 웨이퍼 맵 내 정상 die 비율을 기반으로 pseudo-yield를 정의할 수 있다.

이를 통해 결함 유형과 수율 저하 가능성 간의 관계를 탐색할 수 있다.

---

## 20. Git 작업 기준

### 20.1 결과 요약 markdown 저장 위치

CNN baseline 결과 요약은 다음 위치에 저장한다.

```text
outputs/reports/cnn_baseline_result.md
```

프로젝트 전체 설명 문서는 다음 위치에 저장한다.

```text
docs/project_overview.md
```

### 20.2 Git commit 메시지 예시

전처리 데이터 생성 및 CNN baseline 학습 코드를 추가한 경우:

```bash
git add .
git commit -m "전처리 데이터 재생성 및 CNN baseline 학습 코드 추가"
```

프로젝트 설명 문서를 추가한 경우:

```bash
git add docs/project_overview.md
git commit -m "프로젝트 상세 설명 문서 추가"
```

CNN baseline 결과 보고서를 추가한 경우:

```bash
git add outputs/reports/cnn_baseline_result.md
git commit -m "CNN baseline 결과 요약 문서 추가"
```

### 20.3 데이터 파일 관리 주의

대용량 데이터와 모델 파일은 GitHub에 직접 올리지 않는다. 필요한 경우 Google Drive 링크를 README에 추가하되, 공개 범위는 신중하게 설정한다.

권장 방식은 다음과 같다.

- 공개 가능한 데이터 출처는 Kaggle 링크로 안내
- 직접 생성한 대용량 전처리 파일은 Google Drive에 저장
- Google Drive 링크는 필요한 경우에만 공유
- 링크 공유 시 개인정보나 비공개 데이터가 포함되지 않았는지 확인

---

## 21. 앞으로의 작업 체크리스트

### 21.1 Baseline 분석 마무리

- [ ] Normalized confusion matrix 생성
- [ ] 오분류 Top 20 표 생성
- [ ] Scratch 오분류 샘플 시각화
- [ ] Loc 오분류 샘플 시각화
- [ ] Edge-Loc 오분류 샘플 시각화
- [ ] `cnn_baseline_result.md`에 오분류 분석 섹션 추가

### 21.2 개선 실험

- [ ] class weight 미적용 CNN 학습
- [ ] class weight 적용 / 미적용 결과 비교
- [ ] none downsampling 실험
- [ ] augmentation 실험
- [ ] macro F1-score 중심 비교표 작성

### 21.3 XAI 분석

- [ ] Grad-CAM 코드 구현
- [ ] correct prediction 샘플 Grad-CAM 생성
- [ ] misclassification 샘플 Grad-CAM 생성
- [ ] 클래스별 Grad-CAM 해석 정리
- [ ] XAI 결과 이미지 저장

### 21.4 최종 정리

- [ ] README 업데이트
- [ ] 프로젝트 상세 문서 업데이트
- [ ] 결과 보고서 작성
- [ ] 발표 자료 구성
- [ ] GitHub 정리
- [ ] Velog 또는 포트폴리오용 글 작성

---

## 22. 현재 기준 다음 작업 우선순위

현재 상황에서 가장 적절한 다음 작업 순서는 다음과 같다.

```text
1. docs/project_overview.md 추가
2. cnn_baseline_result.md 정리 및 GitHub 업로드
3. Normalized Confusion Matrix 추가
4. 오분류 Top 20 표 생성
5. Scratch / Loc / Edge-Loc 오분류 이미지 확인
6. class_weight 비교 실험
7. none downsampling 또는 augmentation 실험
8. Grad-CAM 적용
```

현재 CNN baseline은 이미 의미 있는 1차 결과를 만들었다. 따라서 바로 더 복잡한 모델을 만드는 것보다, 먼저 오분류 분석을 통해 모델의 한계를 명확히 정리하는 것이 좋다.

---

## 23. 프로젝트 설명용 최종 문장

README, 보고서, 발표 자료에서 사용할 수 있는 최종 설명 문장은 다음과 같다.

> 본 프로젝트는 AI 반도체 및 HBM 수요 증가로 중요성이 커지고 있는 반도체 수율 관리 문제에 주목한다. 실제 HBM 공정 데이터는 공개되어 있지 않기 때문에, 공개 웨이퍼 맵 데이터셋인 WM-811K / LSWMD를 활용하여 웨이퍼 결함 패턴을 자동 분류하는 CNN 기반 모델을 개발한다. 특히 클래스 불균형 문제를 고려하여 결함 유형별 precision, recall, f1-score를 분석하고, Grad-CAM을 통해 모델이 결함 판단 시 주목한 영역을 시각화한다. 이를 통해 웨이퍼 맵 기반 결함 분석이 첨단 반도체 제조 공정의 수율 저하 원인 파악과 품질 관리에 활용될 수 있는 가능성을 검토한다.

---

## 24. 주의해서 표현해야 할 점

### 24.1 HBM 관련 표현

잘못된 표현:

```text
본 프로젝트는 실제 HBM 제조 데이터를 사용한다.
```

올바른 표현:

```text
본 프로젝트는 실제 HBM 제조 데이터가 공개되어 있지 않다는 한계를 고려하여, 공개 웨이퍼 맵 데이터셋을 활용해 첨단 반도체 수율 관리에 적용 가능한 결함 분석 방법을 탐색한다.
```

### 24.2 수율 개선 관련 표현

잘못된 표현:

```text
본 모델은 HBM 수율을 개선한다.
```

올바른 표현:

```text
본 모델은 웨이퍼 맵 결함 패턴을 자동 분류하고 모델 판단 근거를 시각화함으로써, 향후 수율 저하 원인 분석과 품질 관리에 활용될 수 있는 가능성을 탐색한다.
```

### 24.3 Accuracy 관련 표현

잘못된 표현:

```text
정확도가 93.4%이므로 모델 성능이 매우 우수하다.
```

올바른 표현:

```text
전체 accuracy는 93.4%로 높게 나타났지만, 클래스 불균형이 심한 데이터셋 특성상 macro F1-score와 클래스별 성능을 함께 고려해야 한다.
```

---

## 25. 문서 업데이트 규칙

이 문서는 프로젝트가 진행될 때마다 계속 업데이트한다.

업데이트가 필요한 경우는 다음과 같다.

- 새로운 전처리 방식이 추가된 경우
- 데이터 분할 방식이 바뀐 경우
- 새로운 모델 실험이 추가된 경우
- 성능 수치가 바뀐 경우
- Grad-CAM 결과가 추가된 경우
- README 구조가 바뀐 경우
- 보고서 방향이 바뀐 경우

업데이트 시에는 변경 내용을 Git commit으로 남긴다.

```bash
git add docs/project_overview.md
git commit -m "프로젝트 상세 문서 업데이트"
```
