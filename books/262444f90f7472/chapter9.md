---
title: "Kotest / MockK"
---

## リーダブルコード問題

### 問題1: テストの可読性向上
以下の読みにくいテストをKotestのBDDスタイルで改善してください。

```kotlin
// 改善前
class UserServiceTest {
    @Test
    fun testCreateUser() {
        val userRepository = mockk<UserRepository>()
        val emailService = mockk<EmailService>()
        val userService = UserService(userRepository, emailService)
        
        val user = User("John", "john@example.com")
        every { userRepository.save(any()) } returns user
        every { emailService.sendWelcomeEmail(any()) } just Runs
        
        val result = userService.createUser("John", "john@example.com")
        
        assertThat(result).isNotNull()
        assertThat(result.name).isEqualTo("John")
        assertThat(result.email).isEqualTo("john@example.com")
        verify { userRepository.save(any()) }
        verify { emailService.sendWelcomeEmail(any()) }
    }
    
    @Test
    fun testCreateUserWithInvalidEmail() {
        val userRepository = mockk<UserRepository>()
        val emailService = mockk<EmailService>()
        val userService = UserService(userRepository, emailService)
        
        assertThrows<IllegalArgumentException> {
            userService.createUser("John", "invalid-email")
        }
        
        verify(exactly = 0) { userRepository.save(any()) }
        verify(exactly = 0) { emailService.sendWelcomeEmail(any()) }
    }
}
```

**回答:**
```kotlin
// 改善後
class UserServiceTest : BehaviorSpec({
    Given("a user service with mocked dependencies") {
        val userRepository = mockk<UserRepository>()
        val emailService = mockk<EmailService>()
        val userService = UserService(userRepository, emailService)
        
        When("creating a user with valid data") {
            val userName = "John"
            val userEmail = "john@example.com"
            val expectedUser = User(userName, userEmail)
            
            every { userRepository.save(any()) } returns expectedUser
            every { emailService.sendWelcomeEmail(any()) } just Runs
            
            val result = userService.createUser(userName, userEmail)
            
            Then("it should return the created user") {
                result shouldNotBe null
                result.name shouldBe userName
                result.email shouldBe userEmail
            }
            
            And("it should save the user to repository") {
                verify { userRepository.save(any()) }
            }
            
            And("it should send welcome email") {
                verify { emailService.sendWelcomeEmail(any()) }
            }
        }
        
        When("creating a user with invalid email") {
            val userName = "John"
            val invalidEmail = "invalid-email"
            
            Then("it should throw IllegalArgumentException") {
                shouldThrow<IllegalArgumentException> {
                    userService.createUser(userName, invalidEmail)
                }
            }
            
            And("it should not save user or send email") {
                verify(exactly = 0) { userRepository.save(any()) }
                verify(exactly = 0) { emailService.sendWelcomeEmail(any()) }
            }
        }
    }
})
```

**理由:**
- BDDスタイルにより、テストの意図が明確になる
- Given-When-Then構造でテストの流れが読みやすい
- 各ステップが自然言語で表現され、理解しやすい

### 問題2: モックの適切な設定
以下の複雑なモック設定を読みやすく整理してください。

```kotlin
// 改善前
@Test
fun testComplexUserProcessing() {
    val userRepository = mockk<UserRepository>()
    val emailService = mockk<EmailService>()
    val notificationService = mockk<NotificationService>()
    val auditService = mockk<AuditService>()
    
    every { userRepository.findById(1L) } returns User(1L, "John", "john@example.com")
    every { userRepository.findById(2L) } returns User(2L, "Jane", "jane@example.com")
    every { userRepository.findById(3L) } returns null
    every { userRepository.save(any()) } answers { firstArg() }
    every { emailService.sendEmail(any(), any()) } returns true
    every { emailService.sendEmail("invalid@email", any()) } returns false
    every { notificationService.sendPushNotification(any(), any()) } just Runs
    every { auditService.logAction(any(), any()) } just Runs
    
    val userService = UserService(userRepository, emailService, notificationService, auditService)
    
    // テスト実行...
}
```

**回答:**
```kotlin
// 改善後
class UserServiceTest : BehaviorSpec({
    Given("a user service with mocked dependencies") {
        val userRepository = mockk<UserRepository>()
        val emailService = mockk<EmailService>()
        val notificationService = mockk<NotificationService>()
        val auditService = mockk<AuditService>()
        val userService = UserService(userRepository, emailService, notificationService, auditService)
        
        // モック設定をヘルパー関数に分離
        setupUserRepositoryMocks(userRepository)
        setupEmailServiceMocks(emailService)
        setupNotificationServiceMocks(notificationService)
        setupAuditServiceMocks(auditService)
        
        When("processing multiple users") {
            val userIds = listOf(1L, 2L, 3L)
            val results = userIds.map { userService.processUser(it) }
            
            Then("it should process valid users successfully") {
                results[0] shouldBe "User John processed successfully"
                results[1] shouldBe "User Jane processed successfully"
                results[2] shouldBe "User not found"
            }
        }
    }
})

// モック設定を分離したヘルパー関数
private fun setupUserRepositoryMocks(userRepository: UserRepository) {
    every { userRepository.findById(1L) } returns User(1L, "John", "john@example.com")
    every { userRepository.findById(2L) } returns User(2L, "Jane", "jane@example.com")
    every { userRepository.findById(3L) } returns null
    every { userRepository.save(any()) } answers { firstArg() }
}

private fun setupEmailServiceMocks(emailService: EmailService) {
    every { emailService.sendEmail(any(), any()) } returns true
    every { emailService.sendEmail("invalid@email", any()) } returns false
}

private fun setupNotificationServiceMocks(notificationService: NotificationService) {
    every { notificationService.sendPushNotification(any(), any()) } just Runs
}

private fun setupAuditServiceMocks(auditService: AuditService) {
    every { auditService.logAction(any(), any()) } just Runs
}
```

**理由:**
- モック設定をヘルパー関数に分離し、テストの可読性を向上
- 各モックの設定が独立して管理され、保守しやすい
- テストの本質的な部分に集中できる

### 問題3: テストデータの管理
以下のハードコードされたテストデータを改善してください。

```kotlin
// 改善前
@Test
fun testUserValidation() {
    val userService = UserService()
    
    // 有効なユーザー
    assertTrue(userService.isValidUser("John", "john@example.com", 25))
    assertTrue(userService.isValidUser("Jane", "jane@test.com", 30))
    
    // 無効なユーザー
    assertFalse(userService.isValidUser("", "john@example.com", 25))
    assertFalse(userService.isValidUser("John", "invalid-email", 25))
    assertFalse(userService.isValidUser("John", "john@example.com", -1))
    assertFalse(userService.isValidUser("John", "john@example.com", 150))
}
```

**回答:**
```kotlin
// 改善後
class UserServiceTest : BehaviorSpec({
    Given("a user service") {
        val userService = UserService()
        
        When("validating user data") {
            Then("it should accept valid users") {
                validUsers.forEach { user ->
                    userService.isValidUser(user.name, user.email, user.age) shouldBe true
                }
            }
            
            Then("it should reject users with invalid names") {
                invalidNameUsers.forEach { user ->
                    userService.isValidUser(user.name, user.email, user.age) shouldBe false
                }
            }
            
            Then("it should reject users with invalid emails") {
                invalidEmailUsers.forEach { user ->
                    userService.isValidUser(user.name, user.email, user.age) shouldBe false
                }
            }
            
            Then("it should reject users with invalid ages") {
                invalidAgeUsers.forEach { user ->
                    userService.isValidUser(user.name, user.email, user.age) shouldBe false
                }
            }
        }
    }
})

// テストデータを分離
private data class TestUser(val name: String, val email: String, val age: Int)

private val validUsers = listOf(
    TestUser("John", "john@example.com", 25),
    TestUser("Jane", "jane@test.com", 30),
    TestUser("Bob", "bob@company.org", 45)
)

private val invalidNameUsers = listOf(
    TestUser("", "john@example.com", 25),
    TestUser("   ", "jane@test.com", 30),
    TestUser("A".repeat(101), "bob@company.org", 45)
)

private val invalidEmailUsers = listOf(
    TestUser("John", "invalid-email", 25),
    TestUser("Jane", "@test.com", 30),
    TestUser("Bob", "bob@", 45)
)

private val invalidAgeUsers = listOf(
    TestUser("John", "john@example.com", -1),
    TestUser("Jane", "jane@test.com", 0),
    TestUser("Bob", "bob@company.org", 150)
)
```

**理由:**
- テストデータを構造化し、意図が明確になる
- データ駆動テストにより、多くのケースを効率的にテスト
- テストデータの追加・変更が容易

### 問題4: 非同期処理のテスト
以下の非同期処理のテストを適切に実装してください。

```kotlin
// 改善前
@Test
fun testAsyncUserProcessing() {
    val userService = UserService()
    val users = listOf("user1", "user2", "user3")
    
    val results = mutableListOf<String>()
    val latch = CountDownLatch(users.size)
    
    users.forEach { userId ->
        Thread {
            val result = userService.processUserAsync(userId)
            results.add(result)
            latch.countDown()
        }.start()
    }
    
    latch.await(5, TimeUnit.SECONDS)
    
    assertThat(results).hasSize(3)
    assertThat(results).contains("user1 processed", "user2 processed", "user3 processed")
}
```

**回答:**
```kotlin
// 改善後
class UserServiceTest : BehaviorSpec({
    Given("a user service with async processing") {
        val userService = UserService()
        
        When("processing multiple users asynchronously") {
            val userIds = listOf("user1", "user2", "user3")
            val results = runBlocking {
                userIds.map { userId ->
                    async { userService.processUserAsync(userId) }
                }.awaitAll()
            }
            
            Then("it should process all users successfully") {
                results shouldHaveSize 3
                results shouldContain "user1 processed"
                results shouldContain "user2 processed"
                results shouldContain "user3 processed"
            }
        }
        
        When("processing users with timeout") {
            val userIds = listOf("slow-user", "fast-user")
            
            Then("it should complete within timeout") {
                val results = runBlocking {
                    withTimeout(2000) {
                        userIds.map { userId ->
                            async { userService.processUserAsync(userId) }
                        }.awaitAll()
                    }
                }
                
                results shouldHaveSize 2
            }
        }
        
        When("processing fails for some users") {
            val userIds = listOf("valid-user", "invalid-user", "another-valid-user")
            
            Then("it should handle failures gracefully") {
                val results = runBlocking {
                    userIds.map { userId ->
                        async {
                            try {
                                userService.processUserAsync(userId)
                            } catch (e: Exception) {
                                "Error processing $userId: ${e.message}"
                            }
                        }
                    }.awaitAll()
                }
                
                results shouldHaveSize 3
                results shouldContain "valid-user processed"
                results shouldContain "Error processing invalid-user"
                results shouldContain "another-valid-user processed"
            }
        }
    }
})
```

**理由:**
- runBlockingとasync/awaitを使用して、非同期処理を同期的にテスト
- タイムアウト処理を適切に実装
- エラーハンドリングを含む包括的なテスト

### 問題5: パラメータ化テスト
以下の重複したテストをパラメータ化テストで改善してください。

```kotlin
// 改善前
class CalculatorTest : BehaviorSpec({
    Given("a calculator") {
        val calculator = Calculator()
        
        When("adding positive numbers") {
            Then("it should return correct result") {
                calculator.add(2, 3) shouldBe 5
                calculator.add(10, 20) shouldBe 30
                calculator.add(100, 200) shouldBe 300
            }
        }
        
        When("adding negative numbers") {
            Then("it should return correct result") {
                calculator.add(-2, -3) shouldBe -5
                calculator.add(-10, -20) shouldBe -30
            }
        }
        
        When("adding mixed numbers") {
            Then("it should return correct result") {
                calculator.add(5, -3) shouldBe 2
                calculator.add(-10, 15) shouldBe 5
            }
        }
    }
})
```

**回答:**
```kotlin
// 改善後
class CalculatorTest : BehaviorSpec({
    Given("a calculator") {
        val calculator = Calculator()
        
        When("performing addition") {
            Then("it should return correct results for various inputs") {
                val testCases = listOf(
                    Triple(2, 3, 5),
                    Triple(10, 20, 30),
                    Triple(100, 200, 300),
                    Triple(-2, -3, -5),
                    Triple(-10, -20, -30),
                    Triple(5, -3, 2),
                    Triple(-10, 15, 5)
                )
                
                testCases.forEach { (a, b, expected) ->
                    calculator.add(a, b) shouldBe expected
                }
            }
        }
    }
})

// より高度なパラメータ化テスト
class AdvancedCalculatorTest : BehaviorSpec({
    Given("a calculator") {
        val calculator = Calculator()
        
        data class TestCase(
            val a: Int,
            val b: Int,
            val operation: String,
            val expected: Int
        )
        
        val testCases = listOf(
            TestCase(2, 3, "add", 5),
            TestCase(10, 2, "multiply", 20),
            TestCase(15, 3, "divide", 5),
            TestCase(10, 3, "subtract", 7)
        )
        
        testCases.forEach { testCase ->
            When("performing ${testCase.operation} with ${testCase.a} and ${testCase.b}") {
                Then("it should return ${testCase.expected}") {
                    val result = when (testCase.operation) {
                        "add" -> calculator.add(testCase.a, testCase.b)
                        "multiply" -> calculator.multiply(testCase.a, testCase.b)
                        "divide" -> calculator.divide(testCase.a, testCase.b)
                        "subtract" -> calculator.subtract(testCase.a, testCase.b)
                        else -> throw IllegalArgumentException("Unknown operation")
                    }
                    result shouldBe testCase.expected
                }
            }
        }
    }
})
```

**理由:**
- パラメータ化テストにより、複数のテストケースを効率的に実行
- テストデータとテストロジックを分離し、保守性を向上
- 新しいテストケースの追加が容易

### 問題6: モックの検証とスパイ
以下の複雑なモック検証を適切に実装してください。

```kotlin
// 改善前
class OrderServiceTest : BehaviorSpec({
    Given("an order service") {
        val orderRepository = mockk<OrderRepository>()
        val paymentService = mockk<PaymentService>()
        val emailService = mockk<EmailService>()
        val orderService = OrderService(orderRepository, paymentService, emailService)
        
        When("processing an order") {
            val order = Order("123", 100.0, "user@example.com")
            
            every { orderRepository.save(any()) } returns order
            every { paymentService.processPayment(any(), any()) } returns true
            every { emailService.sendConfirmation(any(), any()) } just Runs
            
            val result = orderService.processOrder(order)
            
            Then("it should process successfully") {
                result shouldBe true
                verify { orderRepository.save(order) }
                verify { paymentService.processPayment(order.id, order.amount) }
                verify { emailService.sendConfirmation(order.email, order.id) }
            }
        }
    }
})
```

**回答:**
```kotlin
// 改善後
class OrderServiceTest : BehaviorSpec({
    Given("an order service") {
        val orderRepository = mockk<OrderRepository>(relaxed = true)
        val paymentService = mockk<PaymentService>(relaxed = true)
        val emailService = mockk<EmailService>(relaxed = true)
        val orderService = OrderService(orderRepository, paymentService, emailService)
        
        When("processing a valid order") {
            val order = Order("123", 100.0, "user@example.com")
            
            every { orderRepository.save(any()) } returns order
            every { paymentService.processPayment(any(), any()) } returns true
            every { emailService.sendConfirmation(any(), any()) } just Runs
            
            val result = orderService.processOrder(order)
            
            Then("it should process successfully") {
                result shouldBe true
                
                verify(exactly = 1) { orderRepository.save(order) }
                verify(exactly = 1) { paymentService.processPayment(order.id, order.amount) }
                verify(exactly = 1) { emailService.sendConfirmation(order.email, order.id) }
                verifyOrder { orderRepository.save(order) }
            }
        }
        
        When("payment fails") {
            val order = Order("456", 50.0, "user@example.com")
            
            every { orderRepository.save(any()) } returns order
            every { paymentService.processPayment(any(), any()) } returns false
            
            val result = orderService.processOrder(order)
            
            Then("it should not send confirmation email") {
                result shouldBe false
                
                verify { orderRepository.save(order) }
                verify { paymentService.processPayment(order.id, order.amount) }
                verify(exactly = 0) { emailService.sendConfirmation(any(), any()) }
            }
        }
        
        When("using spy to track method calls") {
            val realService = OrderService(orderRepository, paymentService, emailService)
            val spyService = spyk(realService)
            
            every { orderRepository.save(any()) } returns order
            every { paymentService.processPayment(any(), any()) } returns true
            every { emailService.sendConfirmation(any(), any()) } just Runs
            
            val result = spyService.processOrder(order)
            
            Then("it should call internal methods") {
                result shouldBe true
                verify { spyService.validateOrder(order) }
                verify { spyService.updateOrderStatus(order, "PROCESSED") }
            }
        }
    }
})
```

**理由:**
- より詳細な検証により、テストの信頼性を向上
- スパイを使用して内部メソッドの呼び出しを検証
- エラーケースも含めた包括的なテストを実装