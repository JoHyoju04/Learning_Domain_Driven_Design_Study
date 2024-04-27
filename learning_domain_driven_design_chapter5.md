# 05_간단한 비즈니스 로직 구현

단순한 비즈니스 로직을 구현하기 위한 아래의 두가지 패턴이 있다. 

# 트랜잭션 스크립트

이 패턴은 시스템 작업을 간단하고 쉬운 절차지향 스크립트로 구성한다. 이 절차는 작업에 트랜잭션을 적용해서 작업이 성공하거나 실패하도록 보장한다. 트랜잭션 스크립트 패턴은 단순한 비즈니스 로직을 가진 지원 하위 도메인에 적합하다.

## 구현

트랜잭션 스크립트의 예

```java
DB.StartTransaction();

var job = DB.LoadNextJob() ;
var json = LoadFile(job.Source);
var xml = ConvertJsonToXml(json) ;
WriteFile(job.Destination, xml.ToString());
DB.MarkJobAsCompleted(job);

DB.Commit();
```

## 그렇게 쉽진 않다

전체를 아우르는 트랜잭션 없이 여러 업데이트를 하는 경우에 네트워크 중단, 데이터베이스 시간 초과 또는 교착 상태, 프로세스를 실행하는 서버의 충돌로 문제가 발생하다면 시스템이 일관되지 않은 상태가 된다.

### 분산 트랜잭션

최신 분산 시스템에서 데이터베이스의 데이터를 변경한 다음 메시지 버스에 메시지를 발행하여 시스템의 다른 컴포넌트에 변경사항을 알리는 것이 일반적이다.

```java
public class LogVisit{
	...
	public void Execute(Guid userid, DataTime visitedOn){
	
		_db.Execute("UPDATE Users SET last_visit=@p1 WHERE user_id=@p2",
								visitedOn,userId);
		_messageBus.Publish( "VISITS TOPIC",
												new { UserId = userId, VisitDate = visitedOn });
	}

}
```

만약 메시지를 보내기 전에 오류가 난다면 User 테이블은 업데이트되지만 뒤에 로직은 실패하게 된다.

8장에서 CQRS 아키텍처 패턴이나 9장의 아웃박스 패턴에서 이를 다루는 방법이 소개된다.

### 암시적 분산 트랜잭션

```java
public class LogVisit{
	...
	public void Execute(Guid userid, DataTime visitedOn){
	
		_db.Execute("UPDATE Users SET visit=visit+1 WHERE user_id=@p1",userId);
	}

}
```

메서드가 수행하는 모든 작업은 하나의 데이터베이스에 있는 하나의 테이블에서 하나의 값을 업데이트 하는 것이다. 그러나 이것은 여전히 **잠재적으로 일관성 없는 상태로 이어질 수 있는 분산 트랜잭션**이다.

메서드가 성공했지만 호출자(클라이언트)에게 결과를 전달하는 데 아래와 같은 경우로 인해 실패할 수 있다.

- Logvisit이 REST 서비스의 일부이고 네트워크 중단이 발생한 경우
- Logvisit이 호출자가 동일한 프로세스에서 실행되고 있지만 호출자가 Logvisit 작업의 성공적인 실행을 추적하기 전에 프로세스가 실패하는 경우

트랜잭션 동작을 보장하는 한 가지 방법은 작업을 멱등성(idempotent)으로 만드는 것이다.

```java
public class LogVisit{
	...
	public void Execute(Guid userid, long expectedVisits){
	
		_db.Execute("UPDATE Users SET visit=@p1 WHERE user_id=@p2",expectedVisits,userId);
	}

}
```

예를 들어, 사용자에게 카운터 값을 전달하도록 요청할 수 있다. 카운터 값을 제공하기 위해 호출자는 먼저 현재 값을 읽고 로컬에서 증가시킨 다음 업데이트된 값을 **매개변수로 제공**해야한다.

```java
public class LogVisit{
	...
	public void Execute(Guid userid, long visits){
	
		_db.Execute("UPDATE Users SET visit=visit+1 WHERE user_id=@p1 and visit=@p2",userId, visits);
	}

}
```

또 다른 방법은 **낙관적 동시성 제어**를 사용하는 것이다. Logvisit 작업을 호출하기 전에 호출자는 카운터의 현재 값을 읽고 매개변수로 Logvisit에 전달했다. Logvisit은 호출자가 처음 읽은 값과 동일한 경우에만 카운터 값을 업데이트 한다.

## 트랜잭션 스크립트를 사용하는 경우

비즈니스 로직이 단순한 절차적 작업처럼 매우 간단한 문제 도메인에 효과적이다. 정의상 비즈니스 로직이 단순한 지원 하위 도메인에 적합하다. 또한 일반 하위 도메인과 같은 외부 시스템과 연동하기 위한 어댑터로 사용하거나 충돌 방지 계층(9장에서 나오는 개념)의 일부로 사용할 수 있다.

# 액티브 레코드

비즈니스 로직이 단순하지만 복잡한 자료구조에서 작동하는 경우 해당 자료구조를 액티브 레코드로 구현할 수 있다. 액티브 레코드 객체는 간단한 CRUD 데이터 접근 방법을 제공하는 자료구조이다.

트랜잭션 스크립트와의 차이점은 액티브 **레코드의 경우 데이터베이스에 직접 접근**하는 대신 트랜잭션 스크립트가 액티브 레코드 객체를 조작한다는 것이다.

이 패턴의 목적은 메모리 상의 객체를 데이터베이스 스키마에 매핑하는 복잡성을 숨기는 것이다. 영속성을 담당하는 것 외에도 액티브 레코드 객체에는 비즈니스 로직이 포함될 수 있다. 예를 들어 필드에 할당된 새 값의 유효성을 검사하거나 객체의 데이터를 조작하는 비즈니스 관련 절차를 구현할 수 있다.

# 추가적으로 찾아본 자료 )

## ActiveRecord 패턴 vs Data Mapper 패턴

### ActiveRecord 패턴

모든 query 메소드들을 모델에 정의하고 객체의 저장, 제거 그리고 불러오는 기능들은 모델의 메소드를 통해 사용하는 패턴이다. 결과적으로 SQL을 직접 사용하지 않으면서 데이터를 조작할 수 있다.

```java
import {BaseEntity, Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
// 모든 active record엔티티들은 BaseEntity클래스를 확장해야한다.
export class User extends BaseEntity {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    isActive: boolean;

    static findByName(firstName: string, lastName: string) {
        return this.createQueryBuilder("user")
            .where("user.firstName = :firstName", { firstName })
            .andWhere("user.lastName = :lastName", { lastName })
            .getMany();
    }
}
```

### **Data Mapper 패턴**

Data Mapper 패턴에서는 모든 쿼리 메소드들을 별도의 클래스에 정의한다.

이때 생성된 별도의 클래스를 *repository*라고 부른다.

결국 데이터베이스를 접근하기 위해 모델이 아닌 repository를 통해 접근하는것을 Data mapper라고 한다.

```java
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    isActive: boolean;

}
```

결론 : 

active record는 MVC모델에서 Model에 해당하는 부분이며, 비즈니스 데이터를 **표현**하고, **로직**을 담는 역할로 SRP 위반과 객체지향보다는 data structure에 가깝다. 그렇기 때문에 빈약한 도메인 모델 안티패턴이라고도 한다.

그렇지만 비즈니스 로직이 단순할 때 액티브 레코드 패턴을 사용하는 데는 아무런 문제가 없다고 한다.

참고 자료 : https://typeorm.io/active-record-data-mapper#what-is-the-active-record-pattern