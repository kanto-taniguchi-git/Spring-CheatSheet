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

## Restコントローラ例

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService svc;
    public UserController(UserService svc) { this.svc = svc; }

    @GetMapping
    public List<User> getAll() { return svc.findAll(); }

    @PostMapping
    public User create(@Valid @RequestBody User u) {
        return svc.create(u);
    }
}
```

## JPAエンティティ(データベースのテーブルと対応するJavaクラス)/リポジトリ例

```java
@Entity
public class User {
  @Id @GeneratedValue
  private Long id;
  private String name;
  // getter/setter
}

public interface UserRepository extends JpaRepository<User, Long> {}
```

## テスト

- 依存: spring-boot-starter-test

- 基本: @SpringBootTest, @WebMvcTest, @DataJpaTest

