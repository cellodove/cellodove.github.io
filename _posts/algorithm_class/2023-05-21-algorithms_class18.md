---
title : "[Algorithm] 18장 그래프 이해"
excerpt: "그래프"

categories :
  - Algorithm_Class

toc: true
toc_sticky: true
last_modified_at: 2023-05-21
---

![algorithms18_image1.jpg](/assets/images/algorithms18_image1.jpg?raw=true)

## 그래프 (Graph)

그래프라 하면 일반적으로 '정점(노드)'과 '간선(엣지)'으로 이루어진 자료구조를 의미한다.

또한 간선의 방향 유무에 따라서 단방향 그래프와 무방향 그래프(또는 양방향)로 나뉜다.

### 용어

- 노드 (Node): 위치를 말함, 정점(Vertex)라고도 한다.
- 간선 (Edge): 위치 간의 관계를 표시한 선으로 노드를 연결한 선이다. (link 또는 branch 라고도 함)
- 인접 정점 (Adjacent Vertex) : 간선으로 직접 연결된 정점(또는 노드)

- 정점의 차수 (Degree): 무방향 그래프에서 하나의 정점에 인접한 정점의 수
- 진입 차수 (In-Degree): 방향 그래프에서 외부에서 오는 간선의 수
- 진출 차수 (Out-Degree): 방향 그래프에서 외부로 향하는 간선의 수
- 경로 길이 (Path Length): 경로를 구성하기 위해 사용된 간선의 수
- 단순 경로 (Simple Path): (A - B - C) 처음 정점과 끝 정점을 제외하고 중복된 정점이 없는 경로
    - A-B-C-A-B-D 와 같은 식의 경로는 중복된 정점이 있으므로 단순 경로가 아니다.
- 사이클 (Cycle): 단순 경로의 시작 정점과 종료 정점이 동일한 경우

## 그래프 종류

여러 종류의 그래프가 있다.

### 무방향 그래프 (Undirected Graph)

![algorithms18_image2.png](/assets/images/algorithms18_image2.png?raw=true)

방향이 없는 그래프

간선을 통해, 노드는 양방향으로 갈 수 있다. 보통 노드 A, B가 연결되어 있을 경우, (A, B) 또는 (B, A) 로 표기한다.

### 방향 그래프 (Directed Graph)

![algorithms18_image3.png](/assets/images/algorithms18_image3.png?raw=true)

간선에 방향이 있는 그래프

보통 노드 A, B가 A -> B 로 가는 간선으로 연결되어 있을 경우, <A, B> 로 표기 (<B, A> 는 B -> A 로 가는 간선이 있는 경우이므로 <A, B> 와 다름)

### *중치 그래프 (Weighted Graph) 또는 네트워크 (Network)

![algorithms18_image4.png](/assets/images/algorithms18_image4.png?raw=true)

간선에 비용 또는 가중치가 할당된 그래프

### 연결 그래프 (Connected Graph) 와 비연결 그래프 (Disconnected Graph)

![algorithms18_image5.png](/assets/images/algorithms18_image5.png?raw=true)

연결 그래프 (Connected Graph)

무방향 그래프에 있는 모든 노드에 대해 항상 경로가 존재하는 경우

비연결 그래프 (Disconnected Graph)

무방향 그래프에서 특정 노드에 대해 경로가 존재하지 않는 경우

### 사이클 (Cycle) 과 비순환 그래프 (Acyclic Graph)

![algorithms18_image6.png](/assets/images/algorithms18_image6.png?raw=true)

사이클 (Cycle)

단순 경로의 시작 노드와 종료 노드가 동일한 경우

비순환 그래프 (Acyclic Graph)

사이클이 없는 그래프

### 완전 그래프 (Complete Graph)

![algorithms18_image7.png](/assets/images/algorithms18_image7.png?raw=true)

그래프의 모든 노드가 서로 연결되어 있는 그래프

### 그래프와 트리의 차이

트리는 그래프 중에 속한 특별한 종류라고 할 수 있다.

|  | 그래프 | 트리 |
| --- | --- | --- |
| 정의 | 노드와 노드를 연결하는 간선으로 표현되는 자료 구조 | 그래프의 한 종류, 방향성이 있는 비순환 그래프 |
| 방향성 | 방향 그래프, 무방향 그래프 둘다 존재함 | 방향 그래프만 존재함 |
| 사이클 | 사이클 가능함, 순환 및 비순환 그래프 모두 존재함 | 비순환 그래프로 사이클이 존재하지 않음 |
| 루트 노드 | 루트 노드 존재하지 않음 | 루트 노드 존재함 |
| 부모/자식 관계 | 부모 자식 개념이 존재하지 않음 | 부모 자식 관계가 존재함 |