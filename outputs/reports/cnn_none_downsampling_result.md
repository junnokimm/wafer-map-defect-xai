# CNN Baseline 개선 실험: None Downsampling 결과 정리

## 1. 문서 목적

이 문서는 `Wafer Map Defect Pattern Classification with Explainable AI` 프로젝트에서 수행한 CNN baseline 개선 실험 중 **none downsampling 실험**의 목적, 방법, 결과, 해석을 정리한 결과 문서이다.

이전 실험에서는 클래스 불균형 문제를 완화하기 위해 `class_weight` 적용 여부를 비교하였다. 그 결과 `class_weight` 적용 모델은 소수 클래스 예측이 불안정했고, `class_weight` 미적용 모델은 전체 accuracy는 높았지만 Scratch와 같은 소수 결함 클래스를 충분히 탐지하지 못하는 한계가 있었다.

이에 따라 본 실험에서는 `none` 클래스의 학습 데이터 수를 줄이는 none downsampling 방식을 적용하여, 전체 성능을 유지하면서 소수 결함 클래스의 분류 성능을 개선할 수 있는지 확인하였다.

---

## 2. 실험 배경

WM-811K / LSWMD 데이터셋은 `none` 클래스의 비중이 매우 큰 불균형 데이터셋이다. 이로 인해 모델은 전체 accuracy를 높이기 위해 다수 클래스인 `none`을 중심으로 학습할 가능성이 있다.

이전 실험에서 확인한 문제는 다음과 같다.

- `class_weight` 적용 모델은 소수 클래스에 더 큰 가중치를 부여했지만, 성능이 불안정하게 나타났다.
- `class_weight` 미적용 모델은 전체 accuracy는 높았으나, Scratch 클래스의 recall이 매우 낮게 나타났다.
- 즉, 모델이 실제 Scratch 샘플을 충분히 탐지하지 못하고 `none` 등 다른 클래스로 오분류하는 문제가 있었다.

따라서 이번 실험에서는 학습 데이터에서 `none` 클래스의 수를 줄이고, 결함 클래스는 유지하는 방식으로 데이터 불균형을 완화하였다.

---

## 3. 실험 설정

### 3.1 실험 이름

```text
cnn_none_downsampling
```

### 3.2 기본 모델 구조

기존 CNN baseline과 동일한 모델 구조를 사용하였다.

```text
Input: 64 x 64 x 1
Conv2D + BatchNormalization + MaxPooling
Conv2D + BatchNormalization + MaxPooling
Conv2D + BatchNormalization + MaxPooling
Conv2D + BatchNormalization + MaxPooling
Flatten
Dense
Dropout
Dense Softmax
```

모델 구조는 기존 baseline과 동일하게 유지하고, 학습 데이터 구성만 변경하였다.

### 3.3 Downsampling 방식

Downsampling은 **train set에만 적용**하였다.

```text
X_train, y_train: none downsampling 적용
X_val, y_val: 원본 유지
X_test, y_test: 원본 유지
```

Validation set과 test set은 기존 실험과 동일하게 유지하여, 모델 간 성능 비교가 공정하게 이루어지도록 하였다.

이번 실험에서는 `none` 클래스의 학습 데이터 수를 전체 defect 클래스 수의 2배 수준으로 제한하였다.

```text
none 클래스 수 = defect 클래스 전체 수 x 2
```

### 3.4 Class weight 사용 여부

이번 실험에서는 `class_weight`를 사용하지 않았다.

```text
class_weight: Not applied
```

이번 실험의 목적은 class weight가 아니라 **학습 데이터 비율 조정의 효과**를 확인하는 것이다.

---

## 4. 실험 결과 요약

| 실험 | Accuracy | Macro F1 | Weighted F1 | Scratch Precision | Scratch Recall | Scratch F1 |
|---|---:|---:|---:|---:|---:|---:|
| Class weight 적용 | 0.9090 | 0.4500 | 0.9000 | 0.0300 | 0.0200 | 0.0200 |
| Class weight 미적용 | 0.9654 | 0.7335 | 0.9629 | 0.7778 | 0.0391 | 0.0745 |
| None downsampling | 0.9632 | 0.7886 | 0.9633 | 0.7722 | 0.3408 | 0.4729 |

None downsampling 모델은 test accuracy 0.9632를 기록하였다. 이는 class weight 미적용 모델의 0.9654와 거의 유사한 수준이다.

그러나 macro F1-score는 0.7335에서 0.7886으로 개선되었다. 이는 전체 accuracy는 거의 유지하면서, 클래스별 균형 성능이 개선되었음을 의미한다.

---

## 5. Scratch 클래스 성능 분석

이번 실험에서 가장 중요한 개선은 Scratch 클래스에서 나타났다.

Class weight 미적용 모델의 Scratch 성능은 다음과 같았다.

```text
Precision: 0.7778
Recall:    0.0391
F1-score:  0.0745
```

None downsampling 모델의 Scratch 성능은 다음과 같다.

```text
Precision: 0.7722
Recall:    0.3408
F1-score:  0.4729
```

Scratch precision은 0.7778에서 0.7722로 거의 유지되었다. 반면 Scratch recall은 0.0391에서 0.3408로 크게 향상되었다.

즉, none downsampling 모델은 Scratch가 아닌 샘플을 Scratch로 과하게 예측하지 않으면서도, 실제 Scratch 샘플을 더 많이 탐지할 수 있게 되었다.

이는 기존 class weight 미적용 모델이 보였던 “Scratch를 거의 탐지하지 못하는 문제”를 크게 완화한 결과이다.

---

## 6. 오분류 패턴 분석

None downsampling 모델의 주요 오분류 패턴은 다음과 같다.

```text
none → Loc: 175
Edge-Loc → none: 119
Loc → none: 79
Edge-Loc → Loc: 74
none → Edge-Loc: 74
Scratch → none: 69
Scratch → Loc: 44
none → Scratch: 11
```

특히 중요한 점은 `none → Scratch` 오분류가 11건으로 낮게 유지되었다는 점이다.

Class weight를 적용한 실험에서는 소수 클래스를 과하게 예측하는 문제가 발생할 수 있었지만, none downsampling에서는 Scratch precision을 높게 유지하면서 recall을 개선하였다.

다만 Scratch 샘플 중 일부는 여전히 `none` 또는 `Loc`로 오분류되었다. 이는 Scratch 패턴이 약하거나 국소 결함 패턴과 시각적으로 유사한 경우 모델이 구분에 어려움을 겪을 수 있음을 의미한다.

---

## 7. 실험 해석

이번 실험 결과는 다음과 같이 해석할 수 있다.

1. `none` 클래스의 과도한 비중은 소수 결함 클래스 탐지 성능을 낮출 수 있다.
2. 단순히 `class_weight`를 적용하는 방식은 소수 클래스 성능을 항상 안정적으로 개선하지는 않는다.
3. Train set에서 `none` 클래스 수를 줄이는 none downsampling 방식은 전체 accuracy를 거의 유지하면서 macro F1-score를 개선하였다.
4. 특히 Scratch 클래스의 recall과 f1-score가 크게 향상되었다.
5. None downsampling은 class weight 미적용 모델보다 결함 탐지 관점에서 더 균형 잡힌 모델을 만들 수 있었다.

따라서 현재까지의 실험 기준으로는 none downsampling 모델이 가장 적절한 개선 baseline으로 판단된다.

---

## 8. 현재까지의 결론

CNN baseline 실험에서는 전체 accuracy가 높게 나타났지만, 클래스 불균형으로 인해 소수 결함 클래스의 성능을 별도로 분석할 필요가 있었다.

Class weight 적용 여부를 비교한 결과, class weight 방식은 실험 조건에 따라 성능이 불안정할 수 있음을 확인하였다. 반면 class weight를 사용하지 않은 모델은 전체 accuracy는 높았으나 Scratch 클래스의 recall이 매우 낮았다.

None downsampling 실험은 이러한 한계를 완화하였다. Test accuracy는 0.9632로 높게 유지되었고, macro F1-score는 0.7886으로 개선되었다. 특히 Scratch f1-score는 0.0745에서 0.4729로 크게 향상되었다.

따라서 본 프로젝트에서는 단순 accuracy보다 macro F1-score, 클래스별 precision / recall / f1-score, normalized confusion matrix, 오분류 패턴을 함께 고려해야 한다.

---

## 9. 다음 작업

다음 단계에서는 none downsampling 모델을 기준으로 다음 작업을 진행할 수 있다.

1. None downsampling 결과를 기존 baseline 결과와 함께 README 또는 프로젝트 문서에 반영한다.
2. Scratch, Loc, Edge-Loc 오분류 샘플을 추가로 시각화하여 정성적 분석을 보강한다.
3. Grad-CAM을 적용하여 모델이 실제 결함 영역을 보고 판단하는지 확인한다.
4. 필요하다면 Scratch / Loc / Edge-Loc 중심의 data augmentation 실험을 추가로 진행한다.

현재 기준으로는 바로 복잡한 모델을 추가하기보다, none downsampling 모델을 중심으로 오분류 샘플과 Grad-CAM 분석을 진행하는 것이 적절하다.

---

## 10. GitHub 반영 대상

이번 실험과 관련하여 GitHub에 반영할 파일은 다음과 같다.

```text
notebooks/02_cnn_baseline_misclassification_analysis.ipynb
outputs/reports/cnn_none_downsampling_result.md
```

모델 파일과 대용량 데이터 파일은 GitHub에 직접 업로드하지 않는다.

GitHub에 올리지 않는 파일 예시는 다음과 같다.

```text
data/processed/*.npy
outputs/models/*.keras
outputs/models/*.h5
```
