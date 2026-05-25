# CNN Baseline Result Summary

## 1. 실험 개요

본 실험에서는 LSWMD wafer map 데이터를 사용하여 결함 유형을 분류하는 기본 CNN baseline 모델을 학습했다.

전처리 과정에서 누락되어 있던 `X_train.npy` 문제를 해결하기 위해 전체 라벨 데이터를 기준으로 train/validation/test 데이터를 다시 생성했다.  
최종 입력 이미지는 모두 `64x64` 크기로 통일했으며, 모델 입력 형태는 `(64, 64, 1)`이다.

## 2. 데이터 구성

최종 데이터 분할은 다음과 같다.

```text
Train: 121,065
Validation: 25,942
Test: 25,943
```

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
데이터셋은 none 클래스가 매우 많은 불균형 구조를 가지고 있다.
따라서 학습 시 class_weight를 적용했다.

## 3. 모델 구성

기본 CNN baseline 모델을 사용했다.
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
## 4. 테스트 결과

최종 test set 평가 결과는 다음과 같다.
```text
Test Loss: 0.2087
Test Accuracy: 0.9340
```
전체 정확도는 약 93.4%로 높게 나타났다.

## 5. 클래스별 성능 요약
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
## 6. 결과 해석

전체 accuracy는 93.4%로 높게 나왔지만, 데이터셋의 클래스 불균형을 고려하면 accuracy만으로 모델 성능을 판단하기는 어렵다.

none 클래스의 support가 22,115개로 매우 많기 때문에, none을 잘 맞추는 것만으로도 전체 accuracy와 weighted average가 높게 나타날 수 있다.
따라서 이 실험에서는 macro average와 클래스별 f1-score를 함께 확인하는 것이 중요하다.

모델은 none, Edge-Ring, Center 클래스에서 좋은 성능을 보였다.
```text
none f1-score: 0.97
Edge-Ring f1-score: 0.97
Center f1-score: 0.88
```
반면 Scratch, Loc, Edge-Loc 클래스에서는 상대적으로 낮은 성능을 보였다.
```text
Scratch f1-score: 0.22
Loc f1-score: 0.64
Edge-Loc f1-score: 0.67
```
특히 Scratch 클래스는 precision이 0.14로 매우 낮았다.
이는 모델이 실제 Scratch가 아닌 샘플까지 Scratch로 많이 예측했다는 의미이다.

즉, 현재 모델은 전체적인 defect classification baseline으로는 의미가 있지만, 소수 클래스와 유사 패턴 클래스에 대한 분류 성능은 아직 개선이 필요하다.

## 7. 결론

이번 CNN baseline 모델은 test accuracy 93.4%를 달성하며 전반적으로 안정적인 성능을 보였다.
그러나 클래스별 성능을 보면 Scratch, Loc, Edge-Loc에서 오분류가 많이 발생했다.

따라서 다음 실험에서는 전체 accuracy 향상보다 클래스별 불균형 문제와 오분류 패턴 분석에 집중할 필요가 있다.
