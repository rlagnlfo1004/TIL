# 0. Test Double

예를 들어서, 저희가 UserService만 테스트 하고 싶어도,  실제, UserService는 UserRespository, DB까지 연결이 되어 있기에, 해당 로직을 테스트하기 위해선 항상 데이터베이스의 영향을 받을 것이고, 이는 데이터베이스의 상태에 따라 다른 결과를 유발할 수도 있습니다. 

이렇게 테스트하려는 객체와 연관된 객체를 사용하기가 어렵고 모호할 때 대신해 줄 수 있는 객체를 **테스트 더블**이라 합니다. 테스트 더블은 실제 객체가 아닌 객체로 코드를 그 외의 모든 코드에서 떼어 놓을 수 있습니다. 즉, 단위 테스트에서의 모듈 의존성 격리를 실현할 수 있습니다.
<br><br>
### 테스트 더블의 종류

> ***Reference***
> 
> 
> https://codinghack.tistory.com/92
> 
> https://tecoble.techcourse.co.kr/post/2020-09-19-what-is-test-double/
> 
- **Dummy:** 그냥 자리를 채우는, 아무 기능 없는 객체, 호출하기만 하면 OK!!
- **Fake:** 실제처럼 보이게 하는 간단한 구현체, 동작은 구현되어있다.(예: DB 대신 `HashMap`).
- **Stub:** 미리 정해진 값을 반환하도록 설정된 객체 (상태 검증).
- **Spy:** 스텁 기능에 더해, 메서드 호출 횟수 등 정보를 기록하는 객체.
- **Mock:** 특정 행위가 발생했는지 검증하는 객체.

이전에는 테스트 코드를 위한 테스트 더블을 직접 만들어야 했지만, 지금은 테스트 프레임워크들이 이를 편하게 사용할 수 있도록 제공해줍니다. 이러한 프레임워크들이 제공해주는 편리한 기능을 효율적이고 올바르게 사용하기 위해, 테스트 프레임워크가 제공해주는 테스트 더블로는 어떤 것이 있는지를 학습을 해보았습니다.

---
<br><br>

# 1. 무엇을 테스트?

> ***Reference***
> 
> 
> https://kakaoentertainment-tech.tistory.com/78
> 

우리가 구현하는 대부분의 애플리케이션의 내부에는 많은 객체들이 다른 객체들과 상호 협력하며 작업을 처리합니다. 이때, Production code를 사용하는 여러 클라이언트 코드 입장에서 협력 대상 객체(코드)는 ‘어떻게 처리되는지’ 보단 ‘무엇을 처리하는가’가 중요합니다. 결국 다시 말해, 저희가 관심있게 보는 것은 “어떻게”가 아닌 **“무엇”**이며, 이 무엇을 우리는 **“interface”**라 부릅니다.
<br><br>

### SUT의 "명세"를 테스트!

- **“SUT(System Under Test)”**는 테스트 하고자 하는 대상(클래스, 메서드 등)
- **“명세”**는 SUT가 어떻게 동작해야 하는지에 대한 규칙이나 기대를 의미합니다. 예를 들어, "입력값이 `null`일 경우 에러를 발생시켜야 한다" 또는 "정상적인 입력이 들어오면 특정 목록에 추가되어야 한다"와 같은 것들입니다.
- 우리가 작성해야 할 테스트 케이스의 수는 이 명세에 따라 발생하는 **"경우의 수(case)"**와 같습니다.
<br><br>

### Example 1

```java
public void removeGroup(SettlementGroupMember group) {
    AssertUtils.notNull(group, "group must not be null");
    this.groups.remove(group);
}
```

위 메서드를 호출하는 Client 코드에서 겪을 수 있는 경우의 수는 2가지 입니다.

1. `group`이 `null`이면, `AssertUtils.notNull`에 의해 `IllegalArgumentException` 에러가 발생해야 합니다.
2. `group`이 `null`이 아니면, `this.groups` 목록에서 해당 `group` 객체가 삭제되어야 합니다.

따라서 이 명세에 따른 경우의 수는 **2가지**가 됩니다. 이 두 가지 경우가 바로 우리가 작성해야 할 테스트 케이스가 됩니다.

즉, **SUT의 구현된 명세(디자인)을 기반으로 경우의 수**를 정의하면, 이 **경우의 수가 Test Case**이고, 우리는 이 Case에 대한 테스트 코드를 구현해 주면됩니다.
<br><br>

### Example 2

예시를 한 개 더 보겠습니다. 만약에 "자판기"를 테스트한다고 가정해보면, 자판기의 "명세(규칙)"는 다음과 같습니다.

1. **동전이 1000원 이상**일 때: **커피를 내어준다.**
2. **동전이 1000원 미만**일 때: **"금액이 부족합니다"**라는 메시지를 띄운다.
3. **동전을 넣지 않았을 때** (0원): **"동전을 넣어주세요"**라는 메시지를 띄운다.

이 명세에 따른 **"경우의 수(case)"**는 몇 가지일까요? → **3가지**입니다.

따라서 우리는 이 **3가지 경우에 대해 각각 테스트 코드를 작성**해야 합니다.

- **테스트 케이스 1:** 1000원을 넣고 커피가 나오는지 확인하는 테스트 코드
- **테스트 케이스 2:** 500원을 넣고 "금액이 부족합니다" 메시지가 뜨는지 확인하는 테스트 코드
- **테스트 케이스 3:** 0원을 넣고 "동전을 넣어주세요" 메시지가 뜨는지 확인하는 테스트 코드

이처럼, **테스트 코드는 SUT가 제공하는 기능의 "명세"를 파악하고, 그 명세에 따라 발생할 수 있는 모든 경우의 수를 하나씩 검증하는 방식으로 작성**하면 됩니다.

---
<br><br>

# 2. How?

> ***Reference***
> 
> 
> https://kakaoentertainment-tech.tistory.com/78
> 
<br><br>

# Step 1. 코드 품질 올리기


우선 특정 테스트 기법과 tool 보단 테스트 하기 쉬운, 즉 Testable 한 (테스트 용이성) Production 코드를 구현하는지에 대해 이야기 하겠습니다. 핵심은 **"테스트하기 쉬운 코드"와 "테스트하기 어려운 코드"를 분리해야 한다**는 것입니다.
<br><br>

### **(1) 테스트하기 쉬운 코드:**

- **결정적(Deterministic) 코드:** 같은 입력에 대해 항상 같은 결과를 반환하는 코드입니다. `plus(10, 20)`은 항상 `30`을 반환하므로 테스트하기 쉽습니다.
- **부수 효과(Side-effects)가 없는 코드:** 외부 상태(예: 데이터베이스, 파일, 시스템 콘솔)를 변경하지 않는 코드입니다. `System.out.println()`처럼 외부 상태를 건드리는 코드는 테스트하기 어렵습니다.

```java
// sut
public int plus(int x, int y) {
    int result = x + y;
    return result;
}

@Test
@DisplayName("10 더하기 20 은 30 이다")
void sutPlus10And20Then30() {
    var sut = new Testable();
    var actual = sut.plus(10, 20);
    assertThat(actual).isEqualTo(30);
}
```
<br><br>

### **(2) 테스트하기 어려운 코드:**

- **비결정적(Non-deterministic) 코드:** 실행 시점마다 결과가 달라지는 코드입니다. `LocalDateTime.now()`처럼 현재 시간을 가져오는 코드는 매번 다른 결과를 반환하므로 테스트하기 어렵습니다.

```java
public String getTimeOfDay() {
    LocalDateTime now = LocalDateTime.now();
    if (now.getHour() >= 0 && now.getHour() < 6) {
        return "Night";
    }
    
    if (now.getHour() >= 6 && now.getHour() < 12) {
        return "Morning";
    }
    
    if (now.getHour() >= 12 && now.getHour() < 18) {
        return "Afternoon";
    }
    
    return "Evening";
}
```

- **부수 효과(Side-effects)가 있는 코드:** 외부 상태를 변경하거나 의존하는 코드입니다. 데이터베이스에서 값을 읽어오거나(의존), 값을 저장하는(변경) 코드가 여기에 해당합니다.

```java
public void checkPossibleAttend(String eventId, String email) {
    Assert.hasLength(eventId, "Event Id must not be empty");
    Assert.hasLength(email, "Email must not be empty");
    
    // 이메일 형식 확인
    if (!EMAIL_REGEX.matcher(email).matches()) {
        throw new IllegalArgumentException("Invalid email : %s".formatted(email));
    }
    
    // 비결정적 코드
    var attendedCount = repository.count();
    
    // 참여 가능 인원수 확인
    if (attendedCount >= ATTEND_LIMIT) {
        throw new EventLimitExceededException("Event Id : %s attend limit exceeded".formatted(eventId));
    }
}
```
<br><br>

### **(3)** 왜 분리해야 하는가?

```java
public void checkPossibleAttend(String eventId, String email, long attendCount) {
    Assert.hasLength(eventId, "Event Id must not be empty");
    Assert.hasLength(email, "Email must not be empty");
    
    if (!EMAIL_REGEX.matcher(email).matches()) {
        throw new IllegalArgumentException("Invalid email : %s".formatted(email));
    }
    
    if (attendCount < ATTEND_LIMIT) {
        throw new EventLimitExceededException("Event Id : %s attend limit exceeded".formatted(eventId));
    }
}

public long getAttendCount(String eventId) {
    var attendCount = repository.countById(eventId);
    return attendCount;
}
```

`checkPossibleAttend` 메서드 예시를 보면, 하나의 메서드 안에 테스트하기 쉬운 부분(`이메일 형식 확인`)과 테스트하기 어려운 부분(`참여 가능 인원수 확인` - `repository.count()` 호출)이 섞여 있습니다.

이처럼 테스트하기 쉬운 코드와 어려운 코드가 섞여 있으면, 전체 메서드가 **테스트하기 어려운 메서드**가 되어 버립니다. `repository.count()`는 매번 다른 결과를 반환할 수 있는 비결정적 코드이기 때문에, 이 메서드를 테스트하려면 실제 데이터베이스를 사용해야 하거나 Mocking을 복잡하게 구성해야 하는 문제가 발생합니다.
<br><br>

### **(4)** 해결 방법: Humble Object Pattern (겸손한 객체 패턴)

해결책은 "테스트가 쉬운 메서드와 어려운 메서드가 가장 바깥쪽에서 만나게 하는 것"입니다. 이를 **Humble Object Pattern**이라고 부릅니다.

```java
public String attend(String eventId, AttendRequest request) {
    Assert.hasLength(eventId, "Event Id must not be empty");
    Assert.notNull(eventId, "AttendRequest Id must not be null");
    
    long attendCount = getAttendCount(eventId);
    domainService.checkPossibleAttend(eventId, request.getEmail(), attendCount);
    
    var saved = eventRepository.save(request.toEntity(UUID.randomUUID().toString()));
    
    return saved.getId();
}

public void checkPossibleAttend(String eventId, String email, long attendCount) {
    Assert.hasLength(eventId, "Event Id must not be empty");
    Assert.hasLength(email, "Email must not be empty");
    
    if (!EMAIL_REGEX.matcher(email).matches()) {
        throw new IllegalArgumentException("Invalid email : %s".formatted(email));
    }
    
    if (attendCount < ATTEND_LIMIT) {
        throw new EventLimitExceededException("Event Id : %s attend limit exceeded".formatted(eventId));
    }
}

public long getAttendCount(String eventId) {
    var attendCount = repository.countById(eventId);
    return attendCount;
}
```

1. **테스트하기 어려운 로직 분리:**
    - `repository.count()`를 호출하는 부분을 `getAttendCount()`라는 별도의 메서드로 분리했습니다. 이 메서드는 외부 상태(데이터베이스)에 의존하는 **테스트하기 어려운 코드**입니다.
2. **테스트하기 쉬운 로직 분리:**
    - 참여 가능 인원을 확인하는 로직(`if (attendCount >= ATTEND_LIMIT)`)을 `checkPossibleAttend()`라는 메서드로 분리했습니다. 이 메서드는 `attendCount`를 파라미터로 받기 때문에, 외부 상태에 의존하지 않고 같은 입력(`attendCount`)에 대해 항상 같은 결과를 반환하는 **테스트하기 쉬운 코드**가 됩니다.
3. **메인 메서드에서 합치기:**
    - `attend()` 메서드는 `getAttendCount()`와 `checkPossibleAttend()`를 호출하여 최종 로직을 완성합니다. `attend()` 메서드는 테스트하기 어렵지만, 이 메서드 자체는 비즈니스 로직을 거의 담고 있지 않고 분리된 메서드들을 호출하는 역할만 하게 됩니다.
    

이렇게 코드를 분리하면, 대부분의 비즈니스 로직이 담긴 `checkPossibleAttend()`와 같은 메서드들을 **단위 테스트(Unit Test)**로 쉽게 검증할 수 있게 됩니다. 테스트하기 어려운 부분(`attend()`, `getAttendCount()`)은 통합 테스트(Integration Test)나 E2E(End-to-End) 테스트와 같은 더 큰 범위의 테스트로 검증할 수 있습니다.
<br><br>

### **(5)** 정리

- **테스트하기 쉬운 코드**는 **결정적인 비즈니스 로직**을 담당하는 역할을 합니다. (e.g., 특정 조건에 따라 값을 계산하거나 검증하는 역할)
- **테스트하기 어려운 코드**는 **외부 시스템과의 상호작용**을 담당하는 역할을 합니다. (e.g., 현재 시간을 가져오거나, 데이터베이스에 접근하거나, 로그를 출력하는 역할)

이 두 가지 역할을 하나의 메서드에 섞어 놓으면, 비즈니스 로직을 검증하는 단순한 테스트를 하고 싶어도 외부 시스템의 복잡한 요소를 함께 고려해야 하므로 테스트가 어려워집니다.

**해결책**: 테스트하기 어려운 코드는 최대한 얇게 만들고, 테스트하기 쉬운 코드를 분리해서 대부분의 로직을 담도록 구성해야 합니다. 이 패턴을 Humble Object Pattern이라고 합니다.

1. **비즈니스 로직(테스트하기 쉬운 책임)을 별도의 메서드나 클래스에 모아놓습니다.** 이 부분은 입출력이 명확하므로 단위 테스트를 통해 쉽게 검증할 수 있습니다.
2. **외부 시스템과의 상호작용(테스트하기 어려운 책임)은 최대한 얇게 만듭니다.** 이 부분은 비즈니스 로직을 최소화하고, 단지 외부 시스템을 호출하는 역할만 하게 합니다.
3. 이 두 가지 역할을 조립하는 코드를 따로 둡니다.

이러한 분리를 통해 궁극적으로 **유지보수가 쉽고 테스트하기 용이한 코드**를 만들 수 있게 됩니다. 이는 저희가 초기부터 추구하고자 하였던, 소프트웨어 개발의 중요한 원칙 중 하나인 **SRP(단일 책임 원칙)**와도 일맥상통하는 이야기입니다.

핵심은 바로 ***"역할과 책임을 잘 나누는 것"***
<br><br><br><br>

# 3. ACC 프로젝트에서의 적용
<br><br>


## 전략 1. 공통 규칙

### (1) **`Given-When-Then` 패턴**

- given에서는 테스트에 필요한 환경 및 데이터와 같은 사전 조건을 준비합니다.
- when에서는 테스트의 핵심 행동, 메서드를 실행합니다.
- then에서는 기대하는 결과를 검증합니다.
<br><br>

### (2) 기타 규칙

- 테스트는 SUT의 디자인(명세)을 기반으로 case를 정의합니다.
- 테스트 케이스는 단일 검증 목적만 수행해야 한다.
- 하나의 테스트는 하나의 assertThat만을 포함하는 것이 가장 이상적입니다.
- 각각의 테스트는 독립적이어야 하며 서로 의존해서는 안됩니다.
- private method는 테스트 하지 않습니다.
    - 하지만, private method의 로직은 반드시 테스트 되어야 합니다. 여기서 하지 말라는 말은 private method를 직접적으로 테스트 하지 말라는 이야기 입니다.
    - 저희가 테스트 코드(case)가 SUT의 명세 기반하에 작성되어 있다면, 명세가 변경되지 않는 한 내부 구현이 어떻게 변경되더라도(ex. private method 추출 등) 테스트 케이스는 Production 코드를 검증할 수 있게 됩니다. 따라서, private method는 직접적으로 테스트 하지 않아도 됩니다.
<br><br>
<br><br>

## 전략 2. 테스트 이름 규칙

테스트 이름을 짓는 것은 단순히 이름을 붙이는 작업이 아니라 소프트웨어의 동작을 문서화하는 중요한 과정이라 생각합니다. 테스트 이름은 통해 테스트 코드를 읽지 않아도 어떤 시나리오를 검증하는지 알 수 있도록 명확해야 합니다. 명확하고 일관된 규칙을 사용하여 테스트 이름을 정의하는 것이 코드의 품질을 높이는 데 큰 도움이 될 듯 합니다.
<br><br>

### (1) **`Given-When-Then` 패턴**

- 메서드 이름은 **`given_조건_when_행위_then_결과`** 패턴으로 작성하여 테스트의 의도를 명확히 합니다.
- 다시 말해,  **'테스트 대상의 상태(Given) → 테스트가 일어난 행위(When) → 기대하는 결과(Then)’**의 논리적인 흐름을 표현합니다.
<br><br>

### (2) JUnit5의 `@DisplayName`

- **`@DisplayName` 을 활용**합니다.
- 한글로 시나리오를 상세하게 작성합니다.
- 한글로 된 문장을 사용하여 테스트의 의도를 상세하게 풀어서 설명합니다.
(ex. "회원가입 시 유효하지 않은 이메일 형식일 경우, 예외가 발생한다")
- 테스트 이름은 테스트 목적을 명확하게 들어낼 수 있게 작성합니다.
<br><br>

### (3) Example

- **예시 1: 간단한 계산기 `add` 메서드**
    - Given: `add` 메서드에 두 정수 `5`와 `10`이 주어졌을 때
    - When: `add` 메서드를 호출하면
    - Then: `15`라는 결과가 반환된다
        - 테스트 이름 (영문): `givenTwoIntegers_whenAddMethodIsCalled_thenReturnsSum`
        - 테스트 이름 (한글, `@DisplayName`): "두 정수 5와 10이 주어졌을 때, 덧셈 메서드를 호출하면 15를 반환한다"
- **예시 2: 회원 가입 서비스 `join` 메서드**
    - Given: 유효한 이메일 형식과 비밀번호가 주어졌을 때
    - When: 회원 가입 서비스의 `join` 메서드를 호출하면
    - Then: 회원 데이터가 데이터베이스에 저장된다
        - 테스트 이름 (영문): `givenValidCredentials_whenJoinMethodIsCalled_thenUserIsSaved`
        - 테스트 이름 (한글, `@DisplayName`): "유효한 회원 정보로 회원 가입 메서드를 호출하면, 사용자 정보가 데이터베이스에 저장된다"
- **예시 3: 예외를 검증하는 테스트**
    - Given: 잔액이 10,000원인 계좌가 있을 때
    - When: 15,000원을 출금하는 메서드를 호출하면
    - Then: 잔액 부족 예외(`InsufficientFundsException`)가 발생한다
        - 테스트 이름 (영문): `givenBalanceIs10000_whenWithdrawal15000_thenThrowsInsufficientFundsException`
        - 테스트 이름 (한글, `@DisplayName`): "잔액이 10,000원일 때 15,000원을 출금 요청하면, 잔액 부족 예외가 발생해야 한다"
- **예시 4: 외부 API 호출이 포함된 테스트**
    - Given: 외부 결제 API 호출이 성공적으로 응답하도록 설정되어 있을 때 (Mocking)
    - When: 결제 서비스의 `pay` 메서드를 호출하면
    - Then: 결제 상태가 `SUCCESS`로 변경된다
        - 테스트 이름 (영문): `givenPaymentApiSuccessResponse_whenPayMethodIsCalled_thenPaymentStatusIsSuccess`
        - 테스트 이름 (한글, `@DisplayName`): "결제 API가 성공적으로 응답할 때, 결제 메서드를 호출하면 결제 상태가 성공으로 변경된다"
<br><br><br><br>


# 4. 단위 테스트(Unit Test)

개별 클래스나 메서드 같은 작은 단위의 코드를 테스트합니다. 외부 시스템(DB, API 등)에 의존하지 않고, **테스트 더블**을 사용하여 SUT의 로직 자체에 집중합니다. 우리의 `Module`과 `Util` 클래스는 주로 단위 테스트의 대상이 됩니다.

<br><br>


## **전략 3.** 프로젝트 구조에 따른 테스트 규칙

저희 프로젝트의 아키텍처는 **테스트하기 쉬운 코드**와 **테스트하기 어려운 코드**가 명확하게 분리되어 있습니다.


- **테스트하기 쉬운 코드:** `Util` 클래스처럼 외부 의존성이 없고, 동일한 입력에 대해 항상 동일한 출력을 보장하는 **결정적(Deterministic)** 코드입니다.
- **테스트하기 어려운 코드:** `Module` 클래스처럼 `WebClient`를 사용해 외부 API에 접근하거나 `JavaMailSender`를 사용하는 등 **부수 효과(Side-effects)**가 있는 코드입니다.
<br><br>

### (1) Util 테스트(**테스트하기 쉬운 코드)**

- **`Util`은 `Mock`이 필요 없습니다.**
- 외부 의존성이 없으므로, 실제 객체를 그대로 사용해 입력과 출력을 검증하는 순수한 단위 테스트를 작성합니다.
- `assertThat()`을 사용하여 예상과 일치하는지 확인합니다.
<br><br>

### **(2) `Module` 테스트 #1 (테스트하기 어려운 코드)**

- `Module`은 외부 API 통신과 같은 부수 효과를 포함합니다. 따라서 **`Mock`을 사용해 외부 의존성을 격리**해야 합니다.
- 외부 API와의 통신 로직(요청 포맷, URL, 헤더 등)과 응답 데이터 처리 로직이 올바른지 검증
- `WebClient`나 `JavaMailSender`와 같은 외부 통신 객체를 **`Mock`**으로 만들고, SUT가 올바르게 호출하는지 `verify()`로 확인합니다.
<br><br>

### (3) **`Module` 테스트 #2 (테스트하기 어려운 코드)**

- `Mockito`는 Java에서 사용되는 모의 객체 사용을 위한 라이브러리입니다.
- Mockito와 의존성 주입의 개념을 활용하면 테스트하려는 객체가 의존하는 다른 객체들을 쉽게 모의 객체로 만들어서 테스트를 진행할 수 있습니다.
- Stub, Mock의 기능을 모두 활용할 수 있습니다.
<br><br>

**예시 1**

```java
@AllArgsConstructor
public class Animal {

  private int age;
  private final AnimalService animalService;

  public int ageAfterOneYear() {
    age = animalService.ageAfterOneYear(age);
    return age;
  }
}

public class AnimalService {

  public int ageAfterOneYear(int age) {
    return age + 1;
  }
}

public class SampleTest {

  @Test
  void test() {
    // Given
    int age = 10;
    AnimalService animalService = Mockito.mock(AnimalService.class);
    Animal animal = new Animal(age, animalService);
    Mockito.when(animalService.ageAfterOneYear(age)).thenReturn(age + 1);

    // When
    int nextAge = animal.ageAfterOneYear();

    // Then
    Assertions.assertThat(nextAge).isEqualTo(age + 1);
    Mockito.verify(animalService, Mockito.times(1)).ageAfterOneYear(age);
  }
}
```

예제를 보면 age와 AnimalService를 의존성 주입 받는 Animal 클래스가 존재하고, 테스트에서는 Animal 클래스의 ageAfterOneYear 메서드를 테스트하려는 것을 확인할 수 있습니다. 단위 테스트 관점에서는 AnimalService가 끼어들면 안됩니다. 따라서, AnimalService 모의 객체를 만들어서 테스트를 진행해야합니다. 

- `when` 문법을 통해 모의 객체가 어떤 입력에 대한 어떤 값을 반환할지에 대한 Stub의 기능을 정의합니다.
- `verify`과 같은 mock의 기능 문법을 통해 실제로 모의 객체에 어떤 일이 일어났는지를 검증할 수 있습니다.
- **`@Mock`과 `@InjectMocks` 애너테이션 사용**
`@InjectMocks`는 `userRepository`와 `paymentService`의 Mock 객체를 자동으로 `myService`에 주입해 줍니다. 이는 테스트 준비(Arrange) 단계를 간소화합니다.
<br><br>

**예시 2**

```java
@ExtendWith(MockitoExtension.class)
class MyServiceTest {

	@Mock
	private UserRepository userRepository;

	@Mock
	private PaymentService paymentService;

	@InjectMocks
	private MyService myService; // 테스트 대상 SUT

	@Test
	void testSomething() {
	    // ...
	}
}

```
<br><br>


이때, 사용되는  `@Mock` 에 대해서 다음 내용을 알고 있어야 합니다.

| 특징 | `@Mock` | `@MockBean` |
| --- | --- | --- |
| **속한 프레임워크** | Mockito | Spring Boot Test |
| **동작 범위** | 스프링 컨텍스트 외부 | 스프링 컨텍스트 내부 |
| **주입 방법** | `@InjectMocks`를 사용해 직접 주입 | `@Autowired`를 통해 컨텍스트에서 주입 |
| **주요 사용처** | 순수 단위 테스트 | 통합 테스트 |
| **빈 재정의 여부** | 해당 없음 (빈이 아님) | 기존 컨텍스트 빈을 대체하거나 추가 |
- `@MockBean`은 **통합 테스트** 환경에서, 스프링 컨텍스트에 이미 존재하는 실제 빈을 **모의 객체로 교체**하는 데 사용됩니다.
- 이는 `@SpringBootTest`와 같이 전체 애플리케이션 컨텍스트를 로드하는 테스트에서 특정 컴포넌트(예: 외부 API 클라이언트, 데이터베이스 리포지토리)만 가짜로 만들어 테스트의 범위를 제어할 때 특히 유용합니다.
<br><br><br><br>


## 전략 4. 테스트 불가능 영역의 대처

<img width="840" height="662" alt="Image" src="https://github.com/user-attachments/assets/ab30d16a-f199-4e9d-a7ca-e6f95e731d9d" />

<img width="848" height="668" alt="Image" src="https://github.com/user-attachments/assets/524038f6-3bf8-4bd3-a62d-4dc876a29f88" />

저희 프로젝트의 특성상 다양한 이유로 테스트 코드 작성이 불가능한 이유가 필연적으로 발생하며 그것에 대해 어떻게 대처할지를 생각해봤습니다. xxx 이유로 테스트 코드 작성이 어려운 영역을 **Black Box 영역**이라 지칭하겠습니다.

Black Box 영역의 가장 큰 문제점은 이 영역이 전이된다는 것입니다. 저희는 Black Box 영역을 직/간접적으로 의존하는 모든 구간들이 Black Box로 전이되지 않도록 격리해야합니다.  **Black Box 영역을 테스트 못하더라도 다른 객체는 여전히 테스트를 진행할 수 있는 환경을 구성해야 합니다.**

예를 들어, Keycloak과 같은 외부 시스템은 **직접 테스트하기 어려운 코드**라고 생각합니다. 이때 저희는 **Mock을 사용한 단위 테스트**를 통해 우리 애플리케이션의 비즈니스 로직을 검증 하는 것에 초점을 맞추는 것이 좋다고 생각합니다.

`@MockBean`을 사용하면 스프링 컨텍스트에 이미 등록된 `KeycloakClient` 빈을 테스트에서만 Mock 객체로 대체할 수 있어, Black Box 영역에 대한 의존성을 쉽게 격리할 수 있습니다.

<img width="860" height="676" alt="Image" src="https://github.com/user-attachments/assets/6c9e056e-8c3d-45b7-87a8-211b7952841b" />

출처 : https://tech.kakaopay.com/post/mock-test-code/
<br><br>
<br><br>

## 전략 5. **`@TestConfiguration` 의 사용**
<br><br>

### **(1) `@TestConfiguration` 이란?**

- **`@TestConfiguration` 은 테스트 환경**에서만 사용되는 빈을 정의할 때 사용합니다.
- 이를 정의하여, 테스트 전용 빈을 만들어 프로덕션 환경의 빈과 분리합니다. 예를 들어, 실제 데이터베이스에 연결하는 빈 대신 인메모리 데이터베이스 빈을 정의할 수 있습니다.
- `@TestConfiguration`이 붙은 클래스는 스프링의 일반적인 컴포넌트 스캔 대상에서 제외됩니다. 애플리케이션의 메인 컨텍스트에 등록되지 않습니다.
- **유연한 설정**: 여러 테스트 클래스에서 필요한 테스트 빈을 별도의 `@TestConfiguration` 클래스로 만들어 `@Import` 애너테이션을 통해 재사용할 수 있습니다.
<br><br>

### (2) 사용 시점

단순히 메서드 내에서 `Mockito.mock()`을 사용해도 되는 간단한 목업 상황이 아니라, **테스트 환경 자체의 설정(`DataSource` 등)을 재구성하거나, 여러 테스트에서 공유할 복잡한 모의 객체 빈이 필요할 때** `@TestConfiguration`을 정의해야 합니다.

1. 실제 `RDS DataSource` 빈 대신, 테스트에 최적화된 **인메모리 DB(`H2` 등) `DataSource` 빈**을 등록할 때.
2. 여러 테스트 클래스에서 공통으로 사용하는 모의 객체나 테스트 전용 유틸리티 빈
<br><br>

### (3) 사용 방법

- 별도의 클래스로 `@TestConfiguration`을 정의합니다.
- `@Import` 어노테이션을 통해 불러와서 사용합니다.

**예시**

**`TestConfig.java`**

```java
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Primary;
import static org.mockito.Mockito.mock;

@TestConfiguration
public class TestConfig {

    @Bean
    @Primary
    public ExternalApiClient mockExternalApiClient() {
        return mock(ExternalApiClient.class);
    }
}
```

**`MyServiceTest.java`**

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.Import;

@SpringBootTest
@Import(TestConfig.class) // 외부 설정 클래스를 가져옴
class MyServiceTest {
    // ... 테스트 코드
}
```
<br><br><br><br>

# 5. 통합 테스트(Integration Teset)

여러 컴포넌트(클래스, 모듈)가 함께 작동하는 시나리오를 테스트합니다. 실제 DB나 API와의 연결을 포함하여, 계층 간의 상호작용이 올바른지 검증합니다. `Controller`부터 `Service`까지의 전체 흐름을 검증할 때 사용됩니다.

- **`@SpringBootTest` 활용**: 스프링 컨텍스트 전체를 로드하여 통합 테스트를 실행하는 방법을 명시할 수 있습니다.
- **`@WebMvcTest` 활용**: 웹 계층(Controller)만 격리하여 테스트하는 방법을 언급할 수 있습니다. 이는 Service나 Repository 빈을 Mocking하여 테스트 범위를 줄이는 데 효과적입니다.
- **API 엔드포인트 테스트**: `MockMvc` 또는 `TestRestTemplate`를 사용하여 실제 HTTP 요청을 보내고 응답을 검증하는 예시를 추가할 수 있습니다.

**예시**

```java
@WebMvcTest(UserController.class) // UserController만 테스트
class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean // UserController가 의존하는 UserService를 Mock으로 교체
    private UserService userService;
    
    @Test
    void userCanBeRetrievedById() throws Exception {
        // given
        given(userService.findById(1L)).willReturn(new User("testuser"));
        
        // when & then
        mockMvc.perform(get("/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.username").value("testuser"));
    }
}
```
<br><br><br><br>

# REFERENCE


https://codinghack.tistory.com/92

https://tecoble.techcourse.co.kr/post/2020-09-19-what-is-test-double/

https://kakaoentertainment-tech.tistory.com/78

https://tech.kakaopay.com/post/mock-test-code/

https://tech.kakaopay.com/post/mock-test-code-part-2/

https://tech.kakaopay.com/post/given-test-code/

https://middlefitting.tistory.com/102

https://devbksheen.tistory.com/entry/%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0JUnit-TDD-BDD-Mockito