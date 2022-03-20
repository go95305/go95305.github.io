---
layout: single
title:  "Spring AOP"
category:
- Spring
- aop
---

<br/>

### Spring AOP

많은 기술면접을 보게되었고 준비할때나 실제 면접때나 항상 나오는 질문으로는 'Spring AOP에 대해 아세요?','Spring AOP에 대해설명해주세요' 였다.
하지만 항상 자신있게 '모르겠습니다..' 라는대답만하였고 진짜 spring을 사용하면서 aop에 대한 개념을 학습할 필요가 있다는 것을 알게 되었다.

간단히 설명하면 '공통된 관심사를 분리하여 관리한다'라고 이해하면될 것같다. @Aspect와 @Around 어노테이션을 적절히 잘 사용하면 중복된 코드를 줄이고 공통된 부분을 쉽게 관리할 수 있다.

코드 예제를 보기전에 내가 aop를 한번에 이해할 수 있었던 또 다른 예제에 대해 말하자면 <b>@Transactional</b> 어노테이션이다.

@Transactional을 사용하게 되면 commit(), rollback()을 코드 상에 작성하지 않더라도 자동으로 수행된다는 점을 알고 있을것이다.

<br/>

~~~java
@Transactional(readOnly=true)
public void deletePost(Long id) throws NoBoardException{
  BoardEntity boardEntity = boardRepository.findById(id).orElseThrow(() -> new NoBoardException(HttpStatus.BAD_REQUEST, "삭제할 게시판이 없습니다."));
  boardEntity.setDelete();
  }
~~~
<br/>
위의 코드를 보면 @Transactional 어노테이션이 붙어있어서 <b>boardEntity.setDelete()</b> 이후에 commit()이 자동으로 수행된다는 것을 알 수 있다.

aop도 간단한 설정을 통해 자동으로 특정 기능을 수행해 준다고 생각하면된다. 그리고 이런 기능 수행을 관심사 즉, 개발자 본인이 구분하여 분류할 수 있다는 뜻이다.

<br/>

### BoardController.java
~~~java
    @LogExecutionTime  // 1. 어노테이션 추가
    @GetMapping
    public ResponseEntity<ResponseMessage<Page<BoardSimpleDto>>> list(@RequestParam(value = "keyword", defaultValue = "") String keyword, Pageable pageable) {
        Page<BoardSimpleDto> page = boardService.getBoardlist(keyword, pageable);
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON_UTF8);

        return ResponseEntity.ok()
                .headers(headers)
                .body(ResponseMessage.<Page<BoardSimpleDto>>builder()
                        .message("게시판 리스트 가져오기")
                        .data(page)
                        .build());

    }
~~~


중복된 코드인 메소드 실행시간을 구하는 코드를 aop로 구현해 볼 것이다. 필자는 어노테이션을 이용하였지만 굳이 어노테이션이 아니라 경로 설정으로 구현할 수 도 있다.

<br/>

###  LogExecutionTime.java
~~~java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
    
}
~~~
@Target(ElementType.METHOD)
- 메소드에 어노테이션을 적용한다는 뜻.

<br/>

@Retention(RetentionPolicy.RUNTIME)
- 어노테이션이 runtime까지 유지되도록 설정.

<br/>

### LogAspect.java
~~~java
@Component
@Aspect
public class LogAspect {
    Logger logger = LoggerFactory.getLogger(LogAspect.class);

    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        Object proceed = pjp.proceed();
        stopWatch.stop();
        logger.info(stopWatch.prettyPrint());
        return proceed;
    }
}
~~~

@Around("@annotation(LogExecutionTime)")
- LogExecutionTime어노테이션을 사용하면 실행된다는 뜻.

<br/>

### 실행결과
~~~
2022-03-20 18:00:59.899 DEBUG 30859 --- [nio-8080-exec-1] org.hibernate.SQL                        : select boardentit0_.id as id1_0_, boardentit0_.created_date as created_2_0_, boardentit0_.modified_date as modified3_0_, boardentit0_.content as content4_0_, boardentit0_.is_deleted as is_delet5_0_, boardentit0_.is_modified as is_modif6_0_, boardentit0_.title as title7_0_, boardentit0_.writer as writer8_0_ from board boardentit0_ where (boardentit0_.title like ? escape ?) and boardentit0_.is_deleted=0 limit ?
Hibernate: select boardentit0_.id as id1_0_, boardentit0_.created_date as created_2_0_, boardentit0_.modified_date as modified3_0_, boardentit0_.content as content4_0_, boardentit0_.is_deleted as is_delet5_0_, boardentit0_.is_modified as is_modif6_0_, boardentit0_.title as title7_0_, boardentit0_.writer as writer8_0_ from board boardentit0_ where (boardentit0_.title like ? escape ?) and boardentit0_.is_deleted=0 limit ?
2022-03-20 18:00:59.901 TRACE 30859 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [%%]
2022-03-20 18:00:59.901 TRACE 30859 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [CHAR] - [\]
2022-03-20 18:00:59.912  INFO 30859 --- [nio-8080-exec-1] com.board.config.LogAspect               : StopWatch '': running time (millis) = 66
-----------------------------------------
ms     %     Task name
-----------------------------------------
00066  100% 
~~~

<br/>

정리하자면, 어노테이션을 통해 공통실행코드의 위치를 설정해 주었고 @Around를 통해 해당 어노테이션이 위치한 메소드에 공통된 메소드가 실행된다는 뜻이다.


<br/>

### REFERENCE TO
- <https://atoz-develop.tistory.com/entry/Spring-%EC%8A%A4%ED%94%84%EB%A7%81-AOP-%EA%B0%9C%EB%85%90-%EC%9D%B4%ED%95%B4-%EB%B0%8F-%EC%A0%81%EC%9A%A9-%EB%B0%A9%EB%B2%95>