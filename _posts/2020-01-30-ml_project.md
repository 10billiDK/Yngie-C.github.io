---
layout: post
title: 2. 머신러닝 프로젝트 처음부터 끝까지
category: Machine Learning
tag: Machine-Learning
---

 

## 1) 큰 그림 보기

- 비즈니스의 문제는 무엇이고, 현재 솔루션은 어떻게 구성되어 있는가?

- 어떤 방법을 사용할 것인가?  지도/비지도/강화 학습 중 어떤 것인가? (만약 지도학습이라면) 분류인가 회귀인가?  배치/온라인 학습 중 어떤 것을 사용할 것인가?

- 성능 측정 지표는 무엇인가? (회귀 문제에서) 많이 사용되는 성능 측정 지표(RMSE, MAE) 중 어떤 것인가?

  - __평균 제곱근 오차__ (Root Mean Square Error, RMSE)

  $$
  RMSE(\mathbf{X},h) = \sqrt{\frac{1}{m}\sum_i^m(h(\mathbf{X^i}-y^i))^2}
  $$

  $m$ : 측정 데이터셋에 있는 샘플 수

  $\mathbb{X}^i$ : i 번째 샘플의 전체 특성값 벡터 (레이블 제외)

  $y^i$ : 해당 레이블(샘플의 기대 출력값)

  $h$ : 시스템의 예측 함수, 가설이라고도 함

  $RMSE(\mathbf{X},h)$ 는 가설h를 사용하여 샘플을 평가하는 __비용함수(Cost Function)__

  - __평균 절대 오차__ (Mean Absolute Error, MAE) : 이상치로 보이는 구역이 많은 경우에 사용할 수 있다.

  $$
  MAE(\mathbf{X},h) = \frac{1}{m}\sum_i^m \vert h(\mathbf{X^i}-y^i)\vert
  $$

  RMSE와 MAE 모두 예측값의 벡터와 타깃값의 벡터 사이의 거리를 재는 방법. 다만 둘의 거리 측정 방법인 노름(norm)이 다르다. 노름에 대한 정보는 [김태완님 블로그](http://taewan.kim/post/norm/) 를 참조하자.

<br/>

## 2) 데이터 가져오기

- 데이터를 가져오고 훑어보자. (블로그 내의 [Pandas Basic](https://yngie-c.github.io/python/2020/01/29/pandasbasic/) 참조)
- 데이터 세트를 훈련 세트(Train set)와 테스트 세트(Test set)로 나눈다. (Train-Test Split, 기억이 나지 않는다면[이전 게시물]([https://yngie-c.github.io/hands%20on%20machine%20learning/2020/01/28/homl1_1/](https://yngie-c.github.io/hands on machine learning/2020/01/28/homl1_1/)) 의 마지막 부분을 참조해도 좋습니다.)

<br/>

## 3) 데이터 이해를 위한 탐색과 시각화

- 무작정 시각화 해보기
- 연속형 변수에 대한 상관관계 조사
  - 상관계수에 대한 내용은 [여기](https://mansoostat.tistory.com/115) 를 참조하도록 하자. 일반적으로는 피어스만 상관계수를 사용한다.
- 특성을 조합하여 새로운 특성 만들기를 시도해보자.

<br/>

## 4) 머신러닝 알고리즘을 위한 데이터 준비

- 데이터 정제 : NaN값 처리하기

  - 해당 구역을 삭제하는 방법 -  `.dropna()`
  - 전체 특성을 삭제하는 방법 -  `.drop()`
  - 특정 값으로 채우는 방법(0, 평균, 중간값 등) - `.fillna()`
  - 사이킷런에서는 `Imputer` 클래스를 사용하여 NaN값을 쉽게 다룰 수 있다.

  ```python
  from sklearn.preprocessing import Imputer
  
  imputer = Imputer(strategy="median")
  imputer.fit(df)
  #df의 특성이 수치형일 경우에만 사용할 수 있다.
  ```

- 텍스트와 범주형 특성 다루기

  - 범주형 특성을 어떻게 다룰 수 있을까?

  ```python
  df_reg = df['region']
  #'region'에는 '서울', '인천', '경기', '대전', '충남', '충북', '강원'이 있다고 가정하자.
  df_reg.head(10)	
  > '서울' '대전' '서울' '인천' '대전' '대전' '서울' '인천' '인천' '경기'
  
  #일 때, pandas의 .factorize() 메서드를 사용하면 각 카테고리가 정수 값으로 매칭된다.
  df_reg_encoded = df_reg.factorize()
  df_reg_encoded[:10]
  > array([0, 1, 0, 2, 1, 1, 0, 2, 2, 3])
  
  #이렇게 되면 기계는 자신과 가까운 값을 더 비슷하다고 생각하게 됨. ex) '서울-대전'을 '서울-인천','서울-경기'보다 유사하다고 여기게 된다. 하지만 실제로는 그렇지 않은 경우가 많다. 그래서 등장하게 된 것이 원-핫 인코딩이다.
  ```

  - __원-핫 인코딩(one-hot encoding)__ : 범주형 특성의 값이 정해질 때 1이 되고, 다른 값이 0이 되도록 한다. 위의 예를 원-핫 인코딩으로 표현해보자. 사이킷런에서는 OneHotEncoder를 통해 숫자로 된 범주형 값을 원-핫 벡터로 바꿀 수 있다.

  ```python
  #df_reg.head(10)을 원-핫 인코딩으로 표현한 후 toarray()를 사용해 넘파이 배열로 변형하면 다음과 같은 결과가 나온다.
  
  from sklearn.preprocessing import OneHotEncoder
  encoder = OneHotEncoder()
  df_reg_1hot = encoder.fit_transform(df_reg_encoded)
  df_reg_1hot.toarray()
  >[[1, 0, 0, 0, 0, 0, 0],
    [0, 1, 0, 0, 0, 0, 0],
    [1, 0, 0, 0, 0, 0, 0],
    [0, 0, 1, 0, 0, 0, 0],
    ...
    [0, 0, 1, 0, 0, 0, 0],
    [0, 0, 0, 1, 0, 0, 0]]
  
  #(책에는 나와있지 않지만) pandas의 .get_dummies()를 활용하여 원-핫 인코딩을 진행하는 방법도 있다.
  ```

- **특성 스케일링** : 머신러닝 알고리즘은 숫자 특성들의 스케일이 많이 다르면 잘 작동하지 않는다. 따라서 값들의 스케일이 많이 차이나는 경우 특성의 범위를 통일하기 위한 방법을 사용한다.

  - min-max 스케일링(정규화, normalization) : 사이킷런에서는 MinMaxScaler 변환기를 제공한다. feature_range 매개변수를 따로 설정하여 설정 범위를 (0,1)에서 다른 것으로 바꿀 수도 있다.

  $$
  Normalization = \frac{\bar{X} - X_{min}}{X_{max}-X_{min}}
  $$

  - 표준화(Standardization) : 범위의 상한과 하한이 없기 때문에 특정 알고리즘에는 문제가 될 수 있다. 하지만 이상치에 영향을 덜 받게 된다. 사이킷런에서는 StandardScaler 변환기가 있다.

  $$
  Z = \frac{\bar{X}-m}{\sigma}
  $$

- 변환 파이프라인 : 연속된 변환을 순서대로 처리할 수 있도록 도와주는 `Pipeline` 클래스. `Pipeline` 을 이용하면 데이터의 전처리와 머신러닝 학습 과정을 통일된 API 기반에서 처리할 수 있어 더욱 직관적인 ML 모델 코드를 생성할 수 있다. 일반적으로 사이킷런의 `Pipeline` 은 텍스트 기반의 피처 벡터화 뿐만 아니라 모든 데이터의 전처리 작업과 Estimator를 결합할 수 있다.

```python
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ('tfidf_vect', TfidVectorizer(stop_words='english', ngram_range=(1,2), max_df=300)),
    ('lr_clf', LogisticRegression(C=10))
])
# 
```

`Pipeline` 에 대한 설명과 코드는 [파이썬 머신러닝 완벽 가이드](http://www.yes24.com/Product/Goods/87044746?scode=032&OzSrank=1), 권철민, 위키북스, 2019 를 참고하였습니다.

<br/>

## 5) 모델 선택과 훈련

- 훈련 세트에서 훈련하고 평가하기
  - 1장에서 말했듯 훈련 세트에서 훈련한 것으로 1번의 테스트 세트를 통한 평가를 받고 제품을 출시하기엔 너무 많은 위험이 있다. 따라서 아래와 같이 훈련 세트 내에 검증을 위한 세트를 만들어 테스트 세트 이전에 평가를 받는다. 
- 교차 검증을 사용한 평가 : K-겹 교차 검증
  - '폴드(fold)'라고 불리는 K개의 서브셋으로 무작위로 분할한 후 그 중 하나를 검증 세트로 하는 K개의 데이터 셋을 만든다. (아래 그림에서의 Test set는 Validation set과 같은 의미이다.)

<p align="center"><img src="https://cdn-images-1.medium.com/max/1600/1*rgba1BIOUys7wQcXcL4U5A.png" alt="KFold" style="zoom:50%;" /></p>
사이킷런에서는 K-겹 교차검증을 위해 `KFold` 와 `StratifiedKFold` 클래스를 제공하고 있다. `StratifedKFold` 는 데이터셋의 불균형한 분포도를 가진 레이블 데이터 집합을 위한 K폴드 방식이다. 아래는 사이킷런에서 이들을 적용할 수 있는 코드이다.

```python
from sklearn.model_selection import KFold
kfold = KFold(n_split=5)
# n_split은 몇 개의 서브셋으로 나눌 것인지에 대한 수치이다. 아래는 StratifiedKFold를 적용하는 코드이다.
from sklearn.model_selection import StratifiedKFold
skfold = StratifiedKFold(n_split=5)
```

<br/>

## 6) 모델 세부 튜닝

- 세부 튜닝 방법
  - **그리드 탐색** (GridSearchCV) : 우리의 탐색 방법에 맞는 하이퍼 파라미터를 찾아야 합니다. 사이킷런에서는 이를 위한 `GridSearchCV` 라는 API를 제공한다. `GridSearchCV` 는 가능한 모든 하이퍼 파라미터 조합을 탐색하고 그 조합에 대해 모두 교차 검증을 사용한다. 아래는 사이킷런에서 `GridSearchCV` 를 적용할 수 있는 코드이다

```python
from sklearn.model_selection import GridSearchCV

grid_parameters = {'hyper_para1':[1, 2, 3],
                  'hyper_para2':[5,7],
                  }
grid_model = GridSearchCV(model_name, param_grid=grid_parameters, cv=5, refit=True)
#hyper_para는 하이퍼파라미터이며 적용하는 머신러닝 모델(model_name)에 따라 달라질 수 있다.
#refit이 True일 경우 가장 성능이 좋은 하이퍼파라미터 설정으로 재학습시킨다.
```



- 랜덤 탐색
  - 하이퍼 파라미터의 조합의 개수가 많아지면 `GridSearchCV` 를 사용하기에 컴퓨터 자원이 너무 많이 소모된다. 그럴 때는 `RandomizedSearchCV` 를 사용하는 것이 좋다. 후자는 모든 조합 대신 임의의 수를 대입하여 지정한 횟수만큼 평가를 진행한다. 예를 들어, 랜덤 탐색을 1,000회 반복하도록 실행하면 하이퍼파라미터마다 각기 다른 1,000개의 값을 탐색한다.
- 앙상블(Ensemble) 방법 : 최상의 모델들을 연결하여 더 나은 성능을 발휘하는 그룹을 만드는 방법.
- 최상의 모델과 오차분석
- 테스트 세트로 시스템 평가 : 테스트 세트에서 최종 모델을 평가해본다. 이 과정에서 테스트 세트에서 성능이 약간 낮아진다고 하이퍼 파라미터를 다시 수정하면 안된다. 만약 수정하면 테스트 세트에 대한 성능은 올라갈 지 모르지만, 새로운 데이터에 대해서는 일반화 되기 어렵기 때문이다.

