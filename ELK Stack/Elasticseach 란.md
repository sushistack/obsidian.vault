 Distribute Document Store

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

## Compact and aligned text (CAT) APIs
### Verbose


```bash
$ GET _cat/master?v=true
```

### Help

```bash
$ GET _cat/master?help
```

### Headers

```bash
GET _cat/nodes?h=ip,port,heapPercent,name
```

### Numeric formats

```bash
GET _cat/indices?bytes=b&s=store.size:desc,index:asc&v=true
```

## 생성/읽기/수정/삭제 (CRUD)

### 인덱스 생성

```bash
PUT /study
```

### 인덱스 조회

```bash
GET /study
```

### 인덱스 alias

```bash
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "study",
        "alias": "alias_for_study"
      }
    }
  ]
}
```

### aliases 목록 조회

```bash
GET /_cat/aliases?v
```

```bash
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "log-*",
        "alias": "logs"
      }
    }
  ]
}
```

멀티테넌시 기능을 이용하기 위해 index 패턴을 지정하여 별칭을 줄 수 있다.

### 도큐먼트 생성

다이나믹 매핑을 이용하여 인덱스를 자동으로 생성할 경우 아래와 같은 포맷으로 바로 도큐먼트를 생성할 수 있다.

```
POST /{indexName}/\_doc/{id}
```

(id 생략시 random id 생성)

#### 생성 예시

```
POST /study/_doc/1'
{
    "name" : "ELK",
    "group" : "couponad",
    "number_participants" : 7
}
```


### 도큐먼트 조회

```bash
GET /{indexName}/_doc/{id} # 특정 Document 조회
GET /{indexName}/_search # 인덱스 전체 조회
```

### 도큐먼트 수정

```bash
PUT /{indexName}/_doc/{id} # PUT은 데이터 전체를 삭제하고 삽입하는 형태
POST /{indexName}/_update/{id} # POST(_update)는 특정 필드만 수정하는 형태
```

### 벌크 데이터

```
POST /_bulk
{"index": {"_index": "bulk_test", "_id": 1}}
{"name":"park", "age":30}
{"index": {"_index": "bulk_test", "_id": 2}}
{"name":"lee", "age":40}
{"index": {"_index": "bulk_test", "_id": 3}}
{"name":"kim", "age":50}
```

## 매핑

: 필드의 데이터 타입을 설정하는 것  
다이나믹 매핑 : elasticsearch가 자동으로 설정  
명시적 매핑 : 사용자가 직접 설정


|**JSON data type**|**Elasticsearch data type**|
|---|---|
|null|필드를 추가하지 않음|
|true or false|boolean|
|floating point number|float|
|integer|long|
|object|object|
|array|null이 아닌 배열의 첫번째 값에 따라 달라짐  <br>ex )  <br>{"field1": [1,2]} -> long  <br>{"field1": ["test1","test2"]} -> keyword|
|string|입력된 값에 따라 date, text/keyword|
### 특정 인덱스 매핑 확인

```bash
GET /{indexName}/_mapping
```

### 명시적 매핑

```bash
PUT /explicit_mapping
```

인덱스 매핑이 정해지면 새로운 필드 추가는 가능하지만, 이미 정의된 필드에 대한 수정과 삭제는 불가능  
+) 숫자 타입은 다이나믹 매핑시 가장 큰 공간을 차지하는 long으로 매핑되어 불필요한 메모리를 차지할 수 있다.

### 매핑타입

다이나믹 매핑시 각 필드에는 적절한 데이터 타입이 매핑된다.  
이 데이터 타입은 필드에 포함된 데이터의 종류와 해당 용도를 나타냅니다.  
예를 들어 문자열을 text 및 keyword 데이터 타입을 이용하여 인덱싱할 수 있습니다.

|데이터 형태|데이터 타입|값 예시|참고 url|
|---|---|---|---|
|텍스트|text, keyword||[https://www.elastic.co/guide/en/elasticsearch/reference/7.10/text.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/text.html)  <br>[https://www.elastic.co/guide/en/elasticsearch/reference/7.10/keyword.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/keyword.html)|
|날짜|date|"2015-01-01" or "2015/01/01 12:10:30"|[https://www.elastic.co/guide/en/elasticsearch/reference/7.10/date.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/date.html)|
|숫자형|long, integer, short, byte, double, ...|100|[https://www.elastic.co/guide/en/elasticsearch/reference/7.10/number.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/number.html)|
|불린|boolean|true|


text : 텍스트 분석기가 텍스트를 작은 단위로 분리하여, 텍스트 내의 단어로 검색이 가능하다.  
-> 집계나 정렬을 지원하지 않는다.  
(text type으로 정렬할 경우 문장의 첫 문자열이 아닌, 분해된 용어를 기준으로 정렬을 수행하기 때문)


keyword : 원문 통째로 검색이 가능하다. 
-> 분석기를 거치지 않고 문자열 전체가 하나의 용어로 인덱싱 된다.  (완전 일치 검색을 사용할 수 있고, 집계나 정렬을 사용할 수 있다.) 

+) 성별(male, female)과 상태(active/deactive)를 나타내는 경우 용어를 분리할 필요가 없다.
이와 같은 범주형 데이터의 경우 데이터 형태가 고정되므로 문자열 집계나 정렬 작업에 활용 가치가 높다.

### 멀티 필드

```
PUT /multiple_field_mapping

{
  "mappings": {
    "properties": {
      "message": {
        "type": "text"
      },
      "contents": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```


## 템플릿

### 생성

```
PUT _index_template/test_template
{
  "index_patterns": ["test_*"],
  "priority": 1,
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "name": {"type": "text"},
        "age": {"type": "short"},
        "gender": {"type": "keyword"}
      }
    }
  }
}
```

priority: 우선순위가 같은 경우가 있을까??  
그런경우는 없다. 다른 템플릿과 index_pattern이 조금이라도 일치한다면 우선순위가 같을 수 없다.  
priority가 없으면 우선순위 0


### 삭제

```
DELETE _index_template/test_template
```

### 인덱스에 적용된 템플릿 확인

```
POST /_index_template/_simulate_index/ilm-history-34
```

### 동적 템플릿을 갖는 인덱스

```
PUT dynamic_index1
{
  "mappings": {
    "dynamic_templates": [
      {
        "my_string_fields": {
          "match_mapping_type": "string",
          "mapping": {"type": "keyword"}
        }
      }
    ]
  }
}
```

참고: [https://www.elastic.co/guide/en/elasticsearch/reference/7.10/dynamic-templates.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/dynamic-templates.html)


## 분석기

분석기: [https://www.elastic.co/guide/en/elasticsearch/reference/7.10/indices-analyze.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/indices-analyze.html)  
캐릭터필터: [https://www.elastic.co/guide/en/elasticsearch/reference/7.10/analysis-charfilters.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/analysis-charfilters.html)
토크나이저: [https://www.elastic.co/guide/en/elasticsearch/reference/7.10/analysis-tokenizers.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/analysis-tokenizers.html)
토큰필터: [https://www.elastic.co/guide/en/elasticsearch/reference/7.10/.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/indices-analyze.html)



### 분석기 테스트 api

```
POST _analyze
{
  "analyzer": "stop",
  "text": "The 10 most loving dog breeds"
}
```

```
POST _analyze
{
  "tokenizer": "standard",
  "filter": ["uppercase"],
  "text": "The 10 most loving dog breeds"
}
```

### 커스텀 분석기

```
PUT customizer_analyzer
{
  "settings": {
    "analysis": {
      "filter": {
        "my_stopwords": {
          "type": "stop",
          "stopwords": ["lions"]
        }
      },
      "analyzer": {
        "my_analyzer" : {
          "type": "custom",
          "char_filter": [],
          "tokenizer": "standard",
          "filter": ["lowercase", "my_stopwords"]
        }
      }
    }
  }
}
```
