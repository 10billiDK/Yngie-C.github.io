---
layout: post
title: 6. 합성곱 신경망(CNN)
category: DL from Scratch
tag: Deep-Learning
---



## 1) 전체 구조

- 합성곱 신경망은 이전의 완전 연결 층에서는 볼 수 없었던, 2개의 층(합성곱 계층 - Conv 과 풀링 계층 - Pooling)이 추가된다. (풀링 계층은 생략될 수도 있다.) 또, 층의 마지막 부분, 즉 아래 그림에서의 Classifer쪽 부분에서는 이전의 완전 연결 층을 결합하여 사용한다.

<p align="center"><img src="https://miro.medium.com/max/700/1*gpqAAvdoj-VcFlKK8y9zAQ.jpeg" alt="CNN Structure"  /></p>




<br/>

## 2) 합성곱 계층

- **완전연결 계층의 문제점** : 이미지는 통상적으로 3차원(가로, 세로, 채널) 데이터. 각 위치에 맞는 공간적 특성을 담고 있다. 완전 연결 층은 이러한 특성을 무시하고 모든 입력 값을 동등한 뉴런으로 취급하여 패턴을 살려내지 못한다. 반면, 합성곱 신경망은 3차원 데이터를 입력받아 다시 3차원 데이터를 내놓기 때문에 형상을 유지하며 학습해나갈 수 있다.



- **합성곱 연산** : 입력 데이터에 특정 크기의 커널(Kernal Matrix)을 적용하여 새로운 출력값을 계산해낸다. 아래 그림은 합성곱이 계산되는 과정이다. 이와 같이 특정 커널이 움직이며 대응하는 원소끼리 곱한 후 총합을 구해낸다. 커널은 완전 연결 층에서의 **가중치** 와 같은 역할을 한다. 합성곱 신경망에도 가중치 외에 편향이 존재하며 이는 커널과의 곱을 구한 후에 특정 값을 모든 값에 더해주는 방식으로 적용된다.

<p align="center"><img src="https://i.stack.imgur.com/Tnfmi.gif" alt="Convolutional Cal" style="zoom: 80%;" /></p>



- **패딩(Padding)** : 연산을 수행하기 전에 입력 데이터 주변을 특정 값으로 채우는 것을 패딩이라고 한다. 위에서 살펴본 그림에서 값을 0으로 하고 폭이 1인 패딩이 적용된 모습을 볼 수 있다. 
- **스트라이드(Stride)** : 필터를 적용하는 위치의 간격을 스트라이드라고 한다. 위에서 살펴본 그림은 스트라이드가 1일 때 커널이 적용되는 모습이며, 스트라이드가 2인 경우에는 2칸씩 뛰면서 커널이 적용된다. 입력 데이터의 크기와 패딩, 스트라이드를 알면 출력값의 크기를 다음과 같은 식을 통해 알 수 있다.

$$
OH = \frac{H+2P-FH}{S}+1 \\ OW = \frac{W+2P-FW}{S}+1 \\ \text{입력 크기 : (H, W)} \quad \text{필터 크기 : (FH, FW)} \quad \text{출력 크기 : (OH, OW)} \quad \text{패딩 : P, 스트라이드 : S}
$$

  

- 3차원 데이터의 합성곱 연산 : $k$ 개의 채널이 있는 경우 $k$ 개의 필터를 적용하여 병렬 연산을 한 뒤 그 합을 출력 데이터에 입력한다. 예를 들어 3개의 채널이 있는 입력 데이터는 3개의 커널과 각각 연산되어 그 총합을 새로운 출력 데이터로 받게 된다.

- 블록으로 생각하기 :

- 배치 처리 :

  

<br/>

## 3) 풀링 계층

- **풀링(Pooling)** : 풀링은 가로, 세로 방향의 공간을 줄이기 위한 연산이다. 아래 그림은 2X2 **최대 풀링(Max pooling)** 을 스트라이드 2로 처리하는 과정을 나타낸 것이다. 풀링 계층에서의 입력 데이터는 최대 풀링 과정을 통해 각 윈도우에서의 최댓값만을 남기고 줄어들게 된다. 풀링에서 스트라이드의 크기는 윈도우 크기와 같게 해주는 것이 일반적이다. 

<p align="center"><img src="https://i2.wp.com/principlesofdeeplearning.com/wp-content/uploads/2018/07/poolingmax.png?resize=768%2C339&amp;ssl=1" alt="Max pooling" style="zoom: 67%;" /></p>

최대 풀링 이외에도 평균 풀링이 있다. 평균 풀링은 각 윈도우 내부 값들의 평균을 새로운 출력 데이터 값으로 계산한다. 하지만 이미지 인식에서는 최대 풀링을 사용하는 것이 일반적이다.



- 풀링 계층의 특징
  - 학습해야 할 매개변수가 없다
  - 채널 수가 변하지 않는다
  - 입력 변화에 영향을 적게 받는다(강건하다) : 입력 데이터가 변하더라도 풀링의 결과는 크게 변하지 않는다.



<br/>

## 4) 합성곱/풀링 계층 구현하기

- **4차원** 배열 : CNN에서의 데이터는 전체 4차원 형태(개수, 채널, 높이, 너비)로 움직인다. 예를 들어 28X28 크기 흑백이미지 60000개로 구성된 MNIST 손글씨 데이터셋을 나타내면 (60000, 1. 28, 28) 이 된다.
- **im2col** (image to column) 로 데이터 전개하기 : 4차원 데이터를 연산하기 위해서 for문 대신 **im2col** 이라는 편의함수를 사용. im2col은 커널을 적용하기 좋도록 3차원 블록을 2차원 평면에 늘어놓는다.
- 합성곱 계층 구현하기

im2col 함수는 아래와 같다.

```python
im2col(input_data, filter_h, filter_w, stride=1, pad=0):
    # input_data : 4차원 배열 형태의 입력 데이터
    # filter_h, filter_w : 필터의 높이와 너비
    # stride : 스트라이드, pad : 패딩
    # col : 2차원 배열
    
    N, C, H, W = input_data.shape
    out_h = (H + 2*pad - filter_h)//stride + 1
    out_w = (W + 2*pad - filter_w)//stride + 1

    img = np.pad(input_data, [(0, 0), (0, 0), (pad, pad), (pad, pad)], 'constant')
    col = np.zeros(N, C, filter_h, filter_w, out_h, out_w)

    for y in range(filter_h):
        y_max = y + stride*out_h
        for x in range(filter_w):
            x_max = x + stride*out_w
            col[:, :, y, x, :, :] = img[:, :, y:y_max:stride, x:x_max:stride]

    col = col.transpose(0, 4, 5, 1, 2, 3).reshape(N*out_h*out_w, -1)
    return col
```

구현한 **im2col** 을 사용하여 합성곱 계층을 구현해보자.

```python
class Convolution:
    def __init__(self, W, b, stride=1, pad=0):
        self.W = W
        self.b = b
        self.stride = stride
        self.pad = pad

    def forward(self, x):
        FN, C, FH, FW = self.W.shape
        N, C, H, W = x.shape
        out_h = int(1 + (H + 2*self.pad - FH) / self.stride)
        out_w = int(1 + (W + 2*self.pad - FW) / self.stride)

        col = im2col(x, FH, FW, self.stride, self.pad)
        col_W = self.W.reshape(FN, -1).T
        out = np.dot(col, col_W) + self.b
        out = out.reshape(N, out_h, out_w, -1).transpose(0, 3, 1, 2)
		# 4차원 데이터(개수, 3차원)를 받아, im2col 함수를 사용하여 2차원(개수, 1차원) 벡터로 변환
        # 가중치를 곱하고 편향을 더해준 값을 다시 4차원으로 가공하여 반환
        
        return out
```



- 풀링 계층 구현하기 : 풀링 계층의 구현도 합성곱 계층과 마찬가지로 im2col을 사용하여 입력 데이터를 전개한다. 

```python
class Pooling:
    def __init__(self, W, b, stride=1, pad=0):
        self.pool_h = pool_h
        self.pool_w = pool_w
        self.stride = stride
        self.pad = pad

    def forward(self, x):
        N, C, H, W = x.shape
        out_h = int(1 + (H - self.pool_h) / self.stride)
        out_w = int(1 + (W - self.pool_w) / self.stride)
		
        #입력 데이터 전개
        col = im2col(x, self.pool_h, self.pool_w, self.stride, self.pad)
        col = col.reshape(-1, self.pool_h*self.pool_w)

        # 최댓값 추출(Max pooling)
        out = np.max(col, axis=1)

        # 다음 Conv층에서 쓸 수 있도록 성형
        out = out.reshape(N, out_h, out_w, C).transpose(0, 3, 1, 2)

        return out
```



<br/>

## 5) CNN 구현하기

위에서 구현한 합성곱 계층과 풀링 계층을 사용하여 간단한 CNN을 만들어보자. 아래 코드로 구성되는 CNN의 구조는 Conv - ReLU - Pool - Affine - ReLU - Affine - Softmax 이다.

```python
class SimpleConvNet:
    def __init__(self, input_dim=(1, 28, 28), 
                 conv_param={'filter_num':30, 'filter_size':5, 'pad':0, 'stride':1},
                 hidden_size=100, output_size=10, weight_init_std=0.01):
        # inp
        filter_num = conv_param['filter_num']
        filter_size = conv_param['filter_size']
        filter_pad = conv_param['pad']
        filter_stride = conv_param['stride']
        input_size = input_dim[1]
        conv_output_size = (input_size - filter_size + 2*filter_pad) / filter_stride + 1
        pool_output_size = int(filter_num * (conv_output_size/2) * (conv_output_size/2))

        # 가중치 초기화
        self.params = {}
        self.params['W1'] = weight_init_std * \
                            np.random.randn(filter_num, input_dim[0], filter_size, filter_size)
        self.params['b1'] = np.zeros(filter_num)
        self.params['W2'] = weight_init_std * \
                            np.random.randn(pool_output_size, hidden_size)
        self.params['b2'] = np.zeros(hidden_size)
        self.params['W3'] = weight_init_std * \
                            np.random.randn(hidden_size, output_size)
        self.params['b3'] = np.zeros(output_size)

        # 계층 생성
        self.layers = OrderedDict()
        self.layers['Conv1'] = Convolution(self.params['W1'], self.params['b1'],
                                           conv_param['stride'], conv_param['pad'])
        self.layers['Relu1'] = Relu()
        self.layers['Pool1'] = Pooling(pool_h=2, pool_w=2, stride=2)
        self.layers['Affine1'] = Affine(self.params['W2'], self.params['b2'])
        self.layers['Relu2'] = Relu()
        self.layers['Affine2'] = Affine(self.params['W3'], self.params['b3'])

        self.last_layer = SoftmaxWithLoss()

    def predict(self, x):
        for layer in self.layers.values():
            x = layer.forward(x)

        return x

    def loss(self, x, t):
        """손실 함수를 구한다.
        Parameters
        ----------
        x : 입력 데이터
        t : 정답 레이블
        """
        y = self.predict(x)
        return self.last_layer.forward(y, t)

    def accuracy(self, x, t, batch_size=100):
        if t.ndim != 1 : t = np.argmax(t, axis=1)
        
        acc = 0.0
        
        for i in range(int(x.shape[0] / batch_size)):
            tx = x[i*batch_size:(i+1)*batch_size]
            tt = t[i*batch_size:(i+1)*batch_size]
            y = self.predict(tx)
            y = np.argmax(y, axis=1)
            acc += np.sum(y == tt) 
        
        return acc / x.shape[0]

    def numerical_gradient(self, x, t):
        """기울기를 구한다（수치미분）.
        Parameters
        ----------
        x : 입력 데이터
        t : 정답 레이블
        Returns
        -------
        각 층의 기울기를 담은 사전(dictionary) 변수
            grads['W1']、grads['W2']、... 각 층의 가중치
            grads['b1']、grads['b2']、... 각 층의 편향
        """
        loss_w = lambda w: self.loss(x, t)

        grads = {}
        for idx in (1, 2, 3):
            grads['W' + str(idx)] = numerical_gradient(loss_w, self.params['W' + str(idx)])
            grads['b' + str(idx)] = numerical_gradient(loss_w, self.params['b' + str(idx)])

        return grads

    def gradient(self, x, t):
        """기울기를 구한다(오차역전파법).
        Parameters
        ----------
        x : 입력 데이터
        t : 정답 레이블
        Returns
        -------
        각 층의 기울기를 담은 사전(dictionary) 변수
            grads['W1']、grads['W2']、... 각 층의 가중치
            grads['b1']、grads['b2']、... 각 층의 편향
        """
        # forward
        self.loss(x, t)

        # backward
        dout = 1
        dout = self.last_layer.backward(dout)

        layers = list(self.layers.values())
        layers.reverse()
        for layer in layers:
            dout = layer.backward(dout)

        # 결과 저장
        grads = {}
        grads['W1'], grads['b1'] = self.layers['Conv1'].dW, self.layers['Conv1'].db
        grads['W2'], grads['b2'] = self.layers['Affine1'].dW, self.layers['Affine1'].db
        grads['W3'], grads['b3'] = self.layers['Affine2'].dW, self.layers['Affine2'].db

        return grads
        
    def save_params(self, file_name="params.pkl"):
        params = {}
        for key, val in self.params.items():
            params[key] = val
        with open(file_name, 'wb') as f:
            pickle.dump(params, f)

    def load_params(self, file_name="params.pkl"):
        with open(file_name, 'rb') as f:
            params = pickle.load(f)
        for key, val in params.items():
            self.params[key] = val

        for i, key in enumerate(['Conv1', 'Affine1', 'Affine2']):
            self.layers[key].W = self.params['W' + str(i+1)]
            self.layers[key].b = self.params['b' + str(i+1)]
```



<br/>

## 6) CNN 시각화하기

합성곱 신경망이 어떤 방법으로 학습을 진행해나가는지 알아보자.

- 1번째 층의 가중치 시각화하기
  - 학습 전의 필터는 무작위로 초기화되고 있기 때문에 흑백 정도에 규칙성이 없다. 
  - 학습 후에는 특정



- 층 깊이에 따른 추출 정보 변화 : 위에서 볼 수 있 듯 1층에서는 엣지나 블롭 등 낮은 수준의 정보가 추출된다. 그렇다면 다른 층에서는 어떤 정보를 학습할까? 딥러닝 시각화에 관한 연구에 따르면, 계층이 깊어질수록 추출되는 정보는 더 추상화 된다고 한다. Pretrained Model인 Alexnet의 경우 1층에서는 엣지나 블롭 등을 학습하고, 3번째 층은 텍스처(Texture)를 학습한다. 더 나아가 5번째 층에서는 사물의 일부를 학습하고 최종적으로 완전 연결 층의 마지막에서는 사물의 클래스에 뉴런이 반응하게 된다.



<br/>

## 7) 대표적인 CNN

- LeNet : 손글씨 숫자를 인식하는 네트워크로, 1998년에 제안되었다. 아래는 LeNet을 그림으로 나타낸 것이다.

  ![LeNet](http://nocotan.github.io/images/20170804/lenet.png)

  - LeNet은 초기에 개발된 모델으로 현재 많이 사용되는 CNN과는 몇 가지 다른 점을 가지고 있다.
    - 활성화 함수 : LeNet의 활성화 함수는 시그모이드 함수인데 비하여, 현재는 주로 ReLU를 활성화 함수를 사용한다.
    - 데이터를 축소하는 방법 : 현재의 모델은 위에서 나온 최대 풀링을 사용하여 데이터를 단순화해나간다. 반면에 LeNet은 데이터의 축소를 위해 서브샘플링을 사용한다.



- AlexNet : 2012년에 발표된 AlexNet은 합성곱 계층과 풀링 계층을 거듭한 뒤 마지막으로 완전 연결 계층을 거쳐 결과를 출력한다. AlexNet은 활성화 함수로 ReLU를 사용하고 있다. LRN(Local Response Normalization)이라는 국소적 정규화를 실시하는 계층을 이용하며 드롭아웃을 이용해 오버피팅을 방지한다. 아래는 AlexNet을 그림으로 나타낸 것이다.



[^1]: 입력층의 경우 따로 명시하지 않는다는 의견도 있다. 