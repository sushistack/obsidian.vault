
> Distribute Document Store

가장 중요한 Idea는 **Inverted Index**(역색인) 이다.  Inverted Index 란 document 에 있는 단어들을 나열하고 각 단어가 나오는 document 를 식별 하는 방식을 말한다.

즉, 단어 검색 시  O(1)의 시간 복잡도를 갖는다.

| Type   | Data structure |
| ------ | -------------- |
| Text   | Inverted Index |
| Number | BKD Tree       |
| Geo    | BKD Tree       |

> 특징
> > 1. 실시간 분석
> > 2. 전문(full text) 검색 엔진
> > 3. RESTful API
> > 4. Multi tenancy: 분산 저장된 인덱스에 대해서 통합 검색이 가능 함.

![](Pasted%20image%2020240924160256.png)

### 도큐먼트
: 단일 데이터 단위
### 인덱스
: 도큐먼트를 모아놓은 집합  
+) 인덱싱 : 도큐먼트를 인덱스에 포함시키는 것
### 노드
: Elasticsearch 인스턴스
### 클러스터
: 노드들의 집합

