---
title:  "EntityManagerFactory 와 EntityManager"
excerpt: "EntityManager, EntityManagerFactory에 대한 동작"

categories:
  - JPA
tags:
  - [JPA, Persistence, EntityManager, EntityManagerFactory]
last_modified_at: 2021-11-15T13:55:00-05:00

toc: true
toc_sticky: true

share: true
comments: true
---


## EntityManagerFactory

EntityManager를 생성하는 Class다. Thread-safe 하며 생성 비용이 많이 들기 때문에, 보통 DB 하나당 하나의 EntityManagerFactory를 생성한다.

```java
EntityManagerFactory factory = Persistence.createEntityManagerFactory("jdbcName");
```

## EntityManager

Entity 들을 영속성 컨텍스트에 두어 관리하는 Class이다.

```java
Entitymanager entityManager = factory.createEntityManager();
```

## 영속성 컨텍스트(Persistence Context)

간략하게 말해 Entity 를 영구히 저장하는 환경이다. EntityManager가 닫히거나, 초기화 되기 전까지 영속된 Entity를 관리한다.

```java
entityManager.persist(entity);
```

persist 메서드를 통해 EntityManager에 entity를 영속 시키게 되면, 그때부터 EntityManager가 영속 시킨 Entity 를 관리한다.

### 1차 캐시

- Map<Key, Value>로 1차 캐시에 저장 된다.
    - Key: @Id로 선언한 필드 값(PK)
    - Value: 해당 Id에 해당하는 Entity
- entityManager.find()를 하면 1차 캐시를 먼저 조회한다.
- 해당 Entity가 1차 캐시에 존재 하다면 바로 반환한다. 없을 경우 DB에서 조회해서 1차 캐시에 저장한다.

### 동일성 보장

- 영속 Entity를 "==" 연산자로 비교 했을때 true임을 보장한다.

### 쓰기 지연

- 한번의 EntityTransaction을 Commit 할때 쓰기 지연 SQL 저장소의 SQL들을 한번에 DB에 보낸다.
- flush를 통해 쓰기 지연 저장소에 있는 SQL들을 DB로 날릴 수 있다.

### 변경 감지

```java
Post post = entityManager.find(Post.class, "post1");

post.setTitle("제목수정");
post.setContents("컨텐츠수정");

post.commit();
```

- 가져온 Entity의 정보와 캐시 되어 있는 Entity의 상태와 비교하여 다르다면 Update 쿼리 수행

## Entity Life Cycle

### 비 영속

Entity 객체를 생성 했지만, 영속 시키지 않은 상태.

```java
// Bulider 패턴 객체 생성 예제.
Post post = Post.builder().title("제목").contents("내용").build();
```

### 영속

EntityManager를 통해서 영속성 컨텍스트에 저장한 상태.

```java
entityManager.persist(post);
```

- transaction commit 시점에 영속성 컨텍스트에 저장된 정보들이 DB로 전송된다.

### 준영속

영속성 컨텍스트 1차 캐시에서 제거 되고 쓰기 지연 SQL 저장소에 있던 해당 Entity 관련 SQL이 모두 제거 된다.

```java
entityManager.detach(post);
entityManager.close();
entityManager.clear();
```

- detach: Entity를 준 영속 상태로 변경.
- clear: 영속성 컨텍스트에 담겨져 있는 모든 것을 초기화 한다.
- close: 영속성 컨텍스트 종료.

### 삭제

Entity 를 영속성 컨텍스트와 데이터베이스에서 삭제한다.

```java
entityManager.remove(post);
```