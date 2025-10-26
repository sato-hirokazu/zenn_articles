---
title: "実践的アルゴリズム / ミニアプリ"
---

## リーダブルコード問題

### 問題1: アルゴリズムの可読性向上
以下の複雑なソートアルゴリズムを読みやすく改善してください。

```kotlin
// 改善前
fun quickSort(arr: IntArray, low: Int, high: Int) {
    if (low < high) {
        val pi = partition(arr, low, high)
        quickSort(arr, low, pi - 1)
        quickSort(arr, pi + 1, high)
    }
}

fun partition(arr: IntArray, low: Int, high: Int): Int {
    val pivot = arr[high]
    var i = low - 1
    for (j in low until high) {
        if (arr[j] <= pivot) {
            i++
            val temp = arr[i]
            arr[i] = arr[j]
            arr[j] = temp
        }
    }
    val temp = arr[i + 1]
    arr[i + 1] = arr[high]
    arr[high] = temp
    return i + 1
}
```

**回答:**
```kotlin
// 改善後
fun IntArray.quickSort() {
    if (size <= 1) return
    quickSortRecursive(0, size - 1)
}

private fun IntArray.quickSortRecursive(low: Int, high: Int) {
    if (low >= high) return
    
    val pivotIndex = partition(low, high)
    quickSortRecursive(low, pivotIndex - 1)
    quickSortRecursive(pivotIndex + 1, high)
}

private fun IntArray.partition(low: Int, high: Int): Int {
    val pivot = this[high]
    var smallerElementIndex = low - 1
    
    for (currentIndex in low until high) {
        if (this[currentIndex] <= pivot) {
            smallerElementIndex++
            swap(smallerElementIndex, currentIndex)
        }
    }
    
    swap(smallerElementIndex + 1, high)
    return smallerElementIndex + 1
}

private fun IntArray.swap(index1: Int, index2: Int) {
    val temp = this[index1]
    this[index1] = this[index2]
    this[index2] = temp
}

// 使用例
fun main() {
    val numbers = intArrayOf(64, 34, 25, 12, 22, 11, 90)
    println("Before sorting: ${numbers.contentToString()}")
    
    numbers.quickSort()
    println("After sorting: ${numbers.contentToString()}")
}
```

**理由:**
- 拡張関数として実装し、より自然なAPIを提供
- 変数名を意味のある名前に変更
- スワップ処理を別関数に分離し、可読性を向上

### 問題2: データ構造の実装
以下の単方向リンクリストを読みやすく改善してください。

```kotlin
// 改善前
class Node(var data: Int, var next: Node? = null)

class LinkedList {
    private var head: Node? = null
    
    fun add(data: Int) {
        val newNode = Node(data)
        if (head == null) {
            head = newNode
        } else {
            var current = head
            while (current?.next != null) {
                current = current.next
            }
            current?.next = newNode
        }
    }
    
    fun remove(data: Int): Boolean {
        if (head == null) return false
        if (head?.data == data) {
            head = head?.next
            return true
        }
        var current = head
        while (current?.next != null) {
            if (current.next?.data == data) {
                current.next = current.next?.next
                return true
            }
            current = current.next
        }
        return false
    }
    
    fun contains(data: Int): Boolean {
        var current = head
        while (current != null) {
            if (current.data == data) return true
            current = current.next
        }
        return false
    }
}
```

**回答:**
```kotlin
// 改善後
sealed class LinkedList<out T> {
    object Empty : LinkedList<Nothing>()
    data class Node<T>(val value: T, val next: LinkedList<T>) : LinkedList<T>()
    
    companion object {
        fun <T> of(vararg elements: T): LinkedList<T> {
            return elements.foldRight(Empty as LinkedList<T>) { element, acc ->
                Node(element, acc)
            }
        }
    }
}

class MutableLinkedList<T> {
    private var head: Node<T>? = null
    private var size = 0
    
    private data class Node<T>(val data: T, var next: Node<T>? = null)
    
    fun add(data: T) {
        val newNode = Node(data)
        if (head == null) {
            head = newNode
        } else {
            findLastNode()?.next = newNode
        }
        size++
    }
    
    fun remove(data: T): Boolean {
        if (head == null) return false
        
        if (head?.data == data) {
            head = head?.next
            size--
            return true
        }
        
        val previousNode = findPreviousNode(data)
        if (previousNode != null) {
            previousNode.next = previousNode.next?.next
            size--
            return true
        }
        
        return false
    }
    
    fun contains(data: T): Boolean {
        return findNode(data) != null
    }
    
    fun size(): Int = size
    
    fun isEmpty(): Boolean = head == null
    
    fun toList(): List<T> {
        val result = mutableListOf<T>()
        var current = head
        while (current != null) {
            result.add(current.data)
            current = current.next
        }
        return result
    }
    
    private fun findLastNode(): Node<T>? {
        var current = head
        while (current?.next != null) {
            current = current.next
        }
        return current
    }
    
    private fun findNode(data: T): Node<T>? {
        var current = head
        while (current != null) {
            if (current.data == data) return current
            current = current.next
        }
        return null
    }
    
    private fun findPreviousNode(data: T): Node<T>? {
        var current = head
        while (current?.next != null) {
            if (current.next?.data == data) return current
            current = current.next
        }
        return null
    }
}

// 使用例
fun main() {
    val list = MutableLinkedList<Int>()
    list.add(1)
    list.add(2)
    list.add(3)
    
    println("Size: ${list.size()}")
    println("Contains 2: ${list.contains(2)}")
    println("List: ${list.toList()}")
    
    list.remove(2)
    println("After removing 2: ${list.toList()}")
}
```

**理由:**
- 不変なLinkedListと可変なMutableLinkedListを分離
- シールドクラスを使用して型安全な実装
- ヘルパー関数で処理を分離し、可読性を向上

### 問題3: ミニアプリの設計
以下のシンプルなタスク管理アプリを読みやすく改善してください。

```kotlin
// 改善前
class TaskManager {
    private val tasks = mutableListOf<Task>()
    
    fun addTask(title: String, priority: Int) {
        tasks.add(Task(title, priority, false))
    }
    
    fun completeTask(index: Int) {
        if (index >= 0 && index < tasks.size) {
            tasks[index] = tasks[index].copy(completed = true)
        }
    }
    
    fun getTasks(): List<Task> = tasks
    
    fun getCompletedTasks(): List<Task> = tasks.filter { it.completed }
    
    fun getPendingTasks(): List<Task> = tasks.filter { !it.completed }
    
    fun getTasksByPriority(priority: Int): List<Task> = tasks.filter { it.priority == priority }
}

data class Task(val title: String, val priority: Int, val completed: Boolean)
```

**回答:**
```kotlin
// 改善後
sealed class TaskPriority(val value: Int) {
    object Low : TaskPriority(1)
    object Medium : TaskPriority(2)
    object High : TaskPriority(3)
    object Critical : TaskPriority(4)
}

data class Task(
    val id: TaskId,
    val title: String,
    val priority: TaskPriority,
    val completed: Boolean = false,
    val createdAt: LocalDateTime = LocalDateTime.now()
) {
    fun markAsCompleted(): Task = copy(completed = true)
    fun isOverdue(): Boolean = !completed && createdAt.isBefore(LocalDateTime.now().minusDays(7))
}

@JvmInline
value class TaskId(val value: String)

class TaskManager {
    private val tasks = mutableMapOf<TaskId, Task>()
    private val idGenerator = AtomicLong(1)
    
    fun addTask(title: String, priority: TaskPriority): TaskId {
        require(title.isNotBlank()) { "Task title cannot be blank" }
        
        val taskId = TaskId("task_${idGenerator.getAndIncrement()}")
        val task = Task(taskId, title, priority)
        tasks[taskId] = task
        return taskId
    }
    
    fun completeTask(taskId: TaskId): Boolean {
        val task = tasks[taskId] ?: return false
        tasks[taskId] = task.markAsCompleted()
        return true
    }
    
    fun getTask(taskId: TaskId): Task? = tasks[taskId]
    
    fun getAllTasks(): List<Task> = tasks.values.toList()
    
    fun getCompletedTasks(): List<Task> = tasks.values.filter { it.completed }
    
    fun getPendingTasks(): List<Task> = tasks.values.filter { !it.completed }
    
    fun getTasksByPriority(priority: TaskPriority): List<Task> = 
        tasks.values.filter { it.priority == priority }
    
    fun getOverdueTasks(): List<Task> = tasks.values.filter { it.isOverdue() }
    
    fun deleteTask(taskId: TaskId): Boolean = tasks.remove(taskId) != null
    
    fun getTaskCount(): Int = tasks.size
    
    fun getCompletionRate(): Double {
        val totalTasks = tasks.size
        if (totalTasks == 0) return 0.0
        val completedTasks = getCompletedTasks().size
        return completedTasks.toDouble() / totalTasks
    }
}

// 使用例
fun main() {
    val taskManager = TaskManager()
    
    val task1 = taskManager.addTask("Learn Kotlin", TaskPriority.High)
    val task2 = taskManager.addTask("Write tests", TaskPriority.Medium)
    val task3 = taskManager.addTask("Review code", TaskPriority.Low)
    
    println("Total tasks: ${taskManager.getTaskCount()}")
    println("Completion rate: ${taskManager.getCompletionRate()}")
    
    taskManager.completeTask(task1)
    println("After completing task 1: ${taskManager.getCompletionRate()}")
    
    val highPriorityTasks = taskManager.getTasksByPriority(TaskPriority.High)
    println("High priority tasks: ${highPriorityTasks.map { it.title }}")
}
```

**理由:**
- タスクの優先度を型安全なシールドクラスで表現
- 一意のIDを使用してタスクを管理
- 作成日時とオーバーデューチェック機能を追加

### 問題4: パフォーマンス最適化
以下の非効率な検索処理を最適化してください。

```kotlin
// 改善前
class UserSearchService {
    private val users = mutableListOf<User>()
    
    fun addUser(user: User) {
        users.add(user)
    }
    
    fun searchByName(name: String): List<User> {
        val result = mutableListOf<User>()
        for (user in users) {
            if (user.name.contains(name, ignoreCase = true)) {
                result.add(user)
            }
        }
        return result
    }
    
    fun searchByEmail(email: String): User? {
        for (user in users) {
            if (user.email == email) {
                return user
            }
        }
        return null
    }
    
    fun searchByAgeRange(minAge: Int, maxAge: Int): List<User> {
        val result = mutableListOf<User>()
        for (user in users) {
            if (user.age in minAge..maxAge) {
                result.add(user)
            }
        }
        return result
    }
}

data class User(val name: String, val email: String, val age: Int)
```

**回答:**
```kotlin
// 改善後
class UserSearchService {
    private val users = mutableMapOf<String, User>()
    private val nameIndex = mutableMapOf<String, MutableSet<String>>()
    private val ageIndex = mutableMapOf<Int, MutableSet<String>>()
    
    fun addUser(user: User) {
        users[user.email] = user
        updateNameIndex(user)
        updateAgeIndex(user)
    }
    
    fun searchByName(name: String): List<User> {
        val searchTerm = name.lowercase()
        val matchingEmails = nameIndex.entries
            .filter { (indexedName, _) -> indexedName.contains(searchTerm) }
            .flatMap { (_, emails) -> emails }
            .toSet()
        
        return matchingEmails.mapNotNull { users[it] }
    }
    
    fun searchByEmail(email: String): User? = users[email]
    
    fun searchByAgeRange(minAge: Int, maxAge: Int): List<User> {
        val matchingEmails = ageIndex.entries
            .filter { (age, _) -> age in minAge..maxAge }
            .flatMap { (_, emails) -> emails }
            .toSet()
        
        return matchingEmails.mapNotNull { users[it] }
    }
    
    fun searchByMultipleCriteria(
        name: String? = null,
        minAge: Int? = null,
        maxAge: Int? = null
    ): List<User> {
        var candidateEmails = users.keys.toSet()
        
        if (name != null) {
            val nameMatches = searchByName(name).map { it.email }.toSet()
            candidateEmails = candidateEmails.intersect(nameMatches)
        }
        
        if (minAge != null || maxAge != null) {
            val ageMatches = searchByAgeRange(
                minAge ?: Int.MIN_VALUE,
                maxAge ?: Int.MAX_VALUE
            ).map { it.email }.toSet()
            candidateEmails = candidateEmails.intersect(ageMatches)
        }
        
        return candidateEmails.mapNotNull { users[it] }
    }
    
    private fun updateNameIndex(user: User) {
        val words = user.name.lowercase().split("\\s+".toRegex())
        words.forEach { word ->
            nameIndex.getOrPut(word) { mutableSetOf() }.add(user.email)
        }
    }
    
    private fun updateAgeIndex(user: User) {
        ageIndex.getOrPut(user.age) { mutableSetOf() }.add(user.email)
    }
    
    fun getStats(): SearchStats {
        return SearchStats(
            totalUsers = users.size,
            uniqueNames = nameIndex.size,
            ageRange = if (ageIndex.isEmpty()) null else 
                ageIndex.keys.minOrNull() to ageIndex.keys.maxOrNull()
        )
    }
}

data class SearchStats(
    val totalUsers: Int,
    val uniqueNames: Int,
    val ageRange: Pair<Int, Int>?
)

// 使用例
fun main() {
    val searchService = UserSearchService()
    
    searchService.addUser(User("John Doe", "john@example.com", 25))
    searchService.addUser(User("Jane Smith", "jane@example.com", 30))
    searchService.addUser(User("Bob Johnson", "bob@example.com", 25))
    
    val johnResults = searchService.searchByName("John")
    println("Users named John: ${johnResults.map { it.name }}")
    
    val age25Results = searchService.searchByAgeRange(25, 25)
    println("Users aged 25: ${age25Results.map { it.name }}")
    
    val combinedResults = searchService.searchByMultipleCriteria(
        name = "John",
        minAge = 20,
        maxAge = 30
    )
    println("Combined search results: ${combinedResults.map { it.name }}")
    
    println("Stats: ${searchService.getStats()}")
}
```

**理由:**
- インデックスを使用して検索パフォーマンスを向上
- 複数条件での検索を効率的に実装
- 統計情報を提供してデバッグとモニタリングを支援

### 問題5: キャッシュシステムの実装
以下の非効率なデータアクセスをキャッシュシステムで改善してください。

```kotlin
// 改善前
class UserService {
    private val userRepository = UserRepository()
    
    fun getUserById(id: String): User? {
        // 毎回データベースにアクセス
        return userRepository.findById(id)
    }
    
    fun getUserByEmail(email: String): User? {
        // 毎回データベースにアクセス
        return userRepository.findByEmail(email)
    }
    
    fun getUsersByRole(role: String): List<User> {
        // 毎回データベースにアクセス
        return userRepository.findByRole(role)
    }
}
```

**回答:**
```kotlin
// 改善後
class UserService {
    private val userRepository = UserRepository()
    private val cache = mutableMapOf<String, User>()
    private val emailToIdMap = mutableMapOf<String, String>()
    private val roleToIdsMap = mutableMapOf<String, Set<String>>()
    private val cacheExpiry = mutableMapOf<String, Long>()
    private val CACHE_TTL = 5 * 60 * 1000L // 5分
    
    fun getUserById(id: String): User? {
        return getCachedUser(id) ?: loadAndCacheUser(id)
    }
    
    fun getUserByEmail(email: String): User? {
        val id = emailToIdMap[email]
        return if (id != null) {
            getUserById(id)
        } else {
            loadAndCacheUserByEmail(email)
        }
    }
    
    fun getUsersByRole(role: String): List<User> {
        val cachedIds = roleToIdsMap[role]
        return if (cachedIds != null && isCacheValid(role)) {
            cachedIds.mapNotNull { getUserById(it) }
        } else {
            loadAndCacheUsersByRole(role)
        }
    }
    
    private fun getCachedUser(id: String): User? {
        return if (isCacheValid(id)) {
            cache[id]
        } else {
            cache.remove(id)
            null
        }
    }
    
    private fun loadAndCacheUser(id: String): User? {
        val user = userRepository.findById(id)
        if (user != null) {
            cache[id] = user
            cacheExpiry[id] = System.currentTimeMillis() + CACHE_TTL
            emailToIdMap[user.email] = user.id
        }
        return user
    }
    
    private fun loadAndCacheUserByEmail(email: String): User? {
        val user = userRepository.findByEmail(email)
        if (user != null) {
            cache[user.id] = user
            cacheExpiry[user.id] = System.currentTimeMillis() + CACHE_TTL
            emailToIdMap[email] = user.id
        }
        return user
    }
    
    private fun loadAndCacheUsersByRole(role: String): List<User> {
        val users = userRepository.findByRole(role)
        val userIds = mutableSetOf<String>()
        
        users.forEach { user ->
            cache[user.id] = user
            cacheExpiry[user.id] = System.currentTimeMillis() + CACHE_TTL
            emailToIdMap[user.email] = user.id
            userIds.add(user.id)
        }
        
        roleToIdsMap[role] = userIds
        cacheExpiry[role] = System.currentTimeMillis() + CACHE_TTL
        
        return users
    }
    
    private fun isCacheValid(key: String): Boolean {
        val expiry = cacheExpiry[key] ?: return false
        return System.currentTimeMillis() < expiry
    }
    
    fun clearCache() {
        cache.clear()
        emailToIdMap.clear()
        roleToIdsMap.clear()
        cacheExpiry.clear()
    }
}
```

**理由:**
- キャッシュシステムにより、データベースアクセスを大幅に削減
- 複数の検索方法に対応した効率的なキャッシュ戦略
- TTL（Time To Live）による自動的なキャッシュ無効化

### 問題6: イベント駆動アーキテクチャの実装
以下の密結合なシステムをイベント駆動アーキテクチャで改善してください。

```kotlin
// 改善前
class OrderService {
    private val inventoryService = InventoryService()
    private val paymentService = PaymentService()
    private val emailService = EmailService()
    private val auditService = AuditService()
    
    fun processOrder(order: Order) {
        // 在庫チェック
        if (!inventoryService.checkAvailability(order.items)) {
            throw InsufficientInventoryException()
        }
        
        // 支払い処理
        val paymentResult = paymentService.processPayment(order.paymentInfo)
        if (!paymentResult.success) {
            throw PaymentFailedException()
        }
        
        // 在庫更新
        inventoryService.updateInventory(order.items)
        
        // メール送信
        emailService.sendOrderConfirmation(order.customerEmail, order.id)
        
        // 監査ログ
        auditService.logOrderProcessed(order.id, order.total)
        
        // 在庫不足時の通知
        if (inventoryService.getLowStockItems().isNotEmpty()) {
            emailService.sendLowStockAlert(inventoryService.getLowStockItems())
        }
    }
}
```

**回答:**
```kotlin
// 改善後
// イベント定義
sealed class DomainEvent {
    data class OrderCreated(val order: Order) : DomainEvent()
    data class PaymentProcessed(val orderId: String, val amount: Double) : DomainEvent()
    data class InventoryUpdated(val items: List<OrderItem>) : DomainEvent()
    data class OrderCompleted(val orderId: String) : DomainEvent()
    data class LowStockDetected(val items: List<Item>) : DomainEvent()
}

// イベントハンドラー
interface EventHandler<T : DomainEvent> {
    fun handle(event: T)
}

class OrderCreatedHandler : EventHandler<DomainEvent.OrderCreated> {
    private val inventoryService = InventoryService()
    
    override fun handle(event: DomainEvent.OrderCreated) {
        if (!inventoryService.checkAvailability(event.order.items)) {
            throw InsufficientInventoryException()
        }
    }
}

class PaymentProcessedHandler : EventHandler<DomainEvent.PaymentProcessed> {
    private val inventoryService = InventoryService()
    private val emailService = EmailService()
    
    override fun handle(event: DomainEvent.PaymentProcessed) {
        // 在庫更新
        inventoryService.updateInventory(event.orderId)
        
        // メール送信
        emailService.sendOrderConfirmation(event.orderId)
        
        // 低在庫チェック
        val lowStockItems = inventoryService.getLowStockItems()
        if (lowStockItems.isNotEmpty()) {
            EventBus.publish(DomainEvent.LowStockDetected(lowStockItems))
        }
    }
}

class LowStockHandler : EventHandler<DomainEvent.LowStockDetected> {
    private val emailService = EmailService()
    
    override fun handle(event: DomainEvent.LowStockDetected) {
        emailService.sendLowStockAlert(event.items)
    }
}

// イベントバス
object EventBus {
    private val handlers = mutableMapOf<Class<*>, MutableList<EventHandler<*>>>()
    
    fun <T : DomainEvent> subscribe(eventType: Class<T>, handler: EventHandler<T>) {
        handlers.getOrPut(eventType) { mutableListOf() }.add(handler as EventHandler<*>)
    }
    
    fun <T : DomainEvent> publish(event: T) {
        val eventHandlers = handlers[event::class.java] ?: return
        eventHandlers.forEach { handler ->
            try {
                (handler as EventHandler<T>).handle(event)
            } catch (e: Exception) {
                // エラーハンドリング
                println("Error handling event: ${e.message}")
            }
        }
    }
}

// 改善されたOrderService
class OrderService {
    private val paymentService = PaymentService()
    private val auditService = AuditService()
    
    init {
        // イベントハンドラーの登録
        EventBus.subscribe(DomainEvent.OrderCreated::class.java, OrderCreatedHandler())
        EventBus.subscribe(DomainEvent.PaymentProcessed::class.java, PaymentProcessedHandler())
        EventBus.subscribe(DomainEvent.LowStockDetected::class.java, LowStockHandler())
    }
    
    fun processOrder(order: Order) {
        // イベント発行
        EventBus.publish(DomainEvent.OrderCreated(order))
        
        // 支払い処理
        val paymentResult = paymentService.processPayment(order.paymentInfo)
        if (!paymentResult.success) {
            throw PaymentFailedException()
        }
        
        // イベント発行
        EventBus.publish(DomainEvent.PaymentProcessed(order.id, order.total))
        
        // 監査ログ
        auditService.logOrderProcessed(order.id, order.total)
        
        // 完了イベント発行
        EventBus.publish(DomainEvent.OrderCompleted(order.id))
    }
}
```

**理由:**
- イベント駆動アーキテクチャにより、システムの疎結合を実現
- 各コンポーネントが独立して動作し、保守性が向上
- 新しい機能の追加が既存コードに影響しない