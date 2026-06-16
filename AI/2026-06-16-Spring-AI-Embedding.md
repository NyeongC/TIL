# Spring AI - Embedding / PGVector 정리

## 1. 임베딩(Embedding)이란?

텍스트 또는 이미지를 AI가 이해하기 쉬운 숫자 벡터(Vector)로 변환하는
과정

------------------------------------------------------------------------

## 2. 예시

``` text
김치
↓
[0.123, 0.555, 0.888 ...]

깍두기
↓
[0.124, 0.552, 0.881 ...]

자동차
↓
[0.991, 0.111, 0.222 ...]
```

유사한 의미일수록 벡터 공간에서 가까워진다.

``` text
김치 ↔ 깍두기 (가까움)

김치 ↔ 자동차 (멀음)
```

------------------------------------------------------------------------

## 3. 주의사항

저장할 때 사용한 임베딩 모델과

검색할 때 사용하는 임베딩 모델은 동일해야 한다.

예)

``` text
저장
→ text-embedding-3-small

검색
→ text-embedding-3-small
```

가능

``` text
저장
→ 모델 A

검색
→ 모델 B
```

비추천

벡터 공간 자체가 달라져 검색 품질이 떨어질 수 있다.

------------------------------------------------------------------------

## 4. 벡터 저장소(Vector Store)

벡터를 저장하고 유사도 검색을 수행하는 저장소

대표 예시

``` text
PGVector
MariaDB Vector
Chroma
Milvus
Pinecone
```

------------------------------------------------------------------------

## 5. PGVector 설치

``` bash
docker run \
--name pgvector \
-d \
-p 5432:5432 \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD=postgres \
-v pgdata:/var/lib/postgresql/data \
pgvector/pgvector:pg17
```

구성

``` text
PostgreSQL
+
PGVector Extension
```

추가로

``` text
pgAdmin
```

설치하여 확인 가능

------------------------------------------------------------------------

## 6. PGVector 활용 흐름

``` text
데이터
↓
임베딩
↓
PGVector 저장
↓
질문
↓
임베딩
↓
유사도 검색
↓
결과 반환
```

------------------------------------------------------------------------

## 7. Spring AI 와 임베딩

Spring AI는 기본적으로

``` text
텍스트 임베딩
```

위주로 제공

예)

``` java
EmbeddingModel
VectorStore
QuestionAnswerAdvisor
```

------------------------------------------------------------------------

## 8. 이미지 임베딩

이번 실습에서는

``` text
ArcFace
```

모델 사용

얼굴 이미지를 벡터로 변환

``` text
얼굴 사진
↓
ArcFace
↓
512 차원 벡터
↓
PGVector 저장
```

------------------------------------------------------------------------

## 9. 실습 환경

도커 이미지 로드

``` text
c:\Users\cjfsu\Documents\book-spring-ai\docker\face-embed-api\image\face-embed-api.tar
```

모델

``` text
ArcFace
```

형태

``` text
REST API
```

호출 방식으로 사용

------------------------------------------------------------------------

## 10. 얼굴 저장 실습

``` text
이미지 업로드
↓
얼굴 임베딩 생성
↓
PGVector 저장
```

예시 코드

``` java
float[] vector = getFaceVector(mf);
```

------------------------------------------------------------------------

## 11. 얼굴 검색 실습

``` text
이미지 업로드
↓
얼굴 임베딩 생성
↓
PGVector 유사도 검색
↓
가장 가까운 얼굴 반환
```

예시 SQL

``` sql
SELECT content,
       (embedding <=> ?::vector) AS similarity
FROM face_vector_store
ORDER BY embedding <=> ?::vector
LIMIT 3;
```

------------------------------------------------------------------------

## 12. 핵심 정리

``` text
Embedding
= 데이터를 벡터로 변환

Vector Store
= 벡터 저장 및 검색

PGVector
= PostgreSQL 기반 벡터 저장소

ArcFace
= 얼굴 임베딩 모델

RAG
= 검색 결과를 LLM에 전달하여 답변 생성
```
