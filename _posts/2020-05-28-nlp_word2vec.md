---
layout: post
title: Word2Vec
category: NLP
tag: NLP
---



본 포스트의 내용은 [고려대학교 강필성 교수님의 강의](https://www.youtube.com/watch?v=pXCHYq6PXto&list=PLetSlH8YjIfVzHuSXtG4jAC2zbEAErXWm) 와 [김기현의 자연어처리 딥러닝 캠프](http://www.yes24.com/Product/Goods/74802622) , [밑바닥에서 시작하는 딥러닝 2](http://www.yes24.com/Product/Goods/72173703) , [한국어 임베딩](http://m.yes24.com/goods/detail/78569687) 책을 참고하였습니다.



# Word2Vec

**Word2Vec** 에는 **CBoW(Continuous Bag-of-Words)** 와 **Skip-gram** 의 두 가지 방법이 있다. 전자는 주변 단어(Context, 문맥)로부터 타깃 단어 $w_t$ 를 예측하고 후자는 타깃 단어로부터 주변 단어를 예측한다. 얼핏 생각해보면 더 많은 정보를 받아 하나의 단어를 예측하는 CBoW의 성능이 더 좋을 것으로 예측하기 쉽다. 하지만 실제로는 역전파가 일어나는 관점에서 생각해보면 CBoW는 하나의 단어에서 오는 정보를 여러 단어에 흘려주고, Skip-gram은 여러 단어로부터 흘러나오는 역전파 정보를 한 단어가 흘려받는다. 때문에 Skip-gram의 성능이 보통 더 잘 나오는 편이다. 아래는 CBoW와 Skip-gram모델을 그림으로 도식화한 것이다.

<p align="center"><img src="https://user-images.githubusercontent.com/45377884/85476034-3985bb00-b5f2-11ea-9894-4b7812edcc3c.png" alt="word2vec1" style="zoom:67%;" /></p>

<p align="center" style="font-size:80%">이미지 출처 : <a href="https://github.com/pilsung-kang/text-analytics">Text-Analytics Github</a></p>

Skip-gram이 진행되는 과정을 살펴보자. 내부에 활성화 함수가 없는 선형적(Linear)인 구조를 가지고 있다. Skip-gram의 목표는 타깃 단어 $w_t$ 로부터 주변 단어를 예측하는 모델을 만드는 것이므로 아래와 같이 목적 함수를 표현할 수 있다.

$$
J(\theta) = \frac{1}{T} \sum^T_{t=1} \sum_{-m \leq j \leq m, j \neq 0} \log p(w_{t+j}|w_t)
$$


위 목적 함수에서 $m$ 은 Window의 크기를 나타내며 주변 단어의 확률을 최대화 하는 파라미터 셋 $\theta$ 를 구하는 것을 목적으로 한다. 각 주변 단어의 확률은 다음과 같이 구해진다. 한번에 역전파 정보를 받는 것과 따로 받아 더해 준 결과가 같으므로 실제 역전파가 흘려받을 때는 각 주변 단어에서 하나씩 흘려받은 결과를 더해준다.

예를 들어, *"The quick brown fox jumps over..."* 로 시작하는 문장을 크기가 좌우 2인 윈도우로 학습한다면 *(the, samples), (the, brown)* / *(quick, the), (quick, brown), (quick, fox)* / *(brown, the), (brown, quick), (brown, fox), (brown, jumps)* 처럼 따로 단어 쌍을 만든 후 학습이 진행된다.


$$
p(c|w) = \frac{\exp(x_w^Tv_c)}{\sum^K_{k=1} \exp(x_w^Tv_k)} \\
x_w : \text{단어 피처 벡터} \quad v_c : \text{단어 분류기 벡터}
$$


CBoW는 주변 단어로부터 정보를 받아 타깃 단어를 예측하므로 아래와 같이 단어의 확률을 나타낼 수 있다. 여러 단어의 정보를 받기 때문에 대문자 $C$ 로 나타낸다.


$$
p(w|C) = \frac{\exp(h_c^Tv_w)}{\sum^K_{k=1} \exp(h_c^Tv_k)} \\
h_C : \text{주변 단어 피처 벡터} \quad v_w : \text{단어 분류기 벡터}
$$


다시 Skip-gram으로 돌아와 구조를 더 자세히 들여다 보자. 수식의 단순화를 위하여 $w_{t+j} \rightarrow o$ , $w_t \rightarrow c$ (outside, center)로 나타내면 Skip-gram의 확률 식을 아래와 같이 나타낼 수 있다.


$$
p(o|c) = \frac{\exp(u_o^Tv_c)}{\sum^W_{w=1} \exp(u_w^Tv_c)}
$$


이 확률을 최대화 하는 벡터인 $v_c$ 를 찾는 것이 목적이므로 위 수식에 로그를 취한 것을 $v_c$ 로 미분한 수식을 살펴보도록 하자.


$$
\frac{\partial}{\partial v_c} \log p(o|c) = \frac{\partial}{\partial v_c} \log \frac{\exp(u_o^Tv_c)}{\sum^W_{w=1} \exp(u_w^Tv_c)} \qquad \qquad \\
\qquad \qquad \quad = \frac{\partial}{\partial v_c} u_o^Tv_c-\frac{\partial}{\partial v_c} \log \sum^W_{w=1} \exp(u^T_w v_c) \\
\qquad \qquad \qquad \qquad \quad = u_o - \frac{1}{\sum^W_{w=1} \exp(u^T_w v_c)} \cdot \sum^W_{w=1} \exp(u^T_w v_c) \cdot u_w \\
= u_o - \sum^W_{w=1} P(w|c) \cdot u_w \\
\because - \frac{1}{\sum^W_{w=1} \exp(u^T_w v_c)} \cdot \sum^W_{w=1} \exp(u^T_w v_c) = - \sum^W_{w=1} \frac{\exp(u^T_w v_c)}{\sum^W_{w=1} \exp(u^T_w v_c)} = - \sum^W_{w=1} P(w|c)
$$


확률을 최대화 하는 것이 목적이므로 경사 상승법을 적용하여 가중치 벡터 $v_c$ 를 개선해나간다.


$$
v_c(t+1) = v_c(t) + \alpha(u_o - \sum^W_{w=1} P(w|c) \cdot u_w)
$$


가중치의 개수는 $2 \times V \times N$ 으로 매우 크다. 가중치의 개수가 많으면 학습 시간이 오래걸리고 컴퓨팅 자원도 많이 사용되므로 가중치의 개수를 줄이기 위해 여러 전략을 취한다. 먼저 자주 등장하는 구(phrase)는 하나의 단어로 취급한다. 두 번째로, 너무 자주 등장하는 단어는 학습 데이터에서 누락시킨다. 고빈도 단어를 누락시키는 확률 $P$ 에 대한 식은 다음과 같다.


$$
P(w_i) = 1 - \sqrt{\frac{t}{f(w_i)}} \\
f(w_i) : w_i \text{가 등장하는 빈도,} \quad t : \text{threshold}
$$


**네거티브 샘플링(Negative Sampling)** 도 계산량을 줄이기 휘한 방법이다. Skip-gram은 한 단어를 구하기 위해서 윈도우 내 모든 단어에 대해 전부 계산을 한다. 모든 단어의 소프트맥스(Softmax)를 계산할 경우 계산량이 많아지므로 그 중 일부를 덜어낸다. 계산 대상이 단어 전체 $\vert V \vert$ 에서 이 때 목적 함수의 수식을 아래와 같이 나타낼 수 있다.


$$
J(\theta) = \frac{1}{T} \sum^T_{t=1} \sum_{-m \leq j \leq m, j \neq 0} \log p(w_{t+j}|w_t) = \frac{1}{T} \sum^T_{t=1} J_t (\theta) \\
J_t (\theta) = \log \sigma(u_o^T v_c) + \sum_{i=1}^T E_{i \sim P(W)} [\log \sigma(-u_i^T v_c)] \\
P(w_i) = \frac{f(w_i)^{3/4}}{\sum_{j=0}^n (f(w_i)^{3/4})}
$$

