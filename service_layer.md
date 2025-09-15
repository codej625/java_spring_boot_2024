# Service Layer

<br />
<br />

* Service 레이어란?

---

```
서비스 레이어(Service Layer)는 소프트웨어 아키텍처에서 비즈니스 로직을 처리하는 계층으로,
애플리케이션의 핵심 기능과 데이터 접근을 조정하는 역할을 한다.
```

<br />
<br />
<br />
<br />

1. 사용 예시

```
단일책임원칙과 트랜잭션 스크립트를 피해
서비스 레이어의 결합도를 낮춘 예시이다.
```

<br />

`entity`

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

@Entity
@Table(name = "users")
@Getter
@Setter
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
}
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

`Domain Service (비즈니스 로직 분리)`

```java
package com.example.demo.service;

import com.example.demo.entity.User;
import java.util.regex.Pattern;

public class UserDomainService {

    public void validateUserCreation(String username, String email) {
        validateUsername(username);
        validateEmail(email);
    }

    public void validateUserUpdate(String email, User existingUser) {
        if (!existingUser.getEmail().equals(email)) {
            validateEmail(email);
        }
    }

    private void validateUsername(String username) {
        if (username == null || username.trim().isEmpty()) {
            throw new IllegalArgumentException("Username cannot be empty");
        }
        if (username.length() < 3) {
            throw new IllegalArgumentException("Username must be at least 3 characters long");
        }
    }

    private void validateEmail(String email) {
        String emailRegex = "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$";
        if (email == null || !Pattern.matches(emailRegex, email)) {
            throw new IllegalArgumentException("Invalid email format");
        }
    }

    public void notifyUserDeactivation(User user) {
        // 비활성화 시 알림 전송 (여기서는 콘솔 출력으로 대체)
        System.out.println("Notification: User " + user.getUsername() + " has been deactivated.");
    }
}
```


<br />

`service`

```java
package com.example.demo.service;

import com.example.demo.dto.UserResponse;
import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

@Service
public class UserService {

    private final UserRepository userRepository;
    private final UserDomainService userDomainService;

    public UserService(UserRepository userRepository, UserDomainService userDomainService) {
        this.userRepository = userRepository;
        this.userDomainService = userDomainService;
    }

    @Transactional
    public UserResponse createUser(String username, String email) {
        userDomainService.validateUserCreation(username, email);
        if (userRepository.existsByEmail(email)) {
            throw new IllegalArgumentException("Email already exists");
        }
        User user = new User();
        user.setUsername(username);
        user.setEmail(email);
        User savedUser = userRepository.save(user);
        return UserResponse.fromEntity(savedUser);
    }

    @Transactional(readOnly = true)
    public Optional<UserResponse> getUserById(Long id) {
        return userRepository.findById(id)
                .map(UserResponse::fromEntity);
    }

    @Transactional(readOnly = true)
    public Optional<UserResponse> getUserByEmail(String email) {
        userDomainService.validateEmail(email);
        return userRepository.findByEmail(email)
                .map(UserResponse::fromEntity);
    }

    @Transactional(readOnly = true)
    public List<UserResponse> getAllActiveUsers() {
        return userRepository.findAll().stream()
                .filter(User::isActive)
                .map(UserResponse::fromEntity)
                .collect(Collectors.toList());
    }

    @Transactional
    public UserResponse updateUser(Long id, String username, String email) {
        Optional<User> existingUser = userRepository.findById(id);
        if (existingUser.isEmpty()) {
            throw new IllegalArgumentException("User not found");
        }
        User user = existingUser.get();
        userDomainService.validateUserUpdate(email, user);
        if (!user.getEmail().equals(email) && userRepository.existsByEmail(email)) {
            throw new IllegalArgumentException("Email already exists");
        }
        user.setUsername(username);
        user.setEmail(email);
        User updatedUser = userRepository.save(user);
        return UserResponse.fromEntity(updatedUser);
    }

    @Transactional
    public void deactivateUser(Long id) {
        Optional<User> user = userRepository.findById(id);
        if (user.isEmpty()) {
            throw new IllegalArgumentException("User not found");
        }
        User existingUser = user.get();
        existingUser.setIsActive(false);
        userRepository.save(existingUser);
        userDomainService.notifyUserDeactivation(existingUser);
    }
}
```

<br />

`controller`

```java
package com.example.demo.controller;

import com.example.demo.dto.UserRequest;
import com.example.demo.dto.UserResponse;
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
    public UserResponse createUser(@RequestBody UserRequest request) {
        return userService.createUser(request.getUsername(), request.getEmail());
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUserById(@PathVariable Long id) {
        Optional<UserResponse> user = userService.getUserById(id);
        return user.map(ResponseEntity::ok)
                   .orElseGet(() -> ResponseEntity.notFound().build());
    }

    @GetMapping
    public List<UserResponse> getAllActiveUsers() {
        return userService.getAllActiveUsers();
    }

    @PutMapping("/{id}")
    public UserResponse updateUser(@PathVariable Long id, @RequestBody UserRequest request) {
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
// Request

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


// Response

@Getter
@Setter
public class UserResponse {
    private Long id;
    private String username;
    private String email;
    private boolean isActive;

    public UserResponse(Long id, String username, String email, boolean isActive) {
        this.id = id;
        this.username = username;
        this.email = email;
        this.isActive = isActive;
    }

    // Entity -> DTO 변환 메서드
    public static UserResponse fromEntity(com.example.demo.entity.User user) {
        return new UserResponse(user.getId(), user.getUsername(), user.getEmail(), user.isActive());
    }
}
```

<br />
<br />

```
개발 과정에서 항상 완벽히 최적화된 코드를 작성하기는 어렵지만,
단일 책임 원칙(SRP)을 준수하고,
결합도를 낮추며 응집도를 높이는 설계를 통해 모듈화된 구조를 만들어내고,
추상화를 활용해 재사용성을 극대화하는 우아한 코드를 지향하자.
```
