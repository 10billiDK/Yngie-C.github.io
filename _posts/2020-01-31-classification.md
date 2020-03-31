---
layout: post
title: 3. 분류
category: Machine Learning
tag: Machine-Learning
---

 

## 1) MNIST

미국 고등학생과 인구조사국 직원들이 쓴 손글씨 데이터(0 부터 9 까지)이다. 총 데이터 셋의 개수는 7만 개이다. 6만 개를 훈련 세트로, 1만 개를 테스트 세트로 각각 할당한다.

```python
X_train, X_test, y_train, y_test = X[:60000], X[60000:], y[:60000], y[60000:]
```



<br/>

## 2) 이진 분류기

문제를 단순화해보자. 이를테면, 5인 것과 5가 아닌 것으로 분류하기로 하자. 이렇게 O와 X, 2개의 클래스로 분류하는 것을 이진 분류기(binary classifier)라고 한다. 본 문제에서는 이진 분류 모델로 확률적 경사 하강법(Stochastic Gradient Descent, SGD)을 사용한다. 사이킷런에서는 `SGDClassifier` 클래스를 제공한다. (확률적 경사 하강법에 대한 자세한 내용은 [이곳]([https://yngie-c.github.io/hands%20on%20machine%20learning/2020/02/01/homl4/](https://yngie-c.github.io/hands on machine learning/2020/02/01/homl4/)) 에서 볼 수 있다.)

```python
y_train_5 = (y_train == 5)
y_test_5 = (y_test == 5)

from sklearn.linear_model import SGDClassifier

sgd_clf = SGDClassifier(max_iter=5, random_state=42)
sgd_clf.fit(X_train, y_train_5)
```



<br/>

## 3) 성능 측정

- 교차 검증을 사용한 정확도 측정

  - 교차검증 직접 구현하기 : 사이킷런이 제공하는 기능보다 교차 검증 과정을 더 많이 제어해야 할 필요가 있다. 이럴 때는 다음과 같은 코드로 cross_val_score() 함수를 직접 구현할 수 있다.

  ```python
  from sklearn.model_selection import StratifiedKFold
  from sklearn.base import clone
  
  skfolds = StratifiedKFold(n_splits=3, random_state=42)
  
  for train_index, test_index in skfolds.split(X_train, y_train_5):
      clone_clf = clone(sgd_clf)
      X_train_folds = X_train[train_index]
      X_train_folds = y_train_5[train_index]
      X_test_fold = X_train[test_index]
      X_test_fold = y_train_5[test_index]
      
      clone_clf.fit(X_train_folds, y_train_folds)
      y_pred = clone_clf.predict(X_test_fold)
      n_correct = sum(y_pred == y_test_fold)
      print(n_correct / len(y_pred))
  ```

  ※ (특히 불균형한 데이터에서는) __정확도를 분류기의 성능 측정 지표로 선호하지 않는다.__ 

- **오차 행렬** : 분류기의 성능을 평가하는 더 좋은 방법. '0'이 '1'로 잘못 분류된 횟수를 센다. 오차 행렬을 만들기 위해서는 실제 타깃과 비교할 수 있도록 먼저 예측값을 만들어야 한다. 사이킷런에서는 `confusion_matrix()` 함수를 사용하여 오차행렬을 출력할 수 있다.

  ```python
  from sklearn.model_selection import cross_val_predict
  y_train_pred = cross_val_predict(sgd_clf, X_train, y_train_5, cv=3)
  
  from sklearn.metrics import confusion_matrix
  confusion_matrix(y_train_5, y_train_pred)
  >>>[[53272, 1307]
     [ 1077, 4344]]
  ```

  - 오차 행렬의 행은 **실제 클래스** 를 나타내고 열은 **예측 클래스** 를 나타낸다.

  |                | 5 가 아닌 것으로 판단 | 5 로 판단 |
  | -------------- | --------------------- | --------- |
  | *실제로 5 (X)* | 53272                 | 1307      |
  | *실제로 5 (O)* | 1077                  | 4344      |
  
- **정밀도와 재현율**

  - 정밀도(Precision)

  $$
  \text{Precision} = \frac{\text{TP}}{\text{TP+FP}}
  $$

  ​			$TP$ : 진짜 양성의 수,  $FP$ : 거짓 양성의 수

  - 재현율 (Recall) : 민감도(sensitivity) 또는 진짜 양성 비율(true positive)이라고도 함
  
  $$
  \text{Recall} = \frac{\text{TP}}{\text{TP+FN}}
  $$

  ​			$FN$ : 거짓 음성의 수

  - F1 점수 (F1 Score) : 정밀도와 재현율의 조화평균으로 구해진다
  
  $$
F = \frac{2}{(1/\text{Precision})+(1/\text{Recall})} = \frac{\text{TP}}{\text{TP}+\frac{\text{FN+FP}}{2}}
  $$

  아래는 사이킷런에서 혼동행렬부터 F1점수까지 볼 수 있는 코드이다.
  
  ```python
  from sklearn.metrics import confusion_matrix, precision_score, recall_score, f1_score

  confusion = confusion_matrix(y_train_5, y_train_pred)
  precision = precision_score(y_train_5, y_train_pred)
  recall = recall_score(y_train_5, y_train_pred)
  f1 = f1_score(y_train_5, y_train_pred)
  
  print(confusion)
  print(precision, recall, f1)
  ```
  
  - **정밀도/재현율 트레이드오프** : 정밀도를 올리면 재현율이 줄고, 재현율을 높이면 정밀도가 내려가기 마련. 특정 수치가 강조되어야 할 경우에는 분류의 결정 임곗값(Threshold)을 설정하여 정밀도 또는 재현율의 수치를 확보할 수 있다. 사이킷런에서는 레이블을 분류하기 위해 `decision_function()` 과 `predict_proba()` 클래스를 제공한다. (두 클래스에 대한 설명은 [안수빈님 깃허브](https://subinium.github.io/MLwithPython-2-4/) 에서 더 자세하게 볼 수 있다.)
    - **임곗값** (Threshold) : `decision_function()` 과 `predict_proba()` 의 리턴값으로부터 클래스(레이블)를 나누는 기준이 되는 값이다. 분류 결정 임곗값은 *Positive* 예측값을 결정하는 확률의 기준이 되므로 임곗값을 낮출수록 **Recall** 이 증가하게 된다. 하지만 FP 값 또한 올라가게 되면서 **Precision** 은 줄어들게 된다. 반대로 임곗값을 높이면 **Recall** 은 낮아지고 **Precision** 은 높아지게 된다.
    - 정밀도-재현율 곡선 ( `precision_recall_curve()` ) : 여러 임곗값(일반적으로 0.11 ~ 0.95)을 담은 넘파이 ndarray와 이 임곗값에 해당하는 정밀도 및 재현율 값을 담은 넘파이 ndarray를 반환한다.
  
  ```python
  from sklearn.metrics import precision_recall_curve
  
  pred_proba_class1 = sgd_clf.predict_proba(X_test)[:,1]
  precisions, recalls, threshold = precision_recall_curve(y_test_5, pred_proba_class1)
  ```
  
  - 정밀도와 재현율의 **맹점** : 정확도뿐만 아니라 정밀도와 재현율에도 맹점이 있다. 앞서 살펴본 것처럼 *Positive* 값을 높일수록 정밀도, 재현율 중 하나의 값이 올라간다. 예를 들어, 극단적으로 임곗값을 낮춘 경우 Recall 은 1에 가까워진다. 시민 1000명 속에 있는 10명의 도둑을 잡기위해 1000명 모두를 용의자로 잡아들이는 극단적인 일이 일어난다. 반대로 임곗값을 매우 높게 올리면 Precision이 1에 가까워진다. 정말 도둑같은 사람 하나를 용의자로 세워 맞춘 뒤, 나머지 도둑 9명을 포함한 999명의 시민을 도둑에서 제외할 때 이런 수치가 나오게 된다. 따라서, 정밀도나 재현율은 비즈니스에 맞게 조정되어야 하고, 한쪽에 적정한 가중치를 두거나 두 가지 모두 고려하는 F1점수를 참고하는 것이 좋다. 



- **ROC 곡선** (Receiver Operating Characteristic, 수신기 조작 특성) : 이진분류에서 널리 사용된다. 정밀도/재현율 곡선과 비슷하지만 다르다.

  - **거짓 양성 비율**(FPR)에 대한 **진짜 양성 비율**(TPR, 재현율)의 곡선. 음성으로 정확하게 분류한 음성 샘플의 비율으로 구한다. FPR은 1에서 **진짜 음성 비율**(TNR, 특이도)을 뺀 값과도 같다.

  $$
  \text{ROC Curve} = \frac{\text{TPR}}{\text{FPR}} = \frac{\text{TPR}}{\text{1 - TNR}}
  $$

  > $TPR$ (민감도) : 실제 Positive값이 정확히 예측되어야 하는 수준 (도둑을 도둑으로 판정) = Recall
  >
  > $TNR$ (특이도) : 실제 Negative값이 정확히 예측되어야 하는 수준 (일반 시민을 일반 시민으로 판정)

  사이킷런에서는 ROC Curve를 그리기 위한 `roc_curve()` 함수가 존재하며 함수는 FPR, TPR, Threshold를 리턴값으로 반환한다.

  ```python
  from sklearn.metrics import roc_curve
  
  pred_proba_class1 = sgd_clf.predict_proba(X_test)[:, 1]
  fprs, tprs, thresholds = roc_curve(y_test, pred_proba_class1)
  ```

  

  - **AUC** : ROC 곡선 아래의 면적을 AUC라고 한다. 완전 랜덤한 분류기의 AUC는 0.5이며 완벽한 분류기의 AUC는 1이다. 1에 가까워질 수록 좋은 수치이며 사이킷런에서는 `roc_auc_score()` 함수를 사용하여 구할 수 있다.

  ```
  from sklearn.metrics import roc_auc_score
  
  pred = sgd_clf.predict(X_test)
  roc_score = roc_auc_score(y_test, pred)
  print(roc_score)
  ```

  <p align="center"><img src="https://t1.daumcdn.net/cfile/tistory/262E8E3F544837AD27" alt="AUC" style="zoom:150%;" /></p>

- 오차 행렬, 정확도, 정밀도, 재현율, 그리고 ROC 곡선

![ConfusionMatrix](https://miro.medium.com/max/625/1*y4HwoAEgx1Js19hCkPM7XA.png)

그래프에서 $\text{TN = ①, FP = ②, FN = ③, TP = ④}$ 라고 하면 정확도, 정밀도, 재현율 등은 아래와 같이 표현된다.
$$
\text{Accuracy} = \frac{\text{TN + TP}}{\text{TN + FP + FN + TP}} = \frac{① + ④}{① +② + ③ + ④}
$$

$$
\text{Precision} = \frac{\text{TP}}{\text{TP+FP}} = \frac{④}{②+④}
$$

$$
\text{Recall} = \frac{\text{TP}}{\text{TP+FN}} = \frac{④}{③+④}
$$

$$
\text{TPR} = \text{Recall} = \frac{\text{TP}}{\text{TP+FN}} = \frac{④}{③+④} \text{,     FPR} = \frac{\text{FP}}{\text{FP+TN}} = \frac{②}{① + ②} = 1 - \text{TNR} = 1 - \frac{①}{① + ②}
$$

<br/>

## 4) 다중 분류

다중 분류기는 둘 이상의 클래스를 구분할 수 있다. 다중 분류기는 크게 두 가지로 나뉨

- 여러 개의 클래스를 직접 처리할 수 있는 다중 분류기 : 랜덤 포레스트, 나이브 베이즈 등

- 이진 분류만 가능한 가능한 분류기 : 서포트 벡터 머신(SVM), 선형 분류기 등

  - **일대다** (One-versus-All, OvA) 전략 : 특정 클래스 하나만 구분하는 이진 분류기 10개를 훈련시켜 결정 점수 중에서 가장 높은 클래스를 선택하는 방법
  - **일대일** (One-versus-One, OvO) 전략 : 각 클래스의 조합마다 이진 분류기를 훈련시키는 것. 클래스가 $N$ 개일 경우 $N*(N-1)/2$ 개의 이진 분류기가 필요함.
  - SVM 같은 특정 이진 분류 알고리즘은 훈련 세트의 크기에 민감하기 때문에 훈련 세트가 작은 OvO를 선호하지만, 대부분의 이진분류 알고리즘은 OvA를 선호한다. 사이킷런에서 OvO나 OvA를 사용하도록 강제하려면 OneVsOneClassifier나 OneVsRestClassifier를 사용한다. 예를 들어, SGDClassifier로 OvO 전략을 사용하고자 한다면 다음과 같은 코드를 사용하면 된다.

  ```python
  from sklearn.multiclass import OneVsOneClassifier
  ovo_clf = OneVsOneClassifier(SGDClassifier(max_iter=5, random_state=42))
  ovo_clf.fit(X_train, y_train)
  ```

만약 샘플을 바로 다중 클래스로 분류할 수 있는 랜덤 포레스트 같은 경우에는 OvO나 OvA를 적용할 필요가 없다. `predict_proba()` 메서드를 호출하면 분류기가 각 샘플에 부여한 클래스별 확률을 얻을 수 있다.

```python
forest_clf.fit(X_train, y_train)
```



<br/>

## 5) 에러 분석

모델의 성능을 향상시키기 위해 에러의 종류를 분석.

- 오차 행렬 살펴보기

```python
y_train_pred = cross_val_predict(sgd_clf, X_train_scaled, y_train, cv=3)
#X_train_scaled는 X_train을 StandardScaler()로 표준화를 진행한 데이터
conf_mx = confusion_matrix(y_train, y_train_pred)
```

오차 행렬을 분석하여 분류기의 성능 향상 방안에 대한 통찰을 얻는다. 분류가 잘 되지 않는 클래스가 어떤 것인지 확인하고, 그 클래스에 해당하는 데이터를 더 모으거나 분류기에 도움 될 만한 특성을 더 찾아볼 수 있다. 

<br/>

## 6) 다중 레이블 분류

 분류기가 샘플마다 여러 개의 클래스를 출력해야 하는 경우도 있음. 여러 명이 등장하는 사진에 어떤 사람이 있는지 판단하기 위해서는 한 장의 사진에 여러 개의 레이블을 할당해야 함. MNIST 예제로 다중 레이블 분류의 예를 살펴보자.

```python
from sklearn.neighbors import KNeighborsClassifier
y_train_large = (y_train >= 7)
y_train_odd = (y_train % 2 == 1)
y_multilabel = np.c_[y_train_large, y_train_odd]
# 해당 손글씨가 7보다 큰 숫자를 쓴 것인지, 그리고 홀수를 쓴 것인지 동시에 판단
knn_clf = KNeighborsClassifier()
knn_clf.fit(X_train, y_multilabel)
```

<br/>

## 7) 다중 출력 분류

**다중 출력 다중 클래스 분류** (다중 출력 분류) : 다중 레이블 분류에서 한 레이블이 다중 클래스가 될 수 있도록 일반화한 것. MNIST 이미지의 노이즈를 제거하는 시스템을 만든다고 해보자. 분류기의 출력이 다중 레이블(1픽셀당 1레이블)이고 각 레이블은 여러 개의 값(0~255의 픽셀 농도)을 가짐. 그러면 이 시스템은 다중 출력 분류 시스템이 되어야 한다.

[이 예시에서처럼 분류와 회귀 사이의 경계는 때때로 모호하다. 레이블의 픽셀 농도는 분류보다 회귀와 비슷하기 때문이다. 더구나 다중 출력 시스템은 분류 작업에 국한되지도 않는다.]