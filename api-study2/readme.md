
### TDD

---
---

비밀번호 암호화

``` kotlin
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder

@Service
class UserService(
    private val userRepository: UserRepository,
    private val redisTemplate: StringRedisTemplate
) {
    private val passwordEncoder = BCryptPasswordEncoder()

    fun registerUser(username: String, password: String): User {
        if (password.length < 8) {
            throw IllegalArgumentException("비밀번호는 8자리 이상 되어야 해요.")
        }

        //BCryptPasswordEncoder를 사용하여 비밀번호를 암호화합니다. registerUser 메서드 내에서 passwordEncoder.encode(password)를 호출하여 비밀번호를 암호화한 뒤, User 객체를 생성하고 저장합니다.
        val encodedPassword = passwordEncoder.encode(password)
        val user = User(username = username, password = encodedPassword)
        return userRepository.save(user)
    }

    // 다른 메서드들은 생략
}
```

- BCryptPasswordEncoder를 사용하여 비밀번호를 암호화하면, 원본 비밀번호 데이터 자체는 데이터베이스에 저장되지 않습니다. 대신 암호화된 해시 값만 저장됩니다. BCrypt는 일방향 해시 함수를 사용하므로, 암호화된 해시 값에서 원본 비밀번호를 복원할 수 없습니다.

### 추가 - 입력된 비밀번호와 데이터베이스에 저장된 해시 값을 비교하여 인증을 수행합니다
``` kotlin
fun validateUser(username: String, password: String): Boolean {
    val user = userRepository.findByUsername(username)
    return user?.let { passwordEncoder.matches(password, user.password) } ?: false
}
```


---
---

### 추가 - API 문서화 (Swagger 사용)

``` kotlin
// SwaggerConfig.kt
@Configuration
class SwaggerConfig {
    // 기존 코드

    @Bean
    fun api(): GroupedOpenApi {
        return GroupedOpenApi.builder()
            .group("user-api")
            .pathsToMatch("/users/**")
            .build()
    }
}
```
- SwaggerConfig에 api 메서드를 추가하여 /users/** 경로에 대한 API 문서를 생성합니다.

``` kotlin
// UserController.kt
@RestController
@RequestMapping("/users")
class UserController(private val userService: UserService) {

    @PostMapping("/register")
    @Operation(summary = "회원 가입 API", description = "새로운 사용자를 등록합니다.")
    fun register(
        @RequestParam username: String,
        @RequestParam password: String
    ): ResponseEntity<User> {
        return ResponseEntity.ok(userService.registerUser(username, password))
    }

    // 다른 메서드들은 생략
}
```
- UserController의 각 메서드에 @Operation 애노테이션을 사용하여 API 문서에 설명을 추가합니다.
- Swagger UI에서 /swagger-ui/index.html 경로를 통해 API 문서를 확인할 수 있습니다.


---
---



Spring Boot와 Kotlin으로 작성된 RESTful API 애플리케이션입니다. 
주요 기능은 사용자 관리(회원 가입, 로그인, 정보 수정, 삭제 등)와 관련된 API를 제공하고 있습니다.

기술 스택 및 환경은 다음과 같습니다:
- 언어: Kotlin
- 프레임워크: Spring Boot
- 데이터베이스: H2 (메모리 DB)
- 캐싱: Redis
- 테스트: JUnit5, Mockito
- 빌드 도구: Gradle

주요 파일 및 디렉토리 구조는 다음과 같습니다:
1. `main/kotlin/com/daelim/database/controller/UserController.kt`: 사용자 관련 API 엔드포인트를 처리하는 컨트롤러 클래스입니다.
2. `main/kotlin/com/daelim/database/service/UserService.kt`: 사용자 관련 비즈니스 로직을 처리하는 서비스 클래스입니다.
3. `main/kotlin/com/daelim/database/repository/UserRepository.kt`: 데이터베이스 연동을 위한 레포지토리 인터페이스입니다.
4. `main/kotlin/com/daelim/database/core/dto/UserUpdateDTO.kt`: 사용자 정보 업데이트를 위한 데이터 전송 객체(DTO) 클래스입니다.
5. `main/kotlin/com/daelim/database/core/entity/User.kt`: 사용자 엔티티 클래스입니다.
6. `main/kotlin/com/daelim/database/infra/CorsConfig.kt`: CORS 설정 클래스입니다.
7. `main/kotlin/com/daelim/database/infra/SwaggerConfig.kt`: Swagger 설정 클래스입니다.
8. `test/kotlin/com/daelim/database/large/UserControllerSpec.kt`: `UserController`에 대한 통합 테스트 클래스입니다.
9. `test/kotlin/com/daelim/database/medium/UserServiceSpec.kt`: `UserService`에 대한 단위 테스트 클래스입니다.
10. `test/kotlin/com/daelim/database/small/UserSpec.kt`: `User` 엔티티에 대한 단위 테스트 클래스입니다.

이 프로젝트에서는 TDD(Test-Driven Development) 기법을 사용하여 테스트 코드를 작성하고 있습니다. 테스트 코드는 `large`, `medium`, `small` 디렉토리로 분류되어 있습니다.

- `large` 디렉토리: 통합 테스트 코드를 포함하고 있으며, 실제 서버가 실행되는 환경에서 테스트를 수행합니다.
- `medium` 디렉토리: 단위 테스트 코드를 포함하고 있으며, 특정 클래스나 메서드의 동작을 검증합니다. 종속성을 가진 객체는 Mock 객체로 대체합니다.
- `small` 디렉토리: 단위 테스트 코드를 포함하고 있으며, 단일 클래스나 메서드의 동작을 검증합니다. 외부 종속성이 없는 순수 함수나 메서드를 테스트합니다.

테스트 코드는 Given-When-Then 패턴을 따르고 있으며, 각 테스트 케이스는 명확한 설명과 함께 작성되어 있습니다.





