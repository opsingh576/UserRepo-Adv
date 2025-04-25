# UserRepo-Adv
UserService for adviser

package com.wsadvising.userservice;

// --- User Entity ---
import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;
    private String password;
    private String address;
    private String phone;

    // Getters and Setters
}

// --- User DTO ---
public class UserDTO {
    private String name;
    private String email;
    private String password;
    private String address;
    private String phone;
    // Getters and Setters
}

// --- User Repository ---
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
    boolean existsByEmail(String email);
    User findByEmail(String email);
}

// --- User Service ---
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public User registerUser(UserDTO dto) {
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new RuntimeException("Email already registered");
        }
        User user = new User();
        user.setName(dto.getName());
        user.setEmail(dto.getEmail());
        user.setPassword(dto.getPassword());
        user.setAddress(dto.getAddress());
        user.setPhone(dto.getPhone());
        return userRepository.save(user);
    }

    public User loginUser(String email, String password) {
        User user = userRepository.findByEmail(email);
        if (user == null || !user.getPassword().equals(password)) {
            throw new RuntimeException("Invalid credentials");
        }
        return user;
    }
}

// --- User Controller ---
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping("/register")
    public User register(@RequestBody UserDTO dto) {
        return userService.registerUser(dto);
    }

    @PostMapping("/login")
    public User login(@RequestParam String email, @RequestParam String password) {
        return userService.loginUser(email, password);
    }
}

// --- Unit Test for UserService ---
import org.junit.jupiter.api.Test;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

public class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    public UserServiceTest() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testRegisterUser_Success() {
        UserDTO dto = new UserDTO();
        dto.setEmail("test@example.com");
        dto.setName("Test");
        dto.setPassword("pass");

        when(userRepository.existsByEmail(dto.getEmail())).thenReturn(false);
        when(userRepository.save(any(User.class))).thenAnswer(i -> i.getArguments()[0]);

        User savedUser = userService.registerUser(dto);
        assertEquals(dto.getEmail(), savedUser.getEmail());
    }

    @Test
    public void testLoginUser_Success() {
        User user = new User();
        user.setEmail("test@example.com");
        user.setPassword("pass");

        when(userRepository.findByEmail("test@example.com")).thenReturn(user);

        User loggedIn = userService.loginUser("test@example.com", "pass");
        assertNotNull(loggedIn);
    }

    @Test
    public void testLoginUser_InvalidPassword() {
        User user = new User();
        user.setEmail("test@example.com");
        user.setPassword("pass");

        when(userRepository.findByEmail("test@example.com")).thenReturn(user);

        assertThrows(RuntimeException.class, () -> {
            userService.loginUser("test@example.com", "wrongpass");
        });
    }
} 

+---------------------+        +-------------------------+         +---------------------+
|     Client (UI)     |<----->|  API Gateway / Router   |<------->| Authentication Svc  |
+---------------------+        +-------------------------+         +---------------------+
                                             |                             
                  -----------------------------------------------------------------------
                 |             |                  |                  |                 |
        +----------------+ +----------------+ +----------------+ +----------------+ +----------------+
        | User Service   | | Advisor Service| | Booking Service| | Search Service | | Notification Svc|
        +----------------+ +----------------+ +----------------+ +----------------+ +----------------+
                 |                    |                  |                |                |
                 |                    |                  |                |                |
         +----------------+  +----------------+  +----------------+  +----------------+  +----------------+
         |  Email Service |  |  Schedule Svc   |  |  Payment Svc   |  | Billing Svc    |  |  Audit Logger  |
         +----------------+  +----------------+  +----------------+  +----------------+  +----------------+

                       ⇩ All Services Communicate via: REST API + JSON + Message Queue (Optional for async ops)
                       ⇩ Data Persistence via respective DBs (MySQL/PostgreSQL)



