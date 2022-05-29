---
layout: single
title:  "Spring Bean, Spring IoC, Annotation의 관계"
category:
- Spring IoC
- Bean
- Annotation
---

<br/>

#### Spring Bean이란?
*Spring IoC 컨테이너가 관리하는 자바 객체.*<br/>
Spring Bean을 설명하기전에 우리가 전통적으로 알고있는 Java Bean부터 설명하겠다.<br/>
간단히 `new`를 통해 Heap Memory에 저장되는 데이터값이라고 생각하면 간단할거같다. 우리가 알고 있는 Java프로그래밍에서는 새로운 객체 생성시 매번
new를 통해 메모리에 저장해왔다. 생성해야할 객체 수가 늘어날수록 new의 사용은 비례하여 증가했다.

~~~java
Mail mail = new Mail(); //new 명령어를 통해 Mail객체를 생성하여 mail변수에 주입.
~~~

<br/>

Spring Bean은 이런 new필요없이 Spring에서 자체적으로 생성하여 알맞은 변수에 셋팅해준다고 생각하면된다.<br/>
`@Controller`를 예로 들겠다. Front단에서 HTTP요청(GET,POST,PUT 등등)을 서버단으로 보낸다. 그리고 이런 요청을 Servlet을 통해 Controller단에서 우선 처리를 하게 되는데 Controller.java가 Bean등록이 안되어있으면 어떻게 될까?
**당연히** 실행되지않는다. Bean객체에 등록되어야 메모리상에 적재되고, 메모리상에 들어와있어야 사용이 가능하다.

<br/>

여기서 `@Controller`어노테이션을 사용하면 자동으로 Spring이 Bean등록을 해준다.
~~~java
@Controller
@RequestMapping("/post")
public class BoardController {
    
    //생략
}
~~~
<br/>

`@Controller` Annotation에 들어가보면 `@Component` Annotation이 사용된것을 확인할 수 있다.
Spring에서는 여러 가지 Annotation을 사용하지만, Bean을 등록하기 위해서는 @Component Annotation을 사용한다. @Component Annotation이 등록되어 있는 경우에는 Spring이 Annotation을 확인하고 자체적으로 Bean 으로 등록한다.

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";

}
~~~

<br/>

그렇다고 꼭 Annotation을 Spring IoC를 통해 Bean객체를 등록 해주는대만 사용하냐? 그건또아니다.<br/> 
바로 **의존성 주입**에도 사용된다.
`@Service`, `@Autowired` Annotation을 예시로 들겠다.

<br/>

아래 코드에서 만약  `@Autowired`없이 boardService를 사용할려면 어떻게 될까? 바로 오류가 발생할것이다.
원인으로는 2가지를 생각할 수 있다. <br/>
1. boardService가 Spring Bean등록이 안됨
2. BoardController의 list메소드가 boardService를 사용하는데 도대체 이 boardService가 무엇이고 어디있는지 모른다.

우선 1번은 Spring Bean에 등록을 해주면된다. 바로 `@Service` Annotation이다. `@Service` Annotation을 확인해보면 위의 `@Controller`와 같이 @Component가 사용되서 Baen등록을 시켜준다는 것을 확인할 수 있다.
~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";

}
~~~

<br/>

2번은 `@Autowired`를 통해 생성된 Controller객체가 사용될때마다 boardService를 주입해주는(의존성 주입) `@Autowired`를 명시해야한다.

~~~java
@RestController
@AllArgsConstructor
@RequestMapping("/post")
public class BoardController {

    @Autowired
    private BoardService boardService;


    @LogExecutionTime
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
}


~~~

<br/>

### REFER TO
- <https://jhhan009.tistory.com/53>
- <https://melonicedlatte.com/2021/07/11/232800.html>


