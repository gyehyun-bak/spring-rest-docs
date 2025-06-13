## Spring REST Docs
Spring Boot 3.5 + Gradle + Java 21 환경에서 Spring REST Docs가 동작하도록 설정해둔 리포지토리입니다.

https://start.spring.io/ 에서 Spring Web과 Spring REST Docs를 포함한 상태의 프로젝트에서 최소한의 설정 추가만 포함하도록 했습니다.

문제점이나 개선안이 있다면 이슈로 제안해주세요.

## 사용 방법

1. 컨트롤러에 API를 정의하고 해당하는 테스트를 작성해 실행합니다.
2. 테스트를 실행하면 테스트에 명시한 스펙을 포함한 .adoc 스니펫이 자동 생성됩니다.
  - 경로: `/build/generated-snippets/{API_이름}`
3. `/src/docs/asciidoc/index.adoc`에 명세하고자 하는 API와 스니펫을 추가합니다.
4. Gradle build를 하면 API 명세가 포함된 정적 페이지가 만들어집니다.
  - 경로:  `/build/docs/asciidoc/index.html`
5. 서버 API 경로의 `/docs/index.html`로 접근할 수 있습니다.
  - 예: `http://localhost:8080/docs/index.html`

### 주의

- `index.adoc`의 변경 내용을 반영하기 위해 `.\gradlew clean build` 해주세요.

### 컨트롤러 작성

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }
}
```

### 테스트 작성

```java
@AutoConfigureMockMvc
@AutoConfigureRestDocs(outputDir = "build/generated-snippets")
@SpringBootTest
class HelloControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void hello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andDo(print())
                .andDo(document("hello"));
    }
}
```

## build.gradle

```groovy
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.5.0'
	id 'io.spring.dependency-management' version '1.1.7'
	id 'org.asciidoctor.jvm.convert' version '3.3.2'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
	toolchain {
		languageVersion = JavaLanguageVersion.of(21)
	}
}

repositories {
	mavenCentral()
}

ext {
	set('snippetsDir', file("build/generated-snippets"))
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
	outputs.dir snippetsDir
	useJUnitPlatform()
}

tasks.named('asciidoctor') {
	inputs.dir snippetsDir
	attributes 'snippets': snippetsDir // index.adoc 내 {snippets} 위치 명시
	dependsOn test
}

// Asciidoctor로 생성한 HTML 문서를 static/docs로 복사 (내장 톰캣에서 제공되게 함)
tasks.register('copyDocs', Copy) {
	dependsOn asciidoctor
	from asciidoctor.outputDir // asciidoctor 결과물 경로
	into "src/main/resources/static/docs" // 정적 리소스로 이동시켜 웹 접근 가능하게 함
}

// build 시 자동으로 문서 생성 및 복사까지 수행되게 설정
tasks.named('build') {
	dependsOn copyDocs
}
```

## /src/docs/asciidoc/index.adoc

```adoc
= API 문서
:toc: left
:toclevels: 2

== Hello API

=== /hello

.request
include::{snippets}/hello/http-request.adoc[]

.response
include::{snippets}/hello/http-response.adoc[]
```

### 동작 화면

<img width="1119" alt="image" src="https://github.com/user-attachments/assets/355a12a7-0b7a-49ec-a956-23d847f94099" />
