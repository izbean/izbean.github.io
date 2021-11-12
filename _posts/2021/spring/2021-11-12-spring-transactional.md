---
title:  "@Transactional?"
excerpt: "spring transaction annotation에 대한 동작, 속성"

categories:
  - Spring
tags:
  - [Transactional, Spring, Transaction, isolation level]
last_modified_at: 2021-11-12T13:05:00-05:00

toc: true
toc_sticky: true

share: true
comments: true
---

Spring에서 isolation level, 특정 Exception 발생시 Rollback, propagation, readonly 같은 Transaction 단위의 속성에 대해 설정 할 수 있는 annotation 이다.

## Transaction?

사전적 의미로 데이터베이스의 상태 변화를 의미하는 말이다, 

Transaction은 기본적으로 아래의 4가지 속성을 가지고 있다.

1. 원자성: 한 Transaction 내에 실행한 작업들은 하나의 단위로 처리한다. 
2. 일관성: 일관성 있는 데이터 베이스 상태를 유지한다.
3. 격리성: 동시에 실행되는 Transaction 이 서로에게 영향을 못 미치도록 격리 수준을 설정 해야 한다.
4. 영속성: Transaction이 정상 처리 되었다면 결과가 항상 저장 되어야 한다.

## Transaction Object

Spring Framework에서는 Transaction 객체를 통해 begin, commit을 자동 수행 해주고 unchecked Exception 발생 시 해당 Transaction의 Rollback을 지원해준다.

### Transaction 객체 속성

Spring Framework를 사용하면 Transaction 객체의 begin, commit을 자동 수행 해준다.

@Transactional annotation을 사용하여 Transaction 단위를 관리 할 수 있다. 해당 annotation에는 아래 다섯가지 속성을 설정 할 수 있다.

1. isolation
2. propagation
3. noRollbackFor
4. rollbackFor
5. readOnly

### 1. isolation

```java
@Transaction(isolation=Isolation.{속성})
public void transaction(TransactionRequestDto transaction) {
		// business logic.
}
```

- 속성
    - DEFAULT
        
        현재 Service DB Connection에 설정 되어 있는 DB Isolation level을 설정한다.
        
    - READ_UNCOMMITED
        - level 0
        - Commit 되지 않은 데이터에 대해 읽기 허용
        
        문제점: Dirty Read
        
    - READ_COMMITED
        - level 1
        - Commit 된 데이터에 대해 읽기 허용
        
        문제점: Unrepeatable Read
        
    - REPEATEABLE_READ
        - level 2
        - Transaction 내에서 다른 Transaction이 데이터를 변경(수정) 하더라도, Transaction 시작 시점의 데이터를 보여준다.
        
        문제점: Phantom Read
        
    - SERIALIZABLE
        - level 3
        - 선행 Transaction이 테이블을 읽은 경우 다른 Transaction이 데이터 변경, 삭제, 추가 작업을 하지 못하게 막는다.
- 문제점
    - Dirty Read
        
        Transaction 1이 수정 중인 데이터를 Transaction 2에서 접근 할 수 있다. 만약 Transaction 1에서 작업이 실패하여 Rollback 되었다면 Transaction 2의 데이터는 잘못된 데이터라고 볼 수 있다.
        
    - Unrepeatable Read
        
        Transaction 2가 반복 조회 중에 Transaction 1이 정보를 수정하고 Commit 한다면, Transaction 2가 조회 했을 때 수정 된 데이터가 조회 된다.
        
    - Phantom Read
        
        Transaction 2가 반복 조회 중에 Transaction 1이 정보를 수정하고 Commit 하여도 Transaction 2가 조회 되는 데이터는 Transaction 발생 시점 데이터와 동일하지만 실제 데이터와 다른 결과가 조회 될 수 있다.
        
- Transaction 격리 수준의 필요성
    
    레벨이 높을수록 무결성을 유지 할 수 있지만, 무조건 적 상위 레벨 사용 시 데이터 Lock 으로 인해 Transaction이 순차적으로 처리 되며 성능 저하를 일으킬 수 있다. 그렇다고 해서 무조건 적 하위 레벨 사용 시 잘못된 값을 조회하거나 처리 할 여지가 있다. 따라서 서비스 의도에 따라 효율적인 방법을 찾아 상황에 맞게 사용하자.
    

### 2. propagation

```java
@Transaction(propagation=Propagation.{속성})
public void transaction(TransactionRequestDto transaction) {
		// business logic.
}
```

- 속성
    - REQUIRED(Default)
        
        이미 진행 중인 Transaction이 있다면 해당 Transaction 속성을 따르고, 없다면 새로운 Transaction을 생성한다.
        
    - REQUIRES_NEW
        
        항상 새로운 Transaction 을 생성한다. 이미 진행 중인 Transaction이 있다면 잠깐 보류 한 뒤 생성 한 Transaction 을 먼저 진행한다.
        
    - SUPPORT
        
        이미 진행 중인 Transaction이 있다면 해당 Transaction 속성을 따르고, 없다면 새로운 Transaction을 생성하지 않는다.
        
    - NOT_SUPPORT
        
        이미 진행 중인 Transaction이 있다면 잠깐 보류 한 뒤, Transaction 없이 작업을 수행 한다.
        
    - MANDATORY
        
        이미 진행 중인 Transaction이 있어야 작업을 수행하고 없다면 Exception 을 발생 시킨다.
        
    - NEVER
        
        Transaction 이 진행 중이지 않을 때 작업을 수행한다. Transaction 이 있다면 Exception 을 발생 시킨다.
        
    - NESTED
        
        진행 중인 Transaction이 있다면 중첩되어 새로운 Transaction이 실행 되며, 존재하지 않으면 REQUIRED와 동일하게 수행 된다.
        

### 3. noRollbackFor

```java
@Transaction(noRollbackFor={class}.class)
public void transaction(TransactionRequestDto transaction) {
		// business logic.
}
```

noRollbackFor 속성에 설정 된 Exception 일 경우 Exception 발생 시에도 Rollback하지 않음.

### 4. rollbackFor

```java
@Transaction(rollbackFor={class}.class)
public void transaction(TransactionRequestDto transaction) {
		// business logic.
}
```

rollbackFor 속성에 설정 된 Exception 일 경우 Checked Exception 이어도 Rollback을 진행한다.

### 5. timeout

```java
@Transaction(timeout={second})
public void transaction(TransactionRequestDto transaction) {
		// business logic.
}
```

지정된 초 단위 내에 Transaction이 수행 되지 않을 경우 rollback 한다.

Default 값인 -1 일 경우 no timeout

### 6. readOnly

```java
@Transaction(readOnly={true, false})
public void transaction(TransactionRequestDto transaction) {
		// business logic.
}
```

값이 true일 때 INSERT, UPDATE, DELETE 요청일 경우 Exception을 발생 시킨다.

- 사용 이유
    - 해당 설정을 사용하면 데이터에 Lock을 적용 할 필요가 없고 접근 할 수 있는 데이터가 변경 되지 않기 때문에, 일관적인 데이터를 가져올 수 있다.
    - Transaction ID를 부여하지 않기 때문에 ID 설정에 대한 오버헤드가 발생 하지 않기 때문에 성능의 이점을 볼 수 있다.