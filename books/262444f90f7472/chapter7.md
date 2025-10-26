---
title: "ジェネリクスとコントラバリアンス"
---

## リーダブルコード問題

### 問題1: 型安全なジェネリック関数の実装
以下の型安全でない関数をジェネリクスを使って型安全にしてください。

```kotlin
// 改善前
fun processItems(items: List<Any>): List<String> {
    val result = mutableListOf<String>()
    for (item in items) {
        when (item) {
            is String -> result.add("String: $item")
            is Int -> result.add("Number: $item")
            is Double -> result.add("Decimal: $item")
            else -> result.add("Unknown: $item")
        }
    }
    return result
}

fun findFirst(items: List<Any>, target: Any): Any? {
    for (item in items) {
        if (item == target) {
            return item
        }
    }
    return null
}
```

**回答:**
```kotlin
// 改善後
inline fun <T> List<T>.processItems(processor: (T) -> String): List<String> {
    return map(processor)
}

inline fun <T> List<T>.findFirst(predicate: (T) -> Boolean): T? {
    return firstOrNull(predicate)
}

// 使用例
val strings = listOf("hello", "world")
val numbers = listOf(1, 2, 3, 4, 5)

val processedStrings = strings.processItems { "String: $it" }
val processedNumbers = numbers.processItems { "Number: $it" }

val firstEven = numbers.findFirst { it % 2 == 0 }
val firstLongString = strings.findFirst { it.length > 4 }
```

**理由:**
- ジェネリクスにより、型安全で再利用可能な関数を作成
- 高階関数を使用して処理ロジックを外部から注入可能
- コンパイル時に型チェックが行われるため、実行時エラーを防止

### 問題2: 共変性と反変性の理解
以下の型不変なコレクション操作を共変性と反変性を活用して改善してください。

```kotlin
// 改善前
open class Animal
class Dog : Animal()
class Cat : Animal()

fun processAnimals(animals: List<Animal>): List<Animal> {
    return animals.filter { it is Dog }
}

fun addAnimals(animals: MutableList<Animal>) {
    animals.add(Dog())
    animals.add(Cat())
}

// 使用時に型エラーが発生
val dogs: List<Dog> = listOf(Dog(), Dog())
// val processed = processAnimals(dogs) // エラー: List<Dog>はList<Animal>に代入不可
```

**回答:**
```kotlin
// 改善後
open class Animal
class Dog : Animal()
class Cat : Animal()

// 共変性を活用（outキーワード）
fun processAnimals(animals: List<Animal>): List<Animal> {
    return animals.filter { it is Dog }
}

// より柔軟な関数
fun <T : Animal> processAnimals(animals: List<T>): List<T> {
    return animals.filter { it is Dog } as List<T>
}

// 反変性を活用（inキーワード）
interface AnimalProcessor<in T> {
    fun process(animal: T): String
}

class DogProcessor : AnimalProcessor<Dog> {
    override fun process(animal: Dog): String = "Processing dog: $animal"
}

class AnimalProcessorImpl : AnimalProcessor<Animal> {
    override fun process(animal: Animal): String = "Processing animal: $animal"
}

// 使用例
val dogs: List<Dog> = listOf(Dog(), Dog())
val processed = processAnimals(dogs) // 型安全

val dogProcessor: AnimalProcessor<Dog> = DogProcessor()
val animalProcessor: AnimalProcessor<Animal> = AnimalProcessorImpl()

// 反変性により、AnimalProcessor<Animal>をAnimalProcessor<Dog>として使用可能
val processor: AnimalProcessor<Dog> = animalProcessor
```

**理由:**
- 共変性（out）により、サブタイプのコレクションをスーパータイプとして扱える
- 反変性（in）により、スーパータイプのプロセッサをサブタイプで使用可能
- ジェネリック境界（T : Animal）により、型制約を明確に表現

### 問題3: 型消去とreified型パラメータ
以下の型消去の問題をreified型パラメータで解決してください。

```kotlin
// 改善前
fun <T> createList(): List<T> {
    return when (T::class) { // エラー: 型消去により実行時にTの型情報が利用できない
        String::class -> listOf("hello", "world")
        Int::class -> listOf(1, 2, 3)
        else -> emptyList()
    }
}

fun <T> isStringList(list: List<Any>): Boolean {
    return list is List<String> // 警告: 型消去により実行時チェックが不完全
}
```

**回答:**
```kotlin
// 改善後
inline fun <reified T> createList(): List<T> {
    return when (T::class) {
        String::class -> listOf("hello", "world") as List<T>
        Int::class -> listOf(1, 2, 3) as List<T>
        else -> emptyList()
    }
}

inline fun <reified T> List<*>.isListOf(): Boolean {
    return this is List<T>
}

inline fun <reified T> List<*>.filterByType(): List<T> {
    return filterIsInstance<T>()
}

// 使用例
val stringList = createList<String>() // ["hello", "world"]
val intList = createList<Int>() // [1, 2, 3]

val mixedList = listOf("hello", 123, "world", 456)
val isStringList = mixedList.isListOf<String>() // false
val strings = mixedList.filterByType<String>() // ["hello", "world"]
```

**理由:**
- reified型パラメータにより、実行時に型情報にアクセス可能
- 型消去の制限を回避し、より柔軟な型操作が可能
- インライン関数と組み合わせることで、パフォーマンスのオーバーヘッドなし

### 問題4: 複雑なジェネリック制約
以下の複雑な型制約を適切に設計してください。

```kotlin
// 改善前
interface Repository<T> {
    fun save(entity: T): T
    fun findById(id: Long): T?
    fun findAll(): List<T>
}

interface Service<T> {
    fun process(entity: T): T
}

// 型制約が不明確で、実装時に問題が発生する可能性
class UserService<T>(private val repository: Repository<T>) : Service<T> {
    override fun process(entity: T): T {
        // 何らかの処理
        return repository.save(entity)
    }
}
```

**回答:**
```kotlin
// 改善後
interface Entity {
    val id: Long?
}

interface Repository<T : Entity> {
    fun save(entity: T): T
    fun findById(id: Long): T?
    fun findAll(): List<T>
}

interface Service<T : Entity> {
    fun process(entity: T): T
}

// 具体的なエンティティ
data class User(override val id: Long?, val name: String, val email: String) : Entity

// 型制約を明確にした実装
class UserService<T : Entity>(
    private val repository: Repository<T>
) : Service<T> {
    
    override fun process(entity: T): T {
        // エンティティの処理ロジック
        val processedEntity = processEntity(entity)
        return repository.save(processedEntity)
    }
    
    private fun processEntity(entity: T): T {
        // 具体的な処理はサブクラスで実装
        return entity
    }
}

// より具体的な実装
class UserServiceImpl(
    private val userRepository: Repository<User>
) : Service<User> {
    
    override fun process(entity: User): User {
        val processedUser = entity.copy(
            name = entity.name.trim().capitalize(),
            email = entity.email.lowercase()
        )
        return userRepository.save(processedUser)
    }
}
```

**理由:**
- 型制約（T : Entity）により、必要なプロパティやメソッドを保証
- ジェネリック型と具体的な型を適切に使い分け
- 型安全性を保ちながら、柔軟性も確保

### 問題5: ジェネリック制約の活用
以下の型制約を適切に設計してください。

```kotlin
// 改善前
fun <T> findMax(items: List<T>): T? {
    if (items.isEmpty()) return null
    
    var max = items[0]
    for (item in items) {
        if (item > max) { // エラー: 比較演算子が定義されていない
            max = item
        }
    }
    return max
}

fun <T> sortItems(items: List<T>): List<T> {
    return items.sorted() // エラー: ソート可能な型が不明
}
```

**回答:**
```kotlin
// 改善後
fun <T : Comparable<T>> findMax(items: List<T>): T? {
    if (items.isEmpty()) return null
    
    var max = items[0]
    for (item in items) {
        if (item > max) {
            max = item
        }
    }
    return max
}

fun <T : Comparable<T>> sortItems(items: List<T>): List<T> {
    return items.sorted()
}

// より柔軟な制約
interface Drawable {
    fun draw(): String
}

interface Movable {
    fun move(x: Int, y: Int)
}

fun <T> processGameObject(obj: T) where T : Drawable, T : Movable {
    obj.move(10, 20)
    println(obj.draw())
}
```

**理由:**
- Comparable制約により、比較可能な型のみを受け入れる
- where句を使用して複数の制約を組み合わせ
- 型安全性を保ちながら、必要な機能を保証

### 問題6: 型消去の回避
以下の型消去の問題を適切に解決してください。

```kotlin
// 改善前
fun <T> createList(): List<T> {
    return when (T::class) { // エラー: 型消去により実行時にTの型情報が利用できない
        String::class -> listOf("hello", "world")
        Int::class -> listOf(1, 2, 3)
        else -> emptyList()
    }
}

fun <T> isStringList(list: List<Any>): Boolean {
    return list is List<String> // 警告: 型消去により実行時チェックが不完全
}
```

**回答:**
```kotlin
// 改善後
inline fun <reified T> createList(): List<T> {
    return when (T::class) {
        String::class -> listOf("hello", "world") as List<T>
        Int::class -> listOf(1, 2, 3) as List<T>
        else -> emptyList()
    }
}

inline fun <reified T> List<*>.isListOf(): Boolean {
    return this is List<T>
}

// より実用的な例
inline fun <reified T> List<*>.filterByType(): List<T> {
    return filterIsInstance<T>()
}

inline fun <reified T> createInstance(): T? {
    return try {
        T::class.constructors.firstOrNull()?.call() as? T
    } catch (e: Exception) {
        null
    }
}
```

**理由:**
- reified型パラメータにより、実行時に型情報にアクセス可能
- 型安全なフィルタリングや型チェックが実現
- リフレクションと組み合わせて動的なオブジェクト作成も可能