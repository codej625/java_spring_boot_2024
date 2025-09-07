# Data Access Object

<br />
<br />

* DAO

---

```
데이터베이스와의 상호작용을 추상화하고 데이터베이스에 접근하는 데 사용된다.
DAO는 주로 CRUD(Create, Read, Update, Delete) 연산을 수행하는 메서드들을 포함한다.
```

<br />
<br />
<br />
<br />

1. 패키지 분류

<br />

`Mybatis를 사용하는 프로젝트`

```
dao // 패키지 이름
dao/model // entity에 해당
dao/mapper // DB와 상호작용을 하는 객체

* model / mapper 명명규칙 예시
UserDAO / UserMapper(DAO)
```

<br />

`JPA를 사용하는 프로젝트`

```
domain // 패키지 이름
domain/entity // entity
domain/repository // DB와 상호작용을 하는 객체

* entity / repository 명명규칙 예시
User / UserRepository(DAO)
```

<br />
<br />
<br />

2. 스프링에서 DAO 사용 예시

<br />

`entity`

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Getter;

import java.util.regex.Pattern;

@Entity
@Table(name = "users")
@Getter
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username", nullable = false)
    private String username;

    @Column(name = "email", unique = true, nullable = false)
    private String email;

    @Column(name = "is_active", nullable = false)
    private boolean isActive = true;

    // JPA용 기본 생성자
    protected User() {}

    public User(String username, String email) {
        validateUsername(username);
        validateEmail(email);
        this.username = username;
        this.email = email;
    }

    // 비즈니스 로직: 사용자 이름 유효성 검사
    private void validateUsername(String username) {
        if (username == null || username.trim().isEmpty()) {
            throw new IllegalArgumentException("Username cannot be empty");
        }
        if (username.length() < 3) {
            throw new IllegalArgumentException("Username must be at least 3 characters long");
        }
    }

    // 비즈니스 로직: 이메일 유효성 검사
    private void validateEmail(String email) {
        String emailRegex = "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$";
        if (email == null || !Pattern.matches(emailRegex, email)) {
            throw new IllegalArgumentException("Invalid email format");
        }
    }

    // 비즈니스 로직: 사용자 비활성화
    public void deactivate() {
        this.isActive = false;
    }

    // 비즈니스 로직: 사용자 정보 업데이트
    public void update(String username, String email) {
        validateUsername(username);
        validateEmail(email);
        this.username = username;
        this.email = email;
    }
}
```

```
User Entity는 데이터베이스의 users 테이블과 매핑된다.

Entity는 단일 객체를 나타내므로 Users 대신 User 사용한다.
```

<br />

`repository (DAO)`

```java
package com.example.demo.repository;

import com.example.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
}
```

<br />

`service`

```java
package com.example.demo.service;

import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // 사용자 생성 -> 이메일 중복 체크 및 비즈니스 규칙 적용
    @Transactional
    public User createUser(String username, String email) {
        if (userRepository.existsByEmail(email)) {
            throw new IllegalArgumentException("Email already exists");
        }
        User user = new User(username, email);
        return userRepository.save(user);
    }

    // ID로 사용자 조회
    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }

    // 이메일로 사용자 조회
    public Optional<User> getUserByEmail(String email) {
        return userRepository.findByEmail(email);
    }

    // 모든 활성 사용자 조회
    @Transactional(readOnly = true)
    public List<User> getAllActiveUsers() {
        return userRepository.findAll().stream()
                .filter(User::isActive)
                .toList();
    }

    // 사용자 업데이트 -> 도메인 로직 호출
    @Transactional
    public User updateUser(Long id, String username, String email) {
        Optional<User> existingUser = userRepository.findById(id);
        if (existingUser.isEmpty()) {
            throw new IllegalArgumentException("User not found");
        }
        if (!existingUser.get().getEmail().equals(email) && userRepository.existsByEmail(email)) {
            throw new IllegalArgumentException("Email already exists");
        }
        User user = existingUser.get();
        user.update(username, email); // Entity의 도메인 로직 호출
        return userRepository.save(user);
    }

    // 사용자 비활성화
    @Transactional
    public void deactivateUser(Long id) {
        Optional<User> user = userRepository.findById(id);
        if (user.isEmpty()) {
            throw new IllegalArgumentException("User not found");
        }
        user.get().deactivate(); // Entity의 도메인 로직 호출
        userRepository.save(user.get());
    }
}
```

<br />

`controller`

```java
package com.example.demo.controller;

import com.example.demo.entity.User;
import com.example.demo.service.UserService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public User createUser(@RequestBody UserRequest request) {
        return userService.createUser(request.getUsername(), request.getEmail());
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        Optional<User> user = userService.getUserById(id);
        return user.map(ResponseEntity::ok)
                   .orElseGet(() -> ResponseEntity.notFound().build());
    }

    @GetMapping
    public List<User> getAllActiveUsers() {
        return userService.getAllActiveUsers();
    }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody UserRequest request) {
        return userService.updateUser(id, request.getUsername(), request.getEmail());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deactivateUser(@PathVariable Long id) {
        userService.deactivateUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

<br />

`DTO`

```java
class UserRequest {
    private String username;
    private String email;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

<br />

`API 테스트`

- POST /api/users: {"username": "john", "email": "john@example.com"}

- GET /api/users: 활성 사용자 목록 조회

- PUT /api/users/1: {"username": "john_updated", "email": "john_updated@example.com"}

- DELETE /api/users/1: 사용자 비활성화

<br />
<br />

```
마치며, 위에선 DAO의 사용 예시를 간단히 보여주기 위해 사용한 코드이다.

실제로 단일책임원칙과 트랜잭션 스크립트로 인한 서비스 레이어의 결합도를 높이는 코드를 사용하지 말기를 바란다.
```
