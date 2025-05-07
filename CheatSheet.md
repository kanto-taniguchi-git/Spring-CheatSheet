# Spring チートシート

## 🚀 プロジェクト作成

```bash
# Spring Initializr を使う場合
curl https://start.spring.io/starter.zip \
  -d dependencies=web,jdbc,validation,lombok \
  -d language=java -d bootVersion=3.4.5 \
  -d javaVersion=21 \
  -d groupId=com.example -d artifactId=demo \
  -o demo.zip
```

■ 解説

- `-d bootVersion=3.4.5`は、Spring Bootのバージョンを指定するオプションです。
- `-d javaVersion=21`は、JDKのバージョンを指定するオプションです。ローカルPCにインストールされているJDKに依存する。
- `-d dependencies=web,jdbc,validation,lombok`は、必要な依存関係を指定するオプションです。
- `-d groupId=com.example -d artifactId=demo`は、プロジェクトのグループIDとアーティファクトIDを指定するオプションです。

## 📦 依存関係一覧

| 依存関係 | 役割 | 入れていないと |
|---|---|---|
| Spring Boot DevTools | ・ホットリロードでソース変更を即時反映<br>・キャッシュ無効化、自動再起動 | ・手動で再起動が必要<br>・テンプレートや設定が更新されないことがある |
| Lombok | ・`@Getter`、`@Setter`、`@Data`などでボイラープレートコードを自動生成<br>・コード量とヒューマンエラーを削減 | ・Getter/Setter/ToString/EqualsAndHashCode/Builder/NoArgsConstructor/AllArgsConstructorを手動で書く必要がある |
| Validation | ・`@NotNull`、`@Size`等で入力を自動バリデーション<br>・エラーレスポンスを簡単に生成 | ・手動でバリデーションを実装する必要がある |
| JDBC API | ・`JdbcTemplate`などでSQL実行やマッピングを簡素化 | ・接続／クエリ／例外処理／リソース解放などを毎回手動で実装する必要がある |
| Spring Data JDBC | ・リポジトリインターフェースによるCRUDの自動実装<br>・軽量なデータアクセスを提供 | ・毎回DAOクラスを実装する必要がある |
| Oracle Driver | ・Oracle DBに接続するためのJDBCドライバ | ・`java.lang.ClassNotFoundException: oracle.jdbc.OracleDriver` などで接続不可 |
| Thymeleaf | ・HTMLテンプレートに動的データを埋め込むテンプレートエンジン | ・JSP等の別テンプレートエンジンを使う必要あり |
| Spring Web | ・`@Controller` / `@RestController` によるMVCやREST API機能を提供 | ・WebアプリやAPI機能が使えず、コントローラが動作しない |

## 主要アノテーション

|アノテーション|用途|
|---|---|
|@SpringBootApplication|Spring Bootアプリケーションのエントリポイントを示す|
|@RestController|REST APIのコントローラーを示す|
|@GetMapping|HTTP GETリクエストを処理するメソッドを示す|
|@PostMapping|HTTP POSTリクエストを処理するメソッドを示す|
|@Service|ビジネスロジックを持つクラスを示す|
|@Repository|データアクセスを行うクラスを示す|
|@Entity, @id|JPAエンティティ|
|@Autowired|依存性注入を行う|
|@Valid|バリデーションを行う|
|@Value("${property}")|プロパティ値の注入|
|@Configuration, @Bean|独自のBeanを定義|

## 基本的なフォルダ構成

```plaintext
demo/                           ← プロジェクトルート（artifactId）
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/demo/
│   │   │        ├── DemoApplication.java       ← 起動クラス（mainメソッド）
│   │   │        ├── controller/                ← Web API（@RestController）を置く
│   │   │        │    └── UserController.java
│   │   │        ├── service/                   ← ビジネスロジック層
│   │   │        │    └── UserService.java
│   │   │        ├── repository/                ← データアクセス層（リポジトリ）
│   │   │        │    └── UserRepository.java
│   │   │        └── model/                     ← エンティティ（@Entity）を置く
│   │   │             └── User.java
│   │   └── resources/
│   │        ├── application.properties         ← 設定ファイル
│   │        ├── static/                        ← 静的ファイル（HTML, CSS, JS）
│   │        └── templates/                     ← テンプレートファイル（Thymeleafなど）
│
└── test/                                       ← テストコード用
    └── java/com/example/demo/
         └── UserControllerTest.java など
```

## DIとコンポーネントスキャン

- DI（依存性注入）: @Autowired やコンストラクタで依存オブジェクトを注入

- コンポーネントスキャン: @SpringBootApplication 配下パッケージを自動スキャン

## Controller/層：Web APIの窓口

```java
@RestController
@RestMapping("/api/users")
public class UserController {
  private final UserService userService;

  // コンストラクタインジェクション
  public UserController(UserService userService) {
    this.userService = userService;
  }

  // 全ユーザ取得
  @GetMapping
  public List<User> getAll() {
    return userService.findAll();
  }

  // IDで取得
  @GetMapping("/{id}")
  public ResponseEntity<User> getById(@PathVariable Long id) {
    return userService.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
  }

    // 新規作成
    @PostMapping
    public User create(@Valid @RequestBody User user) {
        return userService.create(user);
    }

    // 更新
    @PutMapping("/{id}")
    public ResponseEntity<User> update(@PathVariable Long id, @Valid @RequestBody User user) {
        return userService.update(id, user)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    // 削除
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        if (userService.delete(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}
```

## service/ 層：ビジネスロジック

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository repo) {
        this.userRepository = repo;
    }

    public List<User> findAll() {
        return userRepository.findAll();
    }

    public Optional<User> findById(Long id) {
        return userRepository.findById(id);
    }

    public User create(User user) {
        // 例：重複チェックなど
        if (userRepository.existsByEmail(user.getEmail())) {
            throw new IllegalArgumentException("Email already in use");
        }
        return userRepository.save(user);
    }

    public Optional<User> update(Long id, User updated) {
        return userRepository.findById(id).map(user -> {
            user.setName(updated.getName());
            user.setEmail(updated.getEmail());
            return userRepository.save(user);
        });
    }

    public boolean delete(Long id) {
        if (!userRepository.existsById(id)) {
            return false;
        }
        userRepository.deleteById(id);
        return true;
    }
}

```

## 3. repository/ 層：データアクセス

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // 追加クエリメソッドの例
    boolean existsByEmail(String email);
    List<User> findByNameContaining(String keyword);
}
```

## 4. model/ 層：エンティティ/ドメインモデル

```java

@Entity
@Table(name = "users")
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true)
    @Email
    private String email;

    // Lombok を使う場合は @Data や @Getter @Setter などを付与
    public User() {}
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    // getter / setter...
}
```

## 5. resource/：設定とテンプレート

- `application.properties`：アプリケーションの設定ファイル

```properties
# ポート設定
server.port=8080

# Datasource
spring.datasource.url=jdbc:oracle:thin:@//localhost:1521/XEPDB1
spring.datasource.username=USER
spring.datasource.password=PASS

# JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

- `templates/users.html`：Thymeleaf テンプレート

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head><title>ユーザー一覧</title></head>
<body>
  <h1>ユーザー一覧</h1>
  <ul>
    <li th:each="u : ${users}">
      <span th:text="${u.name}">Name</span> —
      <span th:text="${u.email}">Email</span>
    </li>
  </ul>
</body>
</html>
```

## 6. test/ 層：単体テストの例

- UserControllerTest.java

```java
@WebMvcTest(UserController.class)
public class UserControllerTest {
  @Autowired private MockMvc mvc;
  @MockBean private UserService svc;

  @Test
  void getAll_ReturnsList() throws Exception {
    given(svc.findAll()).willReturn(List.of(new User("Taro", "taro@example.com")));
    mvc.perform(get("/api/users"))
      .andExpect(status().isOk())
      .andExpect(jsonPath("$[0].name").value("Taro"));
  }
}
```

- UserServiceTest.java

```java
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {
  @Mock private UserRepository repo;
  @InjectMocks private UserService svc;

  @Test
  void create_DuplicateEmail_Throws() {
    User u = new User("Taro", "taro@example.com");
    when(repo.existsByEmail(u.getEmail())).thenReturn(true);
    assertThrows(IllegalArgumentException.class, () -> svc.create(u));
  }
}
```