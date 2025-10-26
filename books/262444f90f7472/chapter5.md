---
title: "クラス設計"
---

## リーダブルコード問題

### 問題1: データクラスの適切な使用
以下の通常のクラスをデータクラスに変更し、より読みやすくしてください。

```kotlin
// 改善前
class User {
    private var name: String
    private var age: Int
    private var email: String
    
    constructor(name: String, age: Int, email: String) {
        this.name = name
        this.age = age
        this.email = email
    }
    
    fun getName(): String = name
    fun getAge(): Int = age
    fun getEmail(): String = email
    
    fun setName(name: String) { this.name = name }
    fun setAge(age: Int) { this.age = age }
    fun setEmail(email: String) { this.email = email }
    
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other !is User) return false
        return name == other.name && age == other.age && email == other.email
    }
    
    override fun hashCode(): Int {
        var result = name.hashCode()
        result = 31 * result + age
        result = 31 * result + email.hashCode()
        return result
    }
    
    override fun toString(): String {
        return "User(name='$name', age=$age, email='$email')"
    }
}
```

**回答:**
```kotlin
// 改善後
data class User(
    val name: String,
    val age: Int,
    val email: String
)

// 必要に応じてバリデーションを追加
data class User(
    val name: String,
    val age: Int,
    val email: String
) {
    init {
        require(name.isNotBlank()) { "Name cannot be blank" }
        require(age >= 0) { "Age must be non-negative" }
        require(email.contains("@")) { "Email must contain @" }
    }
}
```

**理由:**
- データクラスにより自動的にequals、hashCode、toString、copyが生成される
- コード量が大幅に削減され、保守性が向上
- 不変性（val）により、データの整合性が保たれる

### 問題2: シールドクラスの活用
以下の多態的な処理をシールドクラスを使って型安全にしてください。

```kotlin
// 改善前
abstract class Result
class Success(val data: String) : Result()
class Error(val message: String) : Result()
class Loading : Result()

fun handleResult(result: Result): String {
    return when {
        result is Success -> "Data: ${result.data}"
        result is Error -> "Error: ${result.message}"
        result is Loading -> "Loading..."
        else -> "Unknown result type"
    }
}
```

**回答:**
```kotlin
// 改善後
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
    object Loading : Result()
}

fun handleResult(result: Result): String = when (result) {
    is Result.Success -> "Data: ${result.data}"
    is Result.Error -> "Error: ${result.message}"
    is Result.Loading -> "Loading..."
}
```

**理由:**
- シールドクラスにより、すべてのサブクラスが同じファイル内で定義される
- when式でexhaustive（網羅的）なパターンマッチングが保証される
- else句が不要になり、コンパイル時に型安全性が確保される

### 問題3: ビルダーパターンの実装
以下の複雑なオブジェクト作成をビルダーパターンで改善してください。

```kotlin
// 改善前
class DatabaseConfig(
    val host: String,
    val port: Int,
    val username: String,
    val password: String,
    val databaseName: String,
    val maxConnections: Int,
    val timeout: Long,
    val sslEnabled: Boolean
) {
    constructor(host: String, port: Int, username: String, password: String) : this(
        host, port, username, password, "default_db", 10, 30000, false
    )
    
    constructor(host: String, port: Int, username: String, password: String, databaseName: String) : this(
        host, port, username, password, databaseName, 10, 30000, false
    )
}
```

**回答:**
```kotlin
// 改善後
data class DatabaseConfig(
    val host: String,
    val port: Int,
    val username: String,
    val password: String,
    val databaseName: String = "default_db",
    val maxConnections: Int = 10,
    val timeout: Long = 30000,
    val sslEnabled: Boolean = false
) {
    class Builder {
        private var host: String = ""
        private var port: Int = 0
        private var username: String = ""
        private var password: String = ""
        private var databaseName: String = "default_db"
        private var maxConnections: Int = 10
        private var timeout: Long = 30000
        private var sslEnabled: Boolean = false
        
        fun host(host: String) = apply { this.host = host }
        fun port(port: Int) = apply { this.port = port }
        fun username(username: String) = apply { this.username = username }
        fun password(password: String) = apply { this.password = password }
        fun databaseName(databaseName: String) = apply { this.databaseName = databaseName }
        fun maxConnections(maxConnections: Int) = apply { this.maxConnections = maxConnections }
        fun timeout(timeout: Long) = apply { this.timeout = timeout }
        fun sslEnabled(sslEnabled: Boolean) = apply { this.sslEnabled = sslEnabled }
        
        fun build() = DatabaseConfig(host, port, username, password, databaseName, maxConnections, timeout, sslEnabled)
    }
    
    companion object {
        fun builder() = Builder()
    }
}

// 使用例
val config = DatabaseConfig.builder()
    .host("localhost")
    .port(5432)
    .username("admin")
    .password("secret")
    .databaseName("myapp")
    .maxConnections(20)
    .build()
```

**理由:**
- デフォルト引数により必須パラメータとオプションパラメータを明確に分離
- ビルダーパターンにより、オブジェクト作成時の可読性が向上
- apply関数を使用してメソッドチェーンを実現

### 問題4: インターフェース分離の原則
以下の大きなインターフェースを小さなインターフェースに分割してください。

```kotlin
// 改善前
interface UserService {
    fun createUser(name: String, email: String): User
    fun updateUser(id: Long, name: String, email: String): User
    fun deleteUser(id: Long): Boolean
    fun findUserById(id: Long): User?
    fun findUsersByName(name: String): List<User>
    fun sendEmail(userId: Long, subject: String, body: String)
    fun sendSms(userId: Long, message: String)
    fun generateReport(userId: Long): String
    fun exportUserData(userId: Long): ByteArray
}
```

**回答:**
```kotlin
// 改善後
interface UserRepository {
    fun createUser(name: String, email: String): User
    fun updateUser(id: Long, name: String, email: String): User
    fun deleteUser(id: Long): Boolean
    fun findUserById(id: Long): User?
    fun findUsersByName(name: String): List<User>
}

interface NotificationService {
    fun sendEmail(userId: Long, subject: String, body: String)
    fun sendSms(userId: Long, message: String)
}

interface ReportService {
    fun generateReport(userId: Long): String
    fun exportUserData(userId: Long): ByteArray
}

// 必要に応じて複数のインターフェースを実装
class UserServiceImpl(
    private val userRepository: UserRepository,
    private val notificationService: NotificationService,
    private val reportService: ReportService
) : UserRepository, NotificationService, ReportService {
    
    override fun createUser(name: String, email: String): User = 
        userRepository.createUser(name, email)
    
    override fun sendEmail(userId: Long, subject: String, body: String) = 
        notificationService.sendEmail(userId, subject, body)
    
    // 他のメソッドも同様に委譲
}
```

**理由:**
- インターフェース分離の原則に従い、クライアントが必要とする機能のみを提供
- 各インターフェースが単一の責任を持つ
- テスト時に必要な部分のみをモック可能

### 問題5: ファクトリーパターンの実装
以下の複雑なオブジェクト作成をファクトリーパターンで改善してください。

```kotlin
// 改善前
fun createDatabaseConnection(type: String, host: String, port: Int, username: String, password: String): DatabaseConnection {
    when (type) {
        "mysql" -> {
            val config = MySQLConfig(host, port, username, password)
            return MySQLConnection(config)
        }
        "postgresql" -> {
            val config = PostgreSQLConfig(host, port, username, password)
            return PostgreSQLConnection(config)
        }
        "sqlite" -> {
            val config = SQLiteConfig(host)
            return SQLiteConnection(config)
        }
        else -> throw IllegalArgumentException("Unsupported database type: $type")
    }
}
```

**回答:**
```kotlin
// 改善後
sealed class DatabaseConnection {
    abstract fun connect(): Boolean
    abstract fun disconnect(): Boolean
}

class MySQLConnection(private val config: MySQLConfig) : DatabaseConnection() {
    override fun connect() = true
    override fun disconnect() = true
}

class PostgreSQLConnection(private val config: PostgreSQLConfig) : DatabaseConnection() {
    override fun connect() = true
    override fun disconnect() = true
}

class SQLiteConnection(private val config: SQLiteConfig) : DatabaseConnection() {
    override fun connect() = true
    override fun disconnect() = true
}

object DatabaseConnectionFactory {
    fun create(type: String, host: String, port: Int, username: String, password: String): DatabaseConnection {
        return when (type) {
            "mysql" -> MySQLConnection(MySQLConfig(host, port, username, password))
            "postgresql" -> PostgreSQLConnection(PostgreSQLConfig(host, port, username, password))
            "sqlite" -> SQLiteConnection(SQLiteConfig(host))
            else -> throw IllegalArgumentException("Unsupported database type: $type")
        }
    }
}
```

**理由:**
- ファクトリーパターンにより、オブジェクト作成ロジックを一箇所に集約
- 新しいデータベースタイプの追加が容易
- オブジェクト作成の複雑さをクライアントから隠蔽

### 問題6: デコレーターパターンの実装
以下の機能追加をデコレーターパターンで実装してください。

```kotlin
// 改善前
class EmailService {
    fun sendEmail(to: String, subject: String, body: String) {
        // メール送信ロジック
        println("Sending email to $to: $subject")
    }
}

class LoggingEmailService {
    fun sendEmail(to: String, subject: String, body: String) {
        println("Logging: Sending email to $to")
        // メール送信ロジック
        println("Sending email to $to: $subject")
        println("Logging: Email sent successfully")
    }
}

class CachingEmailService {
    fun sendEmail(to: String, subject: String, body: String) {
        // キャッシュチェック
        println("Checking cache for $to")
        // メール送信ロジック
        println("Sending email to $to: $subject")
        // キャッシュに保存
        println("Caching email for $to")
    }
}
```

**回答:**
```kotlin
// 改善後
interface EmailService {
    fun sendEmail(to: String, subject: String, body: String)
}

class BasicEmailService : EmailService {
    override fun sendEmail(to: String, subject: String, body: String) {
        println("Sending email to $to: $subject")
    }
}

class LoggingEmailService(private val emailService: EmailService) : EmailService {
    override fun sendEmail(to: String, subject: String, body: String) {
        println("Logging: Sending email to $to")
        emailService.sendEmail(to, subject, body)
        println("Logging: Email sent successfully")
    }
}

class CachingEmailService(private val emailService: EmailService) : EmailService {
    private val cache = mutableMapOf<String, String>()
    
    override fun sendEmail(to: String, subject: String, body: String) {
        val cacheKey = "$to:$subject"
        if (cache.containsKey(cacheKey)) {
            println("Email found in cache for $to")
            return
        }
        
        emailService.sendEmail(to, subject, body)
        cache[cacheKey] = body
        println("Email cached for $to")
    }
}

// 使用例
val emailService = LoggingEmailService(
    CachingEmailService(
        BasicEmailService()
    )
)
```

**理由:**
- デコレーターパターンにより、機能を動的に組み合わせ可能
- 各機能が独立しており、再利用性が高い
- 機能の追加・削除が容易
