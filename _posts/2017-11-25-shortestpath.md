---
title: 최단 경로 문제
category: Data structure&Algorithm
tag: [Dijkstra's algorithm, Shortest path, Graph]
---

이번 글에서는 **최단 경로 문제(Shortest Path Problem)**를 살펴보도록 하겠습니다. 이 글은 고려대 김선욱 교수님과 역시 같은 대학의 김황남 교수님 강의와 위키피디아를 정리했음을 먼저 밝힙니다. 그럼 시작하겠습니다.





## 최단거리 문제

최단 경로 문제란 두 노드를 잇는 가장 짧은 경로를 찾는 문제입니다. 가중치가 있는 그래프(Weighted Graph)에서는 엣지 가중치의 합이 최소가 되도록 하는 경로를 찾으려는 것이 목적입니다. 최단 경로 문제엔 다음과 같은 변종들이 존재합니다.

- **단일 출발(single-source) 최단경로** : 단일 노드 $v$에서 출발하여 그래프 내의 모든 다른 노드에 도착하는 가장 짧은 경로를 찾는 문제.
- **단일 도착(single-destination) 최단경로** : 모든 노드들로부터 출발하여 그래프 내의 한 단일 노드 $v$로 도착하는 가장 짧은 경로를 찾는 문제. 그래프 내의 노드들을 거꾸로 뒤집으면 단일 출발 최단경로문제와 동일.
- **단일 쌍(single-pair) 최단 경로** : 주어진 꼭지점 $u$와 $v$에 대해 $u$에서 $v$까지의 최단 경로를 찾는 문제.
- **전체 쌍(all-pair) 최단 경로** : 그래프 내 모든 노드 쌍들 사이의 최단 경로를 찾는 문제.

다익스트라 알고리즘은 여기서 단일 출발 최단경로 문제를 푸는 데 적합합니다. 하지만 여기에서 조금 더 응용하면 나머지 세 문제도 풀 수 있습니다. 





## optimal substructure

여기에서 최단거리의 중요한 속성을 하나 짚고 넘어가겠습니다. 다익스트라 알고리즘이나 벨만-포드 알고리즘이 최단 경로를 찾을 때 써먹는 핵심 정리이기도 합니다.

- 최단경로의 부분 경로는 역시 최단경로이다.

이를 나타낸 예시 그림이 다음 그림입니다. 아래 그림에서 직선은 엣지, 물결선은 경로를 나타냅니다. 각 숫자는 거리 등 가중치를 나타냅니다. 엣지 가중치 합이 최소인 최단경로는 굵게 표시된 상단 첫번째 경로임을 알 수 있습니다. 만약 그렇다면 시작노드로부터 중간에 있는 노드에 이르기까지의 경로(5) 또한 최단경로라는 것이 위 정리의 핵심입니다.



<a href="https://imgur.com/4s5a0iz"><img src="https://i.imgur.com/4s5a0iz.png" width="400px" title="source: imgur.com" /></a>

위 정리를 증명한 내용은 다음과 같습니다.



<a href="https://imgur.com/Ldpb0WM"><img src="https://i.imgur.com/Ldpb0WM.png" width="380px" title="source: imgur.com" /></a>



이와 같이 어떤 문제의 최적 해가 그 부분문제들의 최적해들로 구성(*construct*)될 수 있다면 해당 문제는 *optimal substructure*를 가진다고 말합니다. 이 속성을 가지고 [다이내믹 프로그래밍(dynamic programming)](https://ratsgo.github.io/data%20structure&algorithm/2017/11/15/dynamic/)이나 [탐욕 알고리즘](https://ratsgo.github.io/data%20structure&algorithm/2017/11/22/greedy/)을 적용해 문제를 효율적으로 풀 수 있습니다. 





## edge relaxation

<a href="https://imgur.com/nqdnANR"><img src="https://i.imgur.com/nqdnANR.png" width="400px" title="source: imgur.com" /></a>







## 알고리즘 특성별 비교



**negative weight**

**negative cycle**

**complexity**





