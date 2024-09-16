---
title: 하모니 팀의 Rest Docs 와 MockMvc 를 활용한 API 문서 자동화 
date: "2024-08-17"
tags:
  - 하온
  - Spring
  - Rest Docs
previewImage: rest-docs.png
---

> 💡 현재 포스트는 harmony 팀 크루 [하온](https://github.com/msung99)이 작성했습니다.

## Rest Docs 도입

![alt text](./image.png)

현재 카테부 하모니 팀은 원활한 API 문서 자동화를 위해 Rest Docs 를 도입하였다. Rest Docs 는 자바 진영에서 가장 많이 사용되는 API 자동화 라이브러리이다. 왜 우리는 Rest Docs 를 도입하였는가?

- `(1)` **API 문서 작성을 자동화한다** : 당연하게도, 백엔드와 프론트엔드 개발자 사이의 협업시 가장 중요한 요소 중 하나가 바로 API 명세서이다. 개발해놓은 REST API 명세에 대한 문서화가 잘 되어있어야지 연동이 쉬워진다. Rest Docs 와 같은 API 문서화 툴은 이를 손쉽게 가능하도록 한다.

- `(2)` **문서화의 실수할 여지를 줄여준다** : 구글 독스, 노션등 사람이 직접 API 명세를 문서화할 수 있지만, 이는 사람으로 하여금 실수할 큰 여지를 만든다. 한 API 를 명세하는데에 있어서도 요청과 응답사이의 헤더, 바디, 쿠키, URI, 메소드 종류, 상태코드 값, 쿼리 파라미터 등 많은 요소들을 단 한치의 오차없이 사람이 명세하기란 힘들다. API 문서 자동화 툴을 사용한다면 (일반적으로) 실제 API 스팩과 달라지는 것 없이 오차없는 명세를 작성해준다.

- `(3)` **자동화를 통한 지속적인 명세 최신화** : 또한 요구사항 변경, 팀내 지속적인 변화로 인해 API 스팩을 변경해야하는 상황이 있다면 어떻게할까? 실제 API 내용이 변경된다면, 기존 API 명세 또한 변경되어야 한다. API 자동화 툴을 사용한다면 API 변경이 일어날 때 마다 명세 또한 빠르게 최신화할 수 있다. 노션, 위키등으로 사람이 직접 수정할 때와 달리, 코드 몇 줄만 바꾸는것이 끝이다. 자동화는 잦은 요구사항 변경, 유지.보수에 있어서 큰 장점을 보인다.


## 자바 진영에서 제공하는 API 문서 자동화 툴

Rest Docs 를 비롯하여 API 문서 작성을 도와하는 자동화 툴은 많이 개발되어 있다. 자바 진영에서 가장 많이 쓰이는 툴에는 **Swagger 와 Rest Docs** 이 2가지가 있다. 우리 하모니 팀은 이 중에 **Rest Docs 를 선택하게 되었다.** 또한 Rest Docs 의 **문서 자동화를 도와주는 보조 테스트 도구로 MockMvc, 문서로는 Asciidoc 을 선택하게 되었다.** 우리 팀이 왜 이런 문서화 툴을 선택하게 되었는지 근거를 정리해보겠다.

또한 실제로 Rest Docs 를 활용하여 API 문서 자동화 환경을 구축하는 간단한 환경을 실습해보도록 한다.

## Spring Rest Docs vs Swagger 

우리 팀은 Rest Docs 를 선택했지만, Swagger 또한 막강한 장점을 가지고 있다. 스프링부트 프로젝트에서 Swagger 를 사용하면 컨트롤러 메소드 상단에 간단히 어노테이션을 추가하고, API 명세하면 끝이다. 컨트롤러 계층에 간단히 어노테이션을 붙이고 API 명세를 자동화하는 방식은 직관적이고 명세하기 쉽다.

아래 코드는 이번 하모니 팀 프로젝트 진행전에, 다른 프로젝트에서 Swagger 로 명세했던 코드이다.  

~~~java
@ResponseBody
@PostMapping("/makeTeam")
@Operation(summary = "팀 생성", description = "jwt 를 HttpHeader에 넘겨주시고 생성해주세요. 누락된 정보가 있거나 20글자를 초과할 때 에러를 발생시켰습니다.")
public BaseResponse makeTeam(@RequestBody PostTeamReq postTeamReq) {
    try {
        int userIdxByJwt = jwtService.getUserIdx();
        mappingService.makeTeam(userIdxByJwt, postTeamReq);
        return new BaseResponse();
    } catch (BaseException baseException){
        return new BaseResponse(baseException.getStatus());
    }
}
~~~

간단히 어노테이션만 "틱" 붙혀서 API 문서를 자동으로 생성해주니, 아주 간단하고 편리하다. 하지만 Swagger 는 아래와 같은 단점이 존재한다.

### Swagger 를 왜 사용하지 않았는가?

> - `(1)` API 스팩에 변동이 일어나도, Swagger 는 이를 감지하고 API 명세를 수정해주지 않는다. **즉, 실제 API 스팩과 Swagger 기반 명세서 사이에 스팩이 일치함을 보장하지 않는다.**
> - `(2)` 프로덕션 코드, 즉 컨트롤러 계층에 API 명세에 관련한 코드를 장황하게 추가해줘야한다. 이 떄문에 모든 컨트롤러내의 프로덕션 코드에 가독성을 해친다.

특히 `(1)` 번의 치명적인 단점이 Swagger 를 선택하지 않은 결정적인 이유다. 당연하게도, API 명세는 API 스팩과 항상 일치함을 보장하여 일관성, 무결성을 보장해야한다. **필자가 생각하기에 API 자동화 툴을 사용하는 가장 큰 목적 중 하나가 문서화의 실수할 여지를 줄여주는 것이라 생각하는데, 스웨거는 이 장점을 잃어버리게 만든다.**

Swagger 는 아주 작은 미니 프로젝트, 단기간에 연동 및 릴리즈가 일어나야 하는 프로젝트 (해커톤 등) 에 적합하다. 다시말해, 프로젝트 규모가 커질수록 스웨거는 변동사항에 대해 개발자가 일일이 직접 체크하고 수정해줘야 하기 때문에, 우리 프로젝트에 적합하지 않다고 결론을 내렸다. 

## Spring Rest Docs (vs Swagger)

Spring Rest Docs 는 Swagger 의 단점을 보완한 강력한 API 자동화 툴이라고 생각한다. 어떤점에서 단점을 보완했는가?

![alt text](./image-1.png)

> - `(1)` 작성한 테스트 코드를 기반으로 API 명세가 일어난다. 
> - `(2)` 프로덕션 코드내의 컨트롤러 계층에 문서화를 위한 코드를 추가하지 않아도 된다.

`(1)` 이 안겨주는 장점은 매우 강력하다. **API 명세가 개발자가 작성한 테스트 코드를 기반으로 일어나기 떄문이다.** Junit 진영에서 작성한 테스트가 실행되고, 해당 테스트가 성공했을 때 테스트에 대한 AsciiDoc 스닛펫이 생성된다. (스닛펫이란 API 명세가 자동 생성되기 위한 일종의 리소스를 뜻한다.)

**테스트 코드를 기반으로 API 명세가 생성되니, 테스트가 성공하는 올바른 프로덕션 코드에 대해서만 API 명세가 제공된다.** 다시말해, 테스트가 실패하면 API 명세가 생성되지 않는다. 이 떄문에 Swagger 와 달리 **항상 프로뎍션 코드와 일치하는 API 명세를 제공하게 된다.**

단점을 굳이 뽑으라 함은, 아무래도 API 명세를 위한 테스트 작성이 필요하다는 점이다. 테스트 작성이 처음인 사람들에게 아무래도 Swagger 에 비해 러닝커브가 다소 높은것은 사실이다. 하지만, 개발자에게 테스트는 언제나 항상 중요하지 않겠는가? 🤔

### MockMvc vs RestAssured

Spring Rest Docs 를 활용하여 API 문서를 자동화하려면 테스트 코드가 필요하다. 테스트 코드를 작성시, 자바 진영에서 대표적으로 사용되는 툴이 `MockMvc` 와 `Rest Assured` 이다. 우리 팀은 `MockMvc` 를 사용하게 됐다. 왜 우리 서비스에 `MockMvc` 가 적합하다고 판단하여 이를 선택하게 됐는가?

### Rest Assured

Rest Assured 는 `@SpringBootTest` 와 함께 동작해야한다. `@SpringBootTest` 는 스프링 컨테이너내에 등록된 모든 애플리케이션 빈을 로드하여 빈을 주입하기 때문에, 테스트 수행 속도가 느리다. 하지만, Mocking 이 아닌 실제 객체(Bean) 을 로드하여 테스트 수행하기 때문에 테스트에 대한 신뢰도가 높다. 

정리하자면, **Rest Assured 를 활용한 방식은 @SpringBootTest 로 전 레이어 빈을 로드하기 때문에 테스트가 느리고, 비용이 많이든다.**

### MockMvc

반면 MockMvc 는 테스트 환경 구성에 따라 필요한 최소한의 빈만 로드할 수 있다. 이 떄문에 일반적으로 Rest Assured 를 활용한 방식보다 테스트 속도가 빠르다.

MockMvc 는 `@SpringBootTest` 와 함께 사용할 수도 있고, `@WebMvcTest` 와 함께 사용할 수도 있다. `@WebMvcTest` 는 `@SpringBootTest` 와 달리 프레젠테이션(컨트롤러) 계층의 객체(Bean) 들만 로드한다. 즉, 프레젠테이션 계층을 제외한 나머지 계층의 빈들은 모두 Mocking 한다. 이렇게 각 계층별로 독립적인 계층 하나만을 테스트 하는 기법을 `슬라이스 테스트(Slice Test)` 라고 한다.

### MockMvc 를 선택한 이유 🤔

우리 하모니 팀에서 처음 Rest Docs 를 도입했을 때는 Mocking 에 대한 거부감을 가지고 있었다. 실제 객체를 테스트 해야만 믿음직한 테스트라는 굳은 믿음이 있었기 때문에, `@SpringBootTest` 로 전 레이어의 빈들을 로드하여 테스트했다.

하지만 시간이 거듭될수록 API 문서를 위한 테스트에는 MockMvc 가 적합하다는 판단이 나왔다. 즉, `MockMvc` 와 `@WebMvcTest` 기반 `슬라이스 테스트(Slice Test)` 를 적용한 방식으로 마이그레이션했다. 아무래도 프레젠테이션 계층내에 API 명세 관련 테스트가 많이지다보니, 시간이 지날수록 테스트 속도가 저하되었다. `@SpringBootTest` 로 전 레이어 빈을 로드하는 기법은 이미 `Rest Assured` 기반 인수 테스트에서 진행되고 있었기 떄문에, API 명세를 위해 전 레이어를 또 다시 로드하여 테스트할 필요성이 점점 없어졌다. 

## AsciiDoc vs MarkDown

Spring Rest Docs 는 AsciiDoc 을 문서 번역을 위한 텍스트 프로세서로 활용한다. 쉽게말해, **Asciidoc 이란 .adoc 파일을 html 등의 형식으로 변환해주는 툴이다.** 마크다운 형식의 문서는 이미 대중적으로 사용되고 있는 문서화 포맷이지만, AsciiDoc 에서 제공하는 `include` 라는 강력한 기능이 있었기에 마크다운은 사용하지 않았다.

`include` 기능이란 다른 asciidoc 문서를 현재 asciidoc 문서로 가져와 명세에 포함시킬 수 있는 기능이다. 즉, 이곳저곳에 흩어진 문서를 한 곳에 모아서 제공하여, 문서화 내용을 한 눈에 파악할 수 있게한다.

## Rest Docs 기반 API 문서 자동화 환경 구축하기

백문이불여일타. 실제로 Rest Docs 를 활용하여 API 문서 자동화 환경을 구축하는 간단한 환경을 실습해보도록 한다.

### build.gradle 의존성 추가 및 설정

Rest Docs 도입을 위해 gradle 내에 설정이 필요하다. 이를 위한 방법을 학습해보도록 한다.

~~~java
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.3.2'
	id 'org.asciidoctor.jvm.convert' version '3.3.2' // (1)
}

configurations {
	asciidoctorExtensions // (2)
	compileOnly {
		extendsFrom annotationProcessor
	}
}

ext { 
	snippetsDir = file('build/generated-snippets') // (3)
}

dependencies {
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	asciidoctorExtensions 'org.springframework.restdocs:spring-restdocs-asciidoctor' // (4)
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc' // (5)
}

test {
	outputs.dir snippetsDir // (6)
}


asciidoctor { // (7)
	configurations 'asciidoctorExtensions' // (8)
	inputs.dir snippetsDir // (9)
	dependsOn test // (10)
	sources {
		include("**/index.adoc")
	}
	baseDirFollowsSourceFile()
}

tasks.named('build') {
	dependsOn asciidoctor
}
~~~

- `(1)` asciidoctor 를 위한 플러그인을 적용한다.
- `(2)` asciidoctor 를 확장하는 `asciidoctorExtensions` 에 대한 종속성 구성을 선언한다.
- `(3)` 생성되는 스닛펫들이 생성되는 경로를 설정한다. 이 실습의 경우 명시한바와 같이 `/build/generated-snippets` 에 스니펫이 생성될 것이다.
- `(4)` `asciidoctorExtensions` 에  Rest Docs 및 AsciiDoctor 에 대한 의존성을 추가한다. 이 의존성 주입을 통해 `/build/generated-snippets` 에 있는 `.adoc` 파일을 읽어들여 `.html` 파일을 만들어낼 수 있다.
- `(5)` MockMvc 를 사용하기 위한 의존성을 추가한다. 
- `(6)` 생성될 스닛펫들이 `snippetsDir` 경로에 생성되도록 `test` 테스크에 대한 상세 설정을 부여한다.
- `(7)` asciidoctor 테스크에 대한 세부 설정을 진행한다.
- `(8)` Asciidoctor 확장에 대한 설정을 한다.
- `(9)` 로드해올 스닛펫 경로를 `snippetsDir` 로 설정한다.
- `(10)` `test` 테스트가 수행된 후, `asciidoctor` 테스크가 수행되도록 설정한다. 즉, 테스트가 수행되고 난 뒤 asciidoctor 기반 문서화가 진행된다.
- `(11)` `build` 테스크가 `asciidoctor` 테스크에 의존하도록 설정한다. `./gradlew clean build` 명령 실행시 ascciidoctor 테스크가 수행된 뒤 build 가 수행됨으로, Rest Docs 명세서가 최신화됨을 보장한다.

### RestDocs, MockMvc 를 활용한 테스트 코드 작성

이어서 RestDocs, MockMvc 를 활용한 프레젠테이션 레이어내의 테스트 코드를 작성한다. 

~~~java
@AutoConfigureRestDocs // (1)
@WebMvcTest({  // (2)
        MemberController.class,
})
@ActiveProfiles("test") // (3)
public class MemberControllerTest {

    @Autowired // (4)
    private MockMvc mockvc;

    @MockBean // (5)
    private MemberService memberService;

    @DisplayName("사용자 본인의 회원 정보를 조회하고 상태코드 200을 리턴한다.")
    @Test
    void 본인의_회원_정보를_조회하고_상태코드_200을_리턴한다() throws Exception {
        // given
        given(jwtTokenProvider.getMemberId(anyString())).willReturn(1L);
        given(memberService.findById(anyLong())).willReturn(하온_응답());

        // when, then
        mockMvc.perform(get("/api/member/me") // (6)
                        .header("Authorization", "Bearer aaaaaa.bbbbbb.cccccc")
                        .accept(MediaType.APPLICATION_JSON)
                        .contentType(MediaType.APPLICATION_JSON)
                )
                .andDo(print())
                .andDo(document("member/me/success", // (7)
                        preprocessRequest(prettyPrint()),
                        preprocessResponse(prettyPrint()),
                        requestHeaders(
                                headerWithName("Authorization").description("엑세스 토큰")
                        ),
                        responseFields(
                                fieldWithPath("id").description("고유 id 값"),
                                fieldWithPath("profileImageUrl").description("프로필 이미지 경로"),
                                fieldWithPath("nickname").description("성별. 형식: MEN 또는 WOMEN"),
                                fieldWithPath("birthday").description("생년월일. 형식:yyyy-MM-dd"),
                                fieldWithPath("genderType").description("프로필 이미지 경로.")
                        )
                ))
                .andExpect(status().isOk());
    }
}
~~~

- `(1)` Rest Docs 를 위한 어노테이션을 추가하여, 스닛펫이 생성되도록 한다.
- `(2)` `@WebMvcTest` 를 통해 슬라이스 테스트를 진행한다. API 문서화를 위한 최소한의 빈만 로드하는 것으로, 위 실습의 경우 Member 컨트롤러에 대한 객체만을 로드했다.
- `(3)` 테스트 환경내에서 프로파일을 활성화하기 위한 설정을 진행한다.
- `(4)` MockMvc 오브젝트를 활용하기 위한 의존성을 `@Autowired` 로 주입받는다.
- `(5)` MemberController 가 의존하고 있는 Bean 객체인 MemberService 를 Mocking 한다.
- `(6)` mockMvc 로 API 를 호출한다.
- `(7)` 명세할 API 에 대한 상세 설정을 명시한다. 이때 `preprocessRequest`, `preprocessResponse` 를 활용하여 JSON 포맷을 더 깔끔하게 포맷팅하여 문서를 생성해준다.

### adoc 스닛펫 생성

앞서 작성한 테스트를 실행하고 성공했다면, build -> generated-snippets 내에 스닛펫이 생성되었을 것이다.

![alt text](./image-2.png)

### adoc 스닛펫을 API 명세로 표현

생성된 adoc 스닛펫들은 아직 한 눈에 파악하기 힘들다. 이곳저곳에 스닛펫들이 흩어져있을 뿐더러, `.adoc` 형식은 우리들에게 익숙하지 않아 명세를 읽기에 불편함을 느낄 수 있다. 이 스닛펫들을 한 곳에 모아두고, 그를 우리에게 익숙한 `.html` 형식으로 예쁘게 표현해보자.

우선 아래와 같이 `include` 를 통해 앞서 생성된 adoc 스닛펫을 문서에 끌어모을 수 있다. 이를위해, src -> docs -> asciidoc 내에 폴더를 생성하고, `member.adoc` 파일을 하나 생성한 뒤 아래처럼 asciidoc 문서를 작성해주도록 하자.

~~~java
== ⛳️ Member (회원) 마이페이지 + Header 조회
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2

=== 회원 정보 조회(마이페이지) API

==== HTTP Request

include::{snippets}/member/me/http-request.adoc[]

===== Request Header (Authorization)

include::{snippets}/member/me/request-headers.adoc[]

==== HTTP Response

include::{snippets}/member/me/http-response.adoc[]

===== Response Body

include::{snippets}/member/me/response-fields.adoc[]
~~~

### build 수행 및 html 문서 확인

`asciidoctor` 테스크를 실행하여 html 문서를 생성할 수 있다. 문서는 src -> docs -> asciidoc 경로에 생성한 `.adoc` 파일을 기반으로 생성된다. 앞서 build.gradle 설정시 build 테스크가 asciidoctor 테스크에 의존하도록 설정했으므로, 아래와 같이 빌드 실행시 asciidoctor 테스크가 수행되어 html 명세서 파일이 생성될 것이다.

~~~java
$ ./gradlew clean build
~~~

이후 스프링부트 애플리케이션 실행 후 `http://localhost:8080/docs/index.html` 로 접속하면 HTML 파일이 정상적으로 보이는 것을 확인할 수 있다.

![alt text](./image-3.png)