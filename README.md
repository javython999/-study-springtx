# Spring Transaction 이해
***
### 스프링 트랜잭션 추상화
각각의 데이터 접근 기술들은 트랜잭션을 처리하는 방식에 차이가 있다.
예를 들어 JDBC 기술과 JPA 기술은 트랜잭션을 사용하는 코드 자체가 다르다.
따라서 데이터 접근 기술을 변경하게 되면 트랜잭션을 사용하는 코드도 모두 함께 변경해야 한다.

스프링은 이러한 문제를 해결하기 위해 트랜잭션 추상화를 제공한다.
트랜잭션을 사용하는 입장에서는 스프링 트랜잭션 추상화를 통해 코드 변경 없이 여러 방식을 사용할 수 있게 되는 것이다.
***
### 스프링 트랜잭션 사용 방식
* 선언적 트랜잭션 관리
  * ```@Transactional``` 애노테이션 하나만 선언해서 매우 편리하게 트랜잭션을 적용하는 것을 선언적 트랜잭션 관리라 한다.
  * 선언적 트랜잭션 관리는 과거 XML에 설정하기도 했다.
  * 이름 그대로 해당 로직에 트랜잭션을 적용하겠다 라고 어딘가에 선언하기만 하면 트랜잭션이 적용되는 방식이다.
* 프로그래밍 방식 트랜잭션 관리
  * 트랜잭션 매니저 또는 트랜잭션 템플릿 등을 사용해서 트랜잭션 관련 코드를 직접 작성하는 것을 프로그래밍 방식의 트랜잭션 관리라고 한다.

프래그래밍 방식의 트랜잭션 관리를 사용하게 되면 애플리케이션 코드가 트랜잭션이라는 기술 코드와 강하게 결합하게 된다.
선언적 트랜잭션 관리가 프로그래밍 방식에 비해 훨씬 간편하고 실용적이기 때문에 실무에서는 대부분 선언적 트랜잭션 관리를 사용한다.
***
### 선언적 트랜잭션과 AOP
```@Transactional```을 통한 선언적 트랜잭션 관리 방식을 사용하게 되면 기본적으로 프록시 방식의 AOP가 적용된다.


> 프록시 도입 전
> 
> 트랜잭션을 처리하기 위한 프록시를 도입하기 전에는 서비스 로직에서 트랜잭션을 직접 시작했다.

> 프록시 도입 후
>
> 트랜잭션을 처리하기 위한 프록시를 도입하면 트랜잭션을 처리하는 객체와 비즈니스 로직을 처리하는 서비스 객체를 명확하게 분리할 수 있다.
***
### 트랜잭션 적용 확인
* ```@Transactional```을 통해 선언적 트랜잭션 방식을 적용한다.
* ```@Transactional``` 애노테이션이 적용된 클래스의 ```.getClass()```를 호출하면 클래스 이름에 $$EnhancerBySpringCGLIB이 붙어 있는 것을 확인할 수 있다.
* ```AopUtils.isAopProxy()```를 통해 AOP 프록시가 적용되었는지 확인할 수 있다.
* ```TransactionSynchronizationManager.isActualTransactionActive()```를 사용하면 현재 트랜잭션이 적용되고 있는지 Boolean값이 return된다.
***
### 트랜잭션 적용 위치
스프링에서 우선순위는 항상 **더 구체적이고 자세한 것이 높은 우선순위를 가진다.**
예를 들어 메서드와 클래스에 애노테이션을 붙일 수 있다면 더 구체적인 메서드가 더 높은 우선순위를 가진다.
인터페이스와 해당 인터페이스를 구현한 클래스에 애노테이션을붙일 수 있다면 더 구체적인 클래스가 더 높은 우선순위를 가진다.


스프링의 ```@Transactional```은 두 가지 규칙이 있다.
1. 우선순위 규칙
   * 클래스에 ```@Transactional(readOnly = true)```, 메서드에 ```@Transactional(readOnly = false)``` 적용시
   더 구체적인 메서드의 ```@Transactional(readOnly = false)```가 적용된다.
2. 클래스에 적용하면 메서드는 자동 적용
   * 클래스에 ```@Transcational(readOnly = true)``` 적용시 클래스에 ```@Transational``` 애노테이션이 없어도 자동으로 적용된다.
***
### 트랜잭션 AOP 주의사항 - 프록시 내부 호출
```@Transactional```을 사용하면 스프링 트랜잭션 AOP가 적용된다.
트랜잭션 AOP는 기본적으로 프록시 방식의 AOP를 사용한다.
프록시 객체가 요청을 먼저 받아서 트랜잭션을 처리하고, 실제 객체를 호출해준다.

아래와 같이 ```@Transactional```애노테이션이 없는 메서드와  ```@Transactional```애노테이션이 있는 메서드를 가지고 있는 클래스가 있을 때
```java
class CallService {
    public void external() {
        log.info("call external");
        internal();
    }

    @Transactional
    public void internal() {
        log.info("call internal");
    }
}
```

```internal()```을 호출하게 되면 ```@Transactional```애노테이션이 있기 때문에 ```CallService```의 프록시 객체가 요청을 받아서 처리하게 된다.
```external()```을 호출하게 되면 ```@Transactional```애노테이션이 없기 때문에 프록시 객체가 아닌 실제 ```CallService```객체가 요청을 처리하게 되고
실제```CallService```의 ```internal()```가 호출이 되기 때문에 트랜잭션이 적용이 되지 않는다.
 
이 문제를 해결하기 위한 방법으로 트랜잭션을 처리하는 별도의 클래스로 분리하는 방법이 있다.
```java
class CallService {     
    private final InternalService internalService;
    
    public void external() {
        log.info("call external");
        internalService.internal();
    }
}

class InternalService { 
    @Transactional
    public void internal() {
        log.info("call internal");
    }
}
```
```external()```이 호출되면 실제 ```CallService``` 객체가 요청을 처리하게 된다.
요청 처리중 ```internalService.internal()```을 호출하게 되면 ```internalService```의 프록시 객체가 요청을 처리하게 되고
트랜잭션이 처리된다.

#### public 메서드만 트랜잭션 적용
스프링의 트랜잭션 AOP 기능은 ```public``` 메서드에만 트랜잭션을 적용하도록 기본 설정이 되어있다.
그래서 ```protected``` , ```private``` , ```package-visible``` 에는 트랜잭션이 적용되지 않는다. 생각해보면
```protected``` , ```package-visible``` 도 외부에서 호출이 가능하다. 따라서 이 부분은 앞서 설명한 프록시의
내부 호출과는 무관하고, 스프링이 막아둔 것이다.


스프링이 ```public```에만 트랜잭션을 적용하는 이유는 다음과 같다.
```java
@Transactional
public class Hello { 
    public method1();
    method2();
    protected method3();
    private method4();
}
```
* 이렇게 클래스 레벨에 트랜잭션을 적용하면 모든 메서드에 트랜잭션이 걸릴 수 있다. 그러면 트랜잭션을
  의도하지 않는 곳 까지 트랜잭션이 과도하게 적용된다. 트랜잭션은 주로 비즈니스 로직의 시작점에 걸기
  때문에 대부분 외부에 열어준 곳을 시작점으로 사용한다. 이런 이유로 ```public``` 메서드에만 트랜잭션을
  적용하도록 설정되어 있다.
* ```public``` 이 아닌곳에 ```@Transactional``` 이 붙어 있으면 예외가 발생하지는 않고, 트랜잭션 적용만
  무시된다
***
### 트랜잭션 AOP 주의사항 - 초기화 시점
스프링 초기화 시점에는 트랜잭션 AOP가 적용되지 않을 수 있다.
```@PostConstruct```와 ```@Transactional```을 함께 사용하면 트랜잭션이 적용되지 않는다.
초기화 코드가 먼저 호출되고, 그 다음에 트랜잭션 AOP가 적용되기 때문이다. 따라서 초기화 시점에는 해당 메서드에서 트랜잭션을 획득 할 수 없다.

가장 확실한 대안은 ```ApplicationReadyEvent```을 사용하는 것이다.
```java
@EventListener(value = ApplicationReadyEvent.class)
@Transactional
public void init2() {  
    log.info("Hello init ApplicationReadyEvent");
}
```
이 이벤트는 트랜잭션 AOP를 포함한 스프링 컨테이너가 완전히 생성되고 난 다음에 이벤트가 붙은 메서드를 호출해준다.
***
### 트랜잭션 옵션

#### value, transactionManager
트랜잭션을 사용하려면 먼저 스프링 빈에 등록된 어떤 트랜잭션 매니저를 사용할지 알아야 한다.
생각해보면 코드로 직접 트랜잭션을 사용할 때 분명 트랜잭션 매니저를 주입 받아서 사용했다.
```@Transactional``` 에서도 트랜잭션 프록시가 사용할 트랜잭션 매니저를 지정해주어야 한다.
사용할 트랜잭션 매니저를 지정할 때는 ```value```, ```transactionManager``` 둘 중 하나에 트랜잭션 매니저의
스프링 빈의 이름을 적어주면 된다.
이 값을 생략하면 기본으로 등록된 트랜잭션 매니저를 사용하기 때문에 대부분 생략한다. 그런데 사용하는
트랜잭션 매니저가 둘 이상이라면 다음과 같이 트랜잭션 매니저의 이름을 지정해서 구분하면 된다.
```java
public class TxService {
    @Transactional("memberTxManager")
    public void member() {...}
    
    @Transactional("orderTxManager") 
    public void order() {...}
}
```

#### rollbackFor
예외 발생시 스프링 트랜잭션의 기본 정책은 다음과 같다.
* 언체크 예외인 RuntimeException , Error 와 그 하위 예외가 발생하면 롤백한다.
* 체크 예외인 Exception 과 그 하위 예외들은 커밋한다.

``rollbackFor`` 옵션을 사용하면 기본 정책에 추가로 어떤 예외가 발생할 때 롤백할 지 지정할 수 있다.
```java
@Transactional(rollbackFor = Exception.class)
// 예를 들어서 이렇게 지정하면 체크 예외인 Exception 이 발생해도 롤백하게 된다. 
// (하위 예외들도 대상에포함된다.)
```

#### noRollbackFor
앞서 설명한 rollbackFor 와 반대이다. 기본 정책에 추가로 어떤 예외가 발생했을 때 롤백하면 안되는지
지정할 수 있다.

#### propagation
(추후 수정)

#### isolation
트랜잭션 격리 수준을 지정할 수 있다. 기본 값은 데이터베이스에서 설정한 트랜잭션 격리 수준을 사용하는
``DEFAULT`` 이다. 대부분 데이터베이스에서 설정한 기준을 따른다. 애플리케이션 개발자가 트랜잭션 격리
수준을 직접 지정하는 경우는 드물다

#### timeout
트랜잭션 수행 시간에 대한 타임아웃을 초 단위로 지정한다. 기본 값은 트랜잭션 시스템의 타임아웃을
사용한다. 운영 환경에 따라 동작하는 경우도 있고 그렇지 않은 경우도 있기 때문에 꼭 확인하고 사용해야
한다.
``timeoutString``도 있는데, 숫자 대신 문자 값으로 지정할 수 있다.

#### label
트랜잭션 애노테이션에 있는 값을 직접 읽어서 어떤 동작을 하고 싶을 때 사용할 수 있다. 일반적으로
사용하지 않는다.

#### readOnly
트랜잭션은 기본적으로 읽기 쓰기가 모두 가능한 트랜잭션이 생성된다.
``readOnly=true`` 옵션을 사용하면 읽기 전용 트랜잭션이 생성된다. 이 경우 등록, 수정, 삭제가 안되고 읽기
기능만 작동한다. (드라이버나 데이터베이스에 따라 정상 동작하지 않는 경우도 있다.) 그리고 readOnly
옵션을 사용하면 읽기에서 다양한 성능 최적화가 발생할 수 있다.

``readOnly`` 옵션은 크게 3곳에서 적용된다.
* 프레임워크
  * JdbcTemplate은 읽기 전용 트랜잭션 안에서 변경 기능을 실행하면 예외를 던진다.
  * JPA(하이버네이트)는 읽기 전용 트랜잭션의 경우 커밋 시점에 플러시를 호출하지 않는다. 읽기
    전용이니 변경에 사용되는 플러시를 호출할 필요가 없다. 추가로 변경이 필요 없으니 변경 감지를 위한
    스냅샷 객체도 생성하지 않는다. 이렇게 JPA에서는 다양한 최적화가 발생한다.
* JDBC 드라이버
  * 참고로 여기서 설명하는 내용들은 DB와 드라이버 버전에 따라서 다르게 동작하기 때문에 사전에
  확인이 필요하다.
  * 읽기 전용 트랜잭션에서 변경 쿼리가 발생하면 예외를 던진다.
  * 읽기, 쓰기(마스터, 슬레이브) 데이터베이스를 구분해서 요청한다. 읽기 전용 트랜잭션의 경우 읽기
    (슬레이브) 데이터베이스의 커넥션을 획득해서 사용한다.
* 데이터베이스
  * 데이터베이스에 따라 읽기 전용 트랜잭션의 경우 읽기만 하면 되므로, 내부에서 성능 최적화가
    발생한다.
***
### 예외와 트랜잭션 커밋, 롤백 - 기본
예외 발생시 스프링 트랜잭션 AOP는 예외의 종류에 따라 트랜잭션을 커밋하거나 롤백한다.
* 언체크 예외인 ``RuntimeException``, ``Error``와 그 하위 예외가 발생하면 트랜잭션을 롤백한다.
* 체크 예외인 ``Exception``과 그 하위 예외가 발생하면 트랜잭션을 커밋한다.
* 기본 정책과 무관하게 특정 예외를 강제로 롤백하고 싶으면 ``rollbackFor``를 사용하면 된다.

### 예외와 트랜잭션 커밋, 롤백 - 활용
스프링은 기본적으로 체크 예외는 비즈니스 의미가 있을 때 사용하고, 런타임(언체크) 예외는 복구가 불가능한 예외로 가정한다.
* 체크 예외: 비즈니스 의미가 있을 때 사용
* 언체크 예외: 복구 불가능한 예외

> 비즈니스 예외
> 
> 예제로 비즈니스 예제를 알아보자. 주문을 하는데 상황에 따라 다음과 같이 조치한다.
> * 정상: 주문시 결제를 성공하면 주문 데이터를 저장하고 결제 상태를 ``완료``로 처리한다.
> * 시스템 예외: 주문시 내부에 복구 불가능한 예외가 발생하면 주문 데이터를 롤백한다.
> * 비즈니스 예외: 주문 결제시 잔고가 부족하면 주문 데이터를 저장하고 결제 상태를 ``대기``로 처리한다.
>   * 이 경우 고객에게 잔고 부족을 알리고 별도의 계좌로 입금하도록 안내한다.
***
### 스프링 트랜잭션 전파 - 전파 기본
트랜잭션을 각각 사용하는 것이 아니라, 트랜잭션이 이미 진행중인데, 여기에 추가로 트랜잭션을 수행하면
어떻게 될까? 기존 트랜잭션과 별도의 트랜잭션을 진행할까? 아니면 기존 트랜잭션을 그대로 이어 받아서 트랜잭션을 수행해야 할까?
이런 경우 어떻게 동작할지 결정하는 것을 트랜잭션 전파(propagation)라 한다.

> 외부 트랜잭션이 수행중인데, 내부 트랜잭션이 추가로 수행됨
>
> * 외부 트랜잭션이 수행중이고, 아직 끝나지 않았는데, 내부 트랜잭션이 수행된다.
> * 스프링은 이 경우 외부 트랜잭션과 내부 트랜잭션을 묶어서 하나의 트랜잭션으로 만들어준다. 내부 트랜잭션이 외부 트랜잭션에 참여하는 것이다.

외부 트랜잭션 또는 내부 트랜잭션은 하나의 논리 트랜잭션이다. 이 논리 트랜잭션을 묶어 하나의 물리 트랜잭션으로 만든다.
물리 트랜잭션은 아래의 원칙을 따른다.

* 모든 논리 트랜잭션이 커밋되어야 물리 트랜잭션이 커밋된다.
* 하나의 논리 트랜젹선이라도 롤백되면 물리 트랜잭션은 롤백된다.

#### 외부 트랜잭션 커밋 && 내부 트랜잭션 커밋
```java
void innerCommit() {
    log.info("외부 트랜잭션 시작");
    TransactionStatus outer = txManager.getTransaction(new DefaultTransactionAttribute());
    log.info("outer.isNewTransaction={}", outer.isNewTransaction()); // true

    log.info("내부 트랜잭션 시작");
    TransactionStatus inner = txManager.getTransaction(new DefaultTransactionAttribute());
    log.info("inner.isNewTransaction={}", inner.isNewTransaction()); // false
    log.info("내부 트랜잭션 커밋");
    txManager.commit(inner);

    log.info("외부 트랜잭션 커밋");
    txManager.commit(outer);
}
```
1. 외부 트랜잭션 시작
2. 내부 트랜잭션 시작
3. 내부 트랜잭션 커밋
4. 외부 트랜잭션 커밋

3의 과정에서 트랜잭션 매니저는 트랜잭션의 ``isNewTransaction()``의 여부에 따라 다르게 동작한다.
``true``인 경우에만 ``commit``을 호출하고 ``false``인 경우 ``commit``을 호출해도 실제 커밋을 호출하지 않는다.
``commit``이나 ``rollback``을 호출하면 트랜잭션이 종료되는데 위의 코드처럼 되어있는 경우 내부 트랜잭션에서 커밋이 호출되어버리면
외부 트랜잭션을 처리하지 못하고 트랜잭션이 종료되어버리는 상황이 발생하기 때문이다.

#### 외부 트랜잭션 커밋 && 내부 트랜잭션 롤백
```java
void innerRollback() {
    log.info("외부 트랜잭션 시작");
    TransactionStatus outer = txManager.getTransaction(new DefaultTransactionAttribute());

    log.info("내부 트랜잭션 시작");
    TransactionStatus inner = txManager.getTransaction(new DefaultTransactionAttribute());
    log.info("내부 트랜잭션 롤백");
    txManager.rollback(inner);

    log.info("외부 트랜잭션 커밋");
    assertThatThrownBy(()->txManager.commit(outer)).isInstanceOf(UnexpectedRollbackException.class);
}
```
1. 외부 트랜잭션 시작
2. 내부 트랜잭션 시작
3. 내부 트랜잭션 롤백
4. 외부 트랜잭션 커밋 -> ``UnexpectedRollbackException``오류 발생

4의 과정에서 커밋을 호출하면 ``UnexpectedRollbackException``오류가 발생되면서 ``rollback``된다.
이는 ``하나의 논리 트랜젹선이라도 롤백되면 물리 트랜잭션은 롤백된다``는 원칙을 따른 것이다.
내부 트랜잭션은 커밋 외부 트랜잭션은 롤백인 경우에도 마찬가지다. 내부 트랜잭션은 ``isNewTransaction()``가 ``false``이기 때문에
커밋, 롤백을 호출 할 수가 없다.(호출하면 트랜잭션이 종료되어버리기 때문에) 그래서 3의 과정에서 트랜잭션 동기화 매니저에
``rollbackOnly=true``라는 표시를 해둔다. 이 표시 때문에 4의 과정에서 커밋을 호출하지만 ``rollbackOnly``가 ``true``이기 때문에
``UnexpectedRollbackException``에러가 발생하게 된다.

#### REQUIRES_NEW
외부 트랜잭션과 내부 트랜잭션을 완전히 분리하면 별도의 물리 트랜잭션을 사용하게 된다.
내부 트랜잭션이 외부 트랜잭션에게 영향을 주지 않고, 외부 트랜잭션도 내부 트랜잭션에 영향을 주지 않게 된다.
커밋과 롤백도 각각 별도로 이루어지게 된다. 

```java
void innerRollbackRequiresNew() {
    log.info("외부 트랜잭션 시작");
    TransactionStatus outer = txManager.getTransaction(new DefaultTransactionAttribute());
    log.info("outer.isNewTransaction={}", outer.isNewTransaction());
  
    log.info("내부 트랜잭션 시작");
    DefaultTransactionAttribute definition = new DefaultTransactionAttribute();
    definition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
    TransactionStatus inner = txManager.getTransaction(definition);
    log.info("inner.isNewTransaction={}", inner.isNewTransaction());
  
    log.info("내부 트랜잭션 롤백");
    txManager.rollback(inner);
  
    log.info("외부 트랜잭션 커밋");
    txManager.commit(outer);
}
```
> ``DefaultTransactionAttribute definition = new DefaultTransactionAttribute();``
> ``definition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);``

위 코드로 인해 내부 트랜잭션은 외부 트랜잭션에 참여하는 것이 아니라 새로운 신규 트랜잭션이 생성된다.
``inner.isNewTransaction()``은 ``true``가 된다.

``txManager.rollback(inner);`` 호출시 내부 트랜잭션은 새로 생성된 트랜잭션이기 때문에
``rollbackOnly=true``라는 표시를 하지 않고 롤백이 호출된다.

***
### 스프링 트랜잭션 전파 - 다양한 전파 옵션
전파 옵션에 별도의 설정을 하지 않으면 ``REQUIRED``가 기본으로 사용된다.

> REQUIRED
> 
> 가장 많이 사용하는 기본 설정이다. 기존 트랜잭션이 없으면 생성하고, 있으면 참여한다.
> * 기존 트랜잭션 없음: 새로운 트랜잭션을 생성한다.
> * 기존 트랜잭션 있음: 기존 트랜잭션에 참여한다.

> REQUIRES_NEW
> 
> 항상 새로운 트랜잭션을 생성한다.
> * 기존 트랜잭션 없음: 새로운 트랜잭션을 생성한다.
> * 기존 트랜잭션 있음: 새로운 트랜잭션을 생성한다.


> SUPPORT
> 
> 트랜잭션을 지원한다는 뜻이다. 기존 트랜잭션이 없으면, 없는대로 진행하고, 있으면 참여한다.
>
> * 기존 트랜잭션 없음: 트랜잭션 없이 진행한다.
> * 기존 트랜잭션 있음: 기존 트랜잭션에 참여한다.

> NOT_SUPPORT
>
> 트랜잭션을 지원하지 않는다는 의미이다.
> * 기존 트랜잭션 없음: 트랜잭션 없이 진행한다.
> * 기존 트랜잭션 있음: 트랜잭션 없이 진행한다. (기존 트랜잭션은 보류한다)

> MANDATORY
> 
> 의무사항이다. 트랜잭션이 반드시 있어야 한다. 기존 트랜잭션이 없으면 예외가 발생한다.
> * 기존 트랜잭션 없음: ``IllegalTransactionStateException`` 예외 발생
> * 기존 트랜잭션 있음: 기존 트랜잭션에 참여한다.

> NEVER
> 
> 트랜잭션을 사용하지 않는다는 의미이다. 기존 트랜잭션이 있으면 예외가 발생한다. 기존 트랜잭션도
허용하지 않는 강한 부정의 의미로 이해하면 된다.
> * 기존 트랜잭션 없음: 트랜잭션 없이 진행한다.
> * 기존 트랜잭션 있음: ``IllegalTransactionStateException`` 예외 발생

> NESTED
> * 기존 트랜잭션 없음: 새로운 트랜잭션을 생성한다.
> * 기존 트랜잭션 있음: 중첩 트랜잭션을 만든다.
>   * 중첩 트랜잭션은 외부 트랜잭션의 영향을 받지만, 중첩 트랜잭션은 외부에 영향을 주지 않는다.
>   * 중첩 트랜잭션이 롤백 되어도 외부 트랜잭션은 커밋할 수 있다.
>   * 외부 트랜잭션이 롤백 되면 중첩 트랜잭션도 함께 롤백된다.
> 
> 참고
>  * JDBC savepoint 기능을 사용한다. DB 드라이버에서 해당 기능을 지원하는지 확인이 필요하다.
>  * 중첩 트랜잭션은 JPA에서는 사용할 수 없다.

트랜잭션 전파와 옵션
``isolation`` , ``timeout`` , ``readOnly``는 트랜잭션이 처음 시작될 때만 적용된다. 트랜잭션에 참여하는 경우에는 적용되지 않는다.
예를 들어서 ``REQUIRED`` 를 통한 트랜잭션 시작, ``REQUIRES_NEW`` 를 통한 트랜잭션 시작 시점에만 적용된다.
