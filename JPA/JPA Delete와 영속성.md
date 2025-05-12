## 🔍 Background

onRank 프로젝트에서 Study관련 `REST API` 개발 중, `Delete API`를 개발하는 과정에서 문제가 발생하였다. 아래의 ERD는 개발에 사용되는 ERD의 일부분이다.

![Image](https://github.com/user-attachments/assets/e669f269-b881-4217-8cf6-89b2edfaa38c)
<br></br>
## 🔍 Delete API

Study를 삭제할 때, @OneToMany관계를 이루고 있는, Notice, Post, Schedule, Assignment Entity와 가각의 Entity와 관련있는, FileMetadata Entity까지 삭제가 되어야 한다.
<br></br>
## 📘 JPA 학습(1) - 영속성 전이 (Cascade)

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "studies")
public class Study {

		// ...중략

    @OneToMany(mappedBy = "study", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Member> members = new ArrayList<>();

    @OneToMany(mappedBy = "study", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Notice> notices = new ArrayList<>();
    
    @OneToMany(mappedBy = "study", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Post> posts = new ArrayList<>();

    @OneToMany(mappedBy = "study", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Schedule> schedules = new ArrayList<>();

    @OneToMany(mappedBy = "study", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Assignment> assignments = new ArrayList<>();

		// ...중략
    
}
```
<br></br>
JPA에서 **부모 엔티티(예: Study)** 의 상태 변화(저장, 삭제 등)를 **자식 엔티티(예: Notice, Member 등)에게 전파하는 설정**으로, 대표적으로 다음과 같은 옵션이 있다.

| 옵션 | 설명 |
| --- | --- |
| `PERSIST` | 부모 저장 시 자식도 함께 저장 |
| `REMOVE` | 부모 삭제 시 자식도 삭제 |
| `MERGE` | 병합 시 함께 병합 |
| `DETACH` | 영속성 해제 시 함께 해제 |
| `ALL` | 위 모든 옵션을 포함 |

난 Study가 삭제될 때, 연관된 모든 연관 관계 Entity들이 전부 삭제되기를 원하기에 ALL 옵션을 사용하였다.
<br></br>
## ⚡️ 문제 발생 deleteStudy(…)

- **메서드**: `StudyService.deleteStudy(String username, Long studyId)`
- **기능 목적**: 특정 스터디를 삭제할 때 관련된 모든 엔티티 및 S3 파일까지 함께 제거
- **증상**:
    - `studyRepository.delete(study)` 실행에도 불구하고 **Study가 삭제되지 않는다.**
    - **FileMetadata 또한 삭제되지 않고 DB에 그대로 남는다.**
    - 로그상 예외는 없지만, Delete 쿼리가 실행되지 않는다.
<br></br>
![Image](https://github.com/user-attachments/assets/0b5bd1fc-d83b-4b53-b6d7-af06e2e2d18e)
<br></br>
해당 코드는 다음의 순서로 작동한다:

1. **권한 확인**: `memberService.isMemberCreatorOrHost(...)`
2. **공지사항, 게시글, 과제에 연관된 S3/DB 파일 삭제**
3. **Study의 FileMetadata 삭제**
4. **Study 조회 및 삭제**

하지만 실제 삭제는 이루어지지 않았다.
<br></br>
## 🔍 원인 분석(1)

문제의 근본적인 원인이 무엇인지 알아보자.

 첫 번째,  `memberService.isMemberCreatorOrHost(...)` 메서드를 살펴보자.

![Image](https://github.com/user-attachments/assets/df80aa4e-2a44-4587-9c01-2dd0f8cbcd58)
<br></br>
결론적으로, `isMemberCreatorOrHost(...)` 호출만으로는 Study의 `members` 컬렉션이 로딩되지 않는다. 
<br></br>
`memberRepository.findByStudentStudentIdAndStudyStudyId(...)` 내부에서 `fetch join`이나 `@EntityGraph`를 사용해서 `Study`를 **즉시 로딩(fetch)** 하고, 거기서 다시 `study.getMembers()`를 호출하거나 `Study.members`를 억지로 접근했다면 **그때는 로딩된다.**
<br></br>
하지만, 현재는 레포지토리 메서드를 다음과 같이 **`Study.members`를 로딩하지 않도록 작성하였기에**, `isMemberCreatorOrHost(...)`는 삭제 실패 원인이 아니다.

```java
Optional<Member> findByStudentStudentIdAndStudyStudyId(Long studentId, Long studyId);
```

이 메서드는 JPA의 **파라미터 기반 쿼리 메서드**이며, 내부적으로는 아래와 유사한 JPQL이 생성된다.

```sql
SELECT m FROM Member m
WHERE m.student.studentId = :studentId
         AND m.study.studyId = :studyId
```
<br></br>
다시 한번 말하면, 이때 `m.study`는 **프록시(지연 로딩)** 상태로 로딩되고, **`m.study.getMembers()`를 직접 호출하지 않는 한** `members` 컬렉션은 로딩되지 않는다. 따라서 `isMemberCreatorOrHost(...)`는 `studyRepository.delete(study)`가 실패한 원인이 아니다.
<br></br>
## 🔍 원인 분석(2)

![Image](https://github.com/user-attachments/assets/518e1b3b-a6ce-4c93-85fd-eabefda60008)

위의 `noticeRepository.findByStudyStudyId(...)` 등의 호출로 인해, Notice, Post, Assignment 등의 객체들이 영속성 컨텍스트에 로딩된다. 이후 `studyRepository.delete(study)`를 호출하면, JPA는 Study와 연결된 `Study.notices` 컬렉션에 이미 **영속성 컨텍스트에서 존재하는 자식 객체가 있다고 판단하고,** 이를 **고아 객체로 인식되지 않는다.** 즉, `orphanRemoval = true`가 동작하지 않고, 삭제 무시 또는 실패가 일어난다.
<br></br>
## 📘 JPA 학습(2) - orphanRemoval = true

**부모 엔티티의 자식 컬렉션에서 자식 엔티티를 제거하면,** JPA는 이를 **"고아 객체"**로 간주하고, **DB에서 DELETE 쿼리를 날린다.** 사용 예시로는 다음과 같다.

```java
@Entity
public class Study {
    @OneToMany(mappedBy = "study", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Member> members = new ArrayList<>();
}
```

```java
// Study 내부에서
study.getMembers().remove(member); // → member 엔티티는 DB에서 삭제됨
```
<br></br>
`cascade = REMOVE`와는 다르게, 단순히 컬렉션에서 제거만 해도 삭제된다. 이둘의 차이에 대해 한번 정리해 보자.
<br></br>
### `cascade = REMOVE` vs `orphanRemoval = true`

| 항목 | `cascade = REMOVE` | `orphanRemoval = true` |
| --- | --- | --- |
| 언제 동작? | 부모 엔티티가 삭제될 때 | 부모가 자식을 컬렉션에서 제거할 때 |
| 대상 | 부모 엔티티의 생명주기와 같이 삭제 | 컬렉션에서 빠진 자식 |
| 필수 조건 | 없음 (단방향 가능) | 자식이 `컬렉션에서 제거`되어야 작동 |
<br></br>
## 🔍 문제 분석 번외편 - FileMetadata

이때, `studyRepository.delete(...)`에서 실패가 발생하였기에, 그 이전에 진행한 Post, Notice 등의 첨부파일에 대한 DB의 `FileMetadata`도 모두 **rollback**되어 삭제되지 않는다.

(이때, S3의 파일은 DB와 관련이 없기에 전부 정상적으로 삭제 되었다. 따라서, FileMetadata는 남아있지만, 실제 파일 객체는 S3에 존재하지 않는 것이다.)
<br></br>
### 왜냐하면?

`@Transactional`에 의해 `studyRepository.delete(...)`에서 실제 삭제가 실패했다면, 그 전에 호출된 `fileService.deleteAllFilesAndMetadata(...)` 역시 결국 DB에는 반영되지 않았기 때문이다.
<br></br>
Spring의 `@Transactional`은 다음과 같은 **동작 순서를 가집니다**:

1. `deleteStudy()` 메서드 시작 시 트랜잭션 시작
2. `fileService.deleteAllFilesAndMetadata(...)` 호출
    
    → `fileMetadataRepository.deleteAll(...)` 수행 (JPA는 아직 **flush하지 않음**)
    
3. 이후 `studyRepository.delete(...)` 수행
4. 여기서 오류 발생 or 삭제 실패 (영속성 컨텍스트 충돌 등)
5. `@Transactional`에 의해 전체 트랜잭션 **롤백**
6. `deleteAll(...)`도 포함된 모든 작업 **DB 반영 안 됨**
<br></br>
이에 대해 다음과 같은 사실들을 알 수 있었다

- `fileService.deleteAllFilesAndMetadata()`의 Service 메서드는 자체적으로 내부에서 `@Transactional`을 사용하더라도 **이미 상위 트랜잭션 안에서 실행되므로 별도로 커밋되지는 않는다.**
- 즉, **모든 삭제 작업은 하나의 트랜잭션에 묶여 있음**을 알 수 있다.
- 따라서 `studyRepository.delete()`가 실패하면, **그 이전까지의 모든 삭제 요청도 rollback된다.**
<br></br>
### 예시 흐름

```
@Transactional
└─ deleteStudy()
    ├─ fileService.deleteAllFilesAndMetadata(...)   ← DB에는 아직 반영 안 됨
    └─ studyRepository.delete(study)                ← 실패 발생 (ex. 연관 컬렉션 충돌)
    
[트랜잭션 롤백] → 이전 파일 메타데이터 삭제도 모두 무효
```
<br></br>
## 📘 JPA 학습(3) - @Transactional

`@Transactional` (트랜잭션 경계 선언)은 메서드나 클래스에 적용되어 **하나의 작업 단위(Unit of Work)**를 보장한다. 이 어노테이션이 적용된 메서드 내부에서 발생하는 모든 DB 연산은 하나의 트랜잭션으로 묶이며, **예외 발생 시 전체 롤백**된다. 스프링에서는 일반적으로 Service 계층에서 선언하며, `RuntimeException` 발생 시 자동으로 롤백된다. 트랜잭션 경계 밖에서 Repository를 new로 생성하거나 호출하면 AOP 프록시가 적용되지 않아 동작하지 않는다. 본 케이스에서 중요 포인트로는, 파일 삭제, 엔티티 삭제 등 모든 연산이 하나의 트랜잭션에서 처리되어야 **삭제 실패 시 롤백**이 가능하므로 `@Transactional` 필수적임을 알 수 있다.
<br></br>
## ✅ Solution

결과적으로 해당 문제는 `JPA 영속성 컨텍스트 이해 부족`, `JPA 연관 관계 매핑과 지연 로딩에 대한 이해 부족` 등의 JPA 이해도가 부족하여 발생한 문제이다. 해당 문제를 해결하기 위해서는 **연관 관계를 끊어 주는 것이 핵심이다**. 다음과 같은 **삭제 연관 관계 편의 메소드**를 만들어서 **연관 관계를 끊어줄 수 있다.**
<br></br>
Study 엔티티에 추가할 `clearAllRelations()` 메서드를 통해 One To Many 연관 관계를 끊어 JPA는 `orphanRemoval = true`가 설정된 컬렉션에서 객체가 제거되면, 해당 객체를 orphan(고아)로 판단하여 자동으로 **DELETE**한다.

![Image](https://github.com/user-attachments/assets/cb5f29b2-3976-4647-9dcd-afb716c09936)
<br></br>
## ❗ 단, 아래 조건이 모두 충족돼야 삭제가 동작한다.

| 조건 | 설명 |
| --- | --- |
| `orphanRemoval = true` | Study에서 제거된 자식은 JPA가 DB에서 삭제 처리 |
| `cascade = CascadeType.ALL` | 삭제 연쇄 전파를 보장 |
| `mappedBy = "study"` | 연관 관계의 주인이 Study가 아님을 명확히 설정 |
| `study.getMembers()`는 JPA가 관리하는 영속 컬렉션일 것 | 즉, `study`는 반드시 `studyRepository.findById()` 등으로 **영속 상태**로 불러온 객체여야 함 |
<br></br>
## ✅ 최종 Code

```java
Study study = studyRepository.findById(studyId)
        .orElseThrow(() -> new CustomException(STUDY_NOT_FOUND));

// 연관관계 끊기
study.clearAllRelations();

// 스터디 삭제
studyRepository.delete(study);
```

삭제시 호출 로직
<br></br>
![Image](https://github.com/user-attachments/assets/a68308b4-c3a1-4968-a86d-cffaa3d4176f)

최종 코드
<br></br>
## 🔍 왜 `clear()`로 충분한가?

study.getMembers().clear()만 하면 study.getMembers().remove(member)를 반복해서 하나씩 지우는 것과 동일한 효과를 낼 수 있다. `List.clear()`는 내부적으로 `remove()`를 반복 호출합니다. 즉 다음 두 코드는 동일한 동작을 수행한다.
<br></br>
(1) 명시적 반복 제거

```java
for (Member member : study.getMembers()) {
    study.getMembers().remove(member);
}
```

(2) 단순 clear()

```java
study.getMembers().clear();
```
<br></br>
JPA는 `orphanRemoval = true`가 설정된 컬렉션에서 객체가 제거되면, 해당 객체를 orphan(고아)로 판단하여 자동으로 **DELETE** 한다.
<br></br>
## ⚡️ **영속성과 지연 로딩의 관점에서,**

그렇다면 반드시 삭제시 연관 관계 편의 메소드를 작성해야 할까? 그렇지는 않다. 

**지연 로딩에 의해서 `Entity`를 불러오면 연관된 `Entity`들은 실제로 참조되기 전까지 프록시 객체로 존재한다. 즉 애초에 영속화 시키지 않는다는 말이다.**

앞서 말했듯 이 문제의 근본적인 원인은 `fetch join`으로 인해 삭제 처리해야할 Entity가 영속화된 것이 문제였습니다. 때문에 `fetch join`을 사용하지 않고 **지연 로딩을 이용한 `find 메소드`를 사용하여 `Notice Entity`만 영속화**한다면 해당 삭제 연관 관계 편의 메소드 없이도 삭제 처리가 될 것이다.

하지만 **삭제 비즈니스 로직 상, 연관된 `Entity`들이 영속화 되는 경우에는 연관 관계를 끊어주는 것이 필요하다.**
<br></br>
## 📚 참고

[[Spring boot] JPA Delete is not Working, 영속성와 연관 관계를 고려했는가.](https://velog.io/@jsb100800/spring-12)