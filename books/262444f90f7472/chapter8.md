---
title: "コルーチン入門"
---

## リーダブルコード問題

### 問題1: 非同期処理の適切な構造化
以下のコールバック地獄をコルーチンを使って読みやすくしてください。

```kotlin
// 改善前
fun fetchUserData(userId: String, callback: (User?) -> Unit) {
    fetchUser(userId) { user ->
        if (user != null) {
            fetchUserProfile(user.id) { profile ->
                if (profile != null) {
                    fetchUserOrders(user.id) { orders ->
                        val userData = UserData(user, profile, orders)
                        callback(userData)
                    }
                } else {
                    callback(null)
                }
            }
        } else {
            callback(null)
        }
    }
}

fun fetchUser(userId: String, callback: (User?) -> Unit) {
    // 非同期でユーザー情報を取得
    Thread {
        Thread.sleep(1000)
        callback(User(userId, "John Doe"))
    }.start()
}
```

**回答:**
```kotlin
// 改善後
suspend fun fetchUserData(userId: String): UserData? {
    return try {
        val user = fetchUser(userId) ?: return null
        val profile = fetchUserProfile(user.id) ?: return null
        val orders = fetchUserOrders(user.id)
        UserData(user, profile, orders)
    } catch (e: Exception) {
        println("Error fetching user data: ${e.message}")
        null
    }
}

suspend fun fetchUser(userId: String): User? = withContext(Dispatchers.IO) {
    delay(1000) // 非同期処理をシミュレート
    User(userId, "John Doe")
}

suspend fun fetchUserProfile(userId: String): UserProfile? = withContext(Dispatchers.IO) {
    delay(500)
    UserProfile(userId, "Software Engineer")
}

suspend fun fetchUserOrders(userId: String): List<Order> = withContext(Dispatchers.IO) {
    delay(800)
    listOf(Order("1", 100.0), Order("2", 200.0))
}

// 使用例
fun main() = runBlocking {
    val userData = fetchUserData("123")
    userData?.let {
        println("User: ${it.user.name}")
        println("Profile: ${it.profile.title}")
        println("Orders: ${it.orders.size}")
    }
}
```

**理由:**
- コルーチンにより、非同期処理を同期的なコードのように記述可能
- ネストしたコールバックを平坦化し、可読性を大幅に向上
- エラーハンドリングがtry-catchで統一され、処理の流れが明確

### 問題2: 並行処理の最適化
以下の逐次処理を並行処理に変更して、パフォーマンスを向上させてください。

```kotlin
// 改善前
suspend fun processDataSequentially(): ProcessedData {
    val userData = fetchUserData() // 1秒
    val productData = fetchProductData() // 1.5秒
    val orderData = fetchOrderData() // 2秒
    val analyticsData = fetchAnalyticsData() // 1秒
    
    return ProcessedData(userData, productData, orderData, analyticsData)
    // 合計: 5.5秒
}
```

**回答:**
```kotlin
// 改善後
suspend fun processDataConcurrently(): ProcessedData = coroutineScope {
    val userDataDeferred = async { fetchUserData() }
    val productDataDeferred = async { fetchProductData() }
    val orderDataDeferred = async { fetchOrderData() }
    val analyticsDataDeferred = async { fetchAnalyticsData() }
    
    ProcessedData(
        userData = userDataDeferred.await(),
        productData = productDataDeferred.await(),
        orderData = orderDataDeferred.await(),
        analyticsData = analyticsDataDeferred.await()
    )
    // 合計: 約2秒（最も時間のかかる処理に依存）
}

// より柔軟なバージョン
suspend fun processDataWithTimeout(): ProcessedData? = withTimeout(3000) {
    coroutineScope {
        val userDataDeferred = async { fetchUserData() }
        val productDataDeferred = async { fetchProductData() }
        val orderDataDeferred = async { fetchOrderData() }
        val analyticsDataDeferred = async { fetchAnalyticsData() }
        
        ProcessedData(
            userData = userDataDeferred.await(),
            productData = productDataDeferred.await(),
            orderData = orderDataDeferred.await(),
            analyticsData = analyticsDataDeferred.await()
        )
    }
}
```

**理由:**
- async/awaitにより、独立した処理を並行実行可能
- 処理時間を大幅に短縮（5.5秒 → 約2秒）
- coroutineScopeにより、すべての子コルーチンの完了を待機

### 問題3: エラーハンドリングの改善
以下の例外処理をコルーチンで適切に実装してください。

```kotlin
// 改善前
fun processWithRetry(operation: () -> String, maxRetries: Int = 3): String? {
    var lastException: Exception? = null
    
    repeat(maxRetries) { attempt ->
        try {
            return operation()
        } catch (e: Exception) {
            lastException = e
            println("Attempt ${attempt + 1} failed: ${e.message}")
            if (attempt < maxRetries - 1) {
                Thread.sleep(1000 * (attempt + 1)) // 指数バックオフ
            }
        }
    }
    
    return null
}
```

**回答:**
```kotlin
// 改善後
suspend fun processWithRetry(
    operation: suspend () -> String,
    maxRetries: Int = 3,
    initialDelay: Long = 1000
): Result<String> = withContext(Dispatchers.Default) {
    var lastException: Exception? = null
    var delay = initialDelay
    
    repeat(maxRetries) { attempt ->
        try {
            val result = operation()
            return@withContext Result.success(result)
        } catch (e: Exception) {
            lastException = e
            println("Attempt ${attempt + 1} failed: ${e.message}")
            
            if (attempt < maxRetries - 1) {
                delay(delay)
                delay *= 2 // 指数バックオフ
            }
        }
    }
    
    Result.failure(lastException ?: Exception("All retry attempts failed"))
}

// 使用例
suspend fun fetchDataWithRetry(): String {
    return processWithRetry(
        operation = { 
            // ネットワークリクエストをシミュレート
            delay(500)
            if (Math.random() < 0.7) throw Exception("Network error")
            "Data fetched successfully"
        },
        maxRetries = 3
    ).getOrThrow()
}

// より高度なエラーハンドリング
suspend fun processWithFallback(): String {
    return processWithRetry(
        operation = { fetchDataFromPrimarySource() }
    ).recoverCatching {
        processWithRetry(
            operation = { fetchDataFromSecondarySource() }
        ).getOrThrow()
    }.getOrElse { 
        "Default data" 
    }
}
```

**理由:**
- suspend関数により、非同期処理でのリトライロジックを自然に記述
- Result型を使用して、成功/失敗を型安全に表現
- 指数バックオフによる適切なリトライ間隔

### 問題4: コルーチンスコープの適切な管理
以下のリークしやすいコルーチン管理を改善してください。

```kotlin
// 改善前
class DataProcessor {
    private var job: Job? = null
    
    fun startProcessing() {
        job = GlobalScope.launch {
            while (true) {
                processData()
                delay(1000)
            }
        }
    }
    
    fun stopProcessing() {
        job?.cancel()
    }
    
    private suspend fun processData() {
        // データ処理
    }
}
```

**回答:**
```kotlin
// 改善後
class DataProcessor {
    private val scope = CoroutineScope(Dispatchers.Default + SupervisorJob())
    private var processingJob: Job? = null
    
    fun startProcessing() {
        processingJob = scope.launch {
            while (isActive) {
                try {
                    processData()
                } catch (e: Exception) {
                    println("Error processing data: ${e.message}")
                    // エラーが発生しても他の処理は継続
                }
                delay(1000)
            }
        }
    }
    
    fun stopProcessing() {
        processingJob?.cancel()
    }
    
    fun cleanup() {
        scope.cancel() // スコープ全体をキャンセル
    }
    
    private suspend fun processData() {
        // データ処理
    }
}

// より安全な実装
class SafeDataProcessor : CoroutineScope {
    override val coroutineContext = Dispatchers.Default + SupervisorJob()
    
    fun startProcessing() = launch {
        while (isActive) {
            try {
                processData()
            } catch (e: Exception) {
                handleError(e)
            }
            delay(1000)
        }
    }
    
    private suspend fun processData() {
        // データ処理
    }
    
    private fun handleError(e: Exception) {
        // エラーハンドリング
    }
    
    fun cleanup() {
        coroutineContext.cancel()
    }
}
```

**理由:**
- 適切なコルーチンスコープを使用して、ライフサイクルを管理
- SupervisorJobにより、子コルーチンの失敗が親に影響しない
- isActiveでキャンセル状態をチェックし、適切に終了

### 問題5: チャンネルの活用
以下のプロデューサー・コンシューマーパターンをチャンネルを使って実装してください。

```kotlin
// 改善前
class DataProcessor {
    private val dataQueue = mutableListOf<String>()
    private val isProcessing = AtomicBoolean(false)
    
    fun addData(data: String) {
        synchronized(dataQueue) {
            dataQueue.add(data)
        }
    }
    
    fun startProcessing() {
        if (isProcessing.compareAndSet(false, true)) {
            Thread {
                while (isProcessing.get()) {
                    val data = synchronized(dataQueue) {
                        if (dataQueue.isNotEmpty()) dataQueue.removeAt(0) else null
                    }
                    if (data != null) {
                        processData(data)
                    } else {
                        Thread.sleep(100)
                    }
                }
            }.start()
        }
    }
    
    fun stopProcessing() {
        isProcessing.set(false)
    }
    
    private fun processData(data: String) {
        // データ処理
        println("Processing: $data")
    }
}
```

**回答:**
```kotlin
// 改善後
class DataProcessor {
    private val dataChannel = Channel<String>(Channel.UNLIMITED)
    private val scope = CoroutineScope(Dispatchers.Default + SupervisorJob())
    
    fun addData(data: String) {
        scope.launch {
            dataChannel.send(data)
        }
    }
    
    fun startProcessing() {
        scope.launch {
            for (data in dataChannel) {
                processData(data)
            }
        }
    }
    
    fun stopProcessing() {
        dataChannel.close()
        scope.cancel()
    }
    
    private suspend fun processData(data: String) {
        delay(100) // 処理時間をシミュレート
        println("Processing: $data")
    }
}
```

**理由:**
- チャンネルにより、プロデューサー・コンシューマーパターンを簡潔に実装
- スレッドセーフなデータ共有が自動的に保証
- コルーチンによる非同期処理でパフォーマンスが向上

### 問題6: フローとコールドストリーム
以下のデータストリーム処理をフローで実装してください。

```kotlin
// 改善前
class DataStream {
    private val listeners = mutableListOf<(String) -> Unit>()
    
    fun addListener(listener: (String) -> Unit) {
        listeners.add(listener)
    }
    
    fun removeListener(listener: (String) -> Unit) {
        listeners.remove(listener)
    }
    
    fun emitData(data: String) {
        listeners.forEach { it(data) }
    }
    
    fun startDataGeneration() {
        Thread {
            repeat(10) { i ->
                emitData("Data $i")
                Thread.sleep(1000)
            }
        }.start()
    }
}
```

**回答:**
```kotlin
// 改善後
class DataStream {
    private val _dataFlow = MutableSharedFlow<String>()
    val dataFlow: SharedFlow<String> = _dataFlow.asSharedFlow()
    
    suspend fun emitData(data: String) {
        _dataFlow.emit(data)
    }
    
    fun startDataGeneration() = CoroutineScope(Dispatchers.Default).launch {
        repeat(10) { i ->
            emitData("Data $i")
            delay(1000)
        }
    }
}

// 使用例
fun main() = runBlocking {
    val dataStream = DataStream()
    
    // 複数のコレクターが同じデータを受信
    launch {
        dataStream.dataFlow.collect { data ->
            println("Collector 1: $data")
        }
    }
    
    launch {
        dataStream.dataFlow.collect { data ->
            println("Collector 2: $data")
        }
    }
    
    dataStream.startDataGeneration()
    delay(5000)
}
```

**理由:**
- フローにより、リアクティブなデータストリームを簡潔に実装
- SharedFlowにより、複数のコレクターが同じデータを受信可能
- コルーチンと組み合わせて、非同期データ処理が容易