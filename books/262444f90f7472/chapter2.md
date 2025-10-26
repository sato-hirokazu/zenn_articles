---
title: "関数とスコープ"
---

## リーダブルコード問題

### 問題1: 関数の分割と責任の明確化
以下の長い関数を複数の小さな関数に分割して、より読みやすくしてください。

```kotlin
// 改善前
fun processUserData(users: List<String>): List<String> {
    val result = mutableListOf<String>()
    for (user in users) {
        if (user.isNotEmpty()) {
            val trimmed = user.trim()
            if (trimmed.length >= 3) {
                val capitalized = trimmed.capitalize()
                if (capitalized.contains("@")) {
                    result.add(capitalized)
                }
            }
        }
    }
    return result
}
```

**回答:**
```kotlin
// 改善後
fun processUserData(users: List<String>): List<String> {
    return users
        .filter { isValidUser(it) }
        .map { formatUser(it) }
}

private fun isValidUser(user: String): Boolean {
    return user.isNotEmpty() && 
           user.trim().length >= 3 && 
           user.trim().contains("@")
}

private fun formatUser(user: String): String {
    return user.trim().capitalize()
}
```

**理由:**
- 単一責任の原則に従い、各関数が一つの責任を持つように分割
- 関数名で処理内容が明確になる
- ネストが浅くなり、可読性が向上
- テストしやすい単位に分割

### 問題2: デフォルト引数の活用
以下のオーバーロードされた関数をデフォルト引数を使って簡潔にしてください。

```kotlin
// 改善前
fun createMessage(name: String): String {
    return "Hello, $name!"
}

fun createMessage(name: String, title: String): String {
    return "Hello, $title $name!"
}

fun createMessage(name: String, title: String, suffix: String): String {
    return "Hello, $title $name! $suffix"
}
```

**回答:**
```kotlin
// 改善後
fun createMessage(
    name: String, 
    title: String = "", 
    suffix: String = ""
): String {
    val titlePart = if (title.isNotEmpty()) "$title " else ""
    val suffixPart = if (suffix.isNotEmpty()) " $suffix" else ""
    return "Hello, ${titlePart}${name}!${suffixPart}"
}
```

**理由:**
- オーバーロードを削減し、コードの重複を排除
- デフォルト引数により、必要な場合のみパラメータを指定
- 関数の実装が一箇所に集約され、保守性が向上

### 問題3: ローカル関数の活用
以下の関数内で重複している処理をローカル関数として抽出してください。

```kotlin
// 改善前
fun validateUserInput(name: String, email: String, age: String): List<String> {
    val errors = mutableListOf<String>()
    
    if (name.isBlank()) {
        errors.add("Name cannot be blank")
    } else if (name.length < 2) {
        errors.add("Name must be at least 2 characters")
    }
    
    if (email.isBlank()) {
        errors.add("Email cannot be blank")
    } else if (!email.contains("@")) {
        errors.add("Email must contain @")
    }
    
    if (age.isBlank()) {
        errors.add("Age cannot be blank")
    } else if (age.toIntOrNull() == null) {
        errors.add("Age must be a number")
    }
    
    return errors
}
```

**回答:**
```kotlin
// 改善後
fun validateUserInput(name: String, email: String, age: String): List<String> {
    val errors = mutableListOf<String>()
    
    fun addErrorIf(condition: Boolean, message: String) {
        if (condition) errors.add(message)
    }
    
    addErrorIf(name.isBlank(), "Name cannot be blank")
    addErrorIf(name.isNotBlank() && name.length < 2, "Name must be at least 2 characters")
    
    addErrorIf(email.isBlank(), "Email cannot be blank")
    addErrorIf(email.isNotBlank() && !email.contains("@"), "Email must contain @")
    
    addErrorIf(age.isBlank(), "Age cannot be blank")
    addErrorIf(age.isNotBlank() && age.toIntOrNull() == null, "Age must be a number")
    
    return errors
}
```

**理由:**
- 重複していた条件チェックとエラー追加の処理をローカル関数として抽出
- コードの重複を削減し、保守性を向上
- ローカル関数により、関数内でのみ使用される処理を適切にスコープ化

### 問題4: 高階関数の活用
以下の重複した処理を高階関数を使って簡潔にしてください。

```kotlin
// 改善前
fun processNumbers(numbers: List<Int>): List<String> {
    val result = mutableListOf<String>()
    for (number in numbers) {
        if (number > 0) {
            result.add("Positive: $number")
        }
    }
    return result
}

fun processStrings(strings: List<String>): List<String> {
    val result = mutableListOf<String>()
    for (string in strings) {
        if (string.length > 3) {
            result.add("Long: $string")
        }
    }
    return result
}
```

**回答:**
```kotlin
// 改善後
fun <T> List<T>.filterAndTransform(
    predicate: (T) -> Boolean,
    transform: (T) -> String
): List<String> {
    return filter(predicate).map(transform)
}

fun processNumbers(numbers: List<Int>): List<String> {
    return numbers.filterAndTransform(
        predicate = { it > 0 },
        transform = { "Positive: $it" }
    )
}

fun processStrings(strings: List<String>): List<String> {
    return strings.filterAndTransform(
        predicate = { it.length > 3 },
        transform = { "Long: $it" }
    )
}
```

**理由:**
- 高階関数により、重複した処理ロジックを抽象化
- 汎用的な関数を作成し、再利用性を向上
- コードの重複を削減し、保守性を向上

### 問題5: 関数のオーバーロードの整理
以下のオーバーロードされた関数をより簡潔に整理してください。

```kotlin
// 改善前
fun createUser(name: String): User {
    return User(name, null, null, null)
}

fun createUser(name: String, email: String): User {
    return User(name, email, null, null)
}

fun createUser(name: String, email: String, age: Int): User {
    return User(name, email, age, null)
}

fun createUser(name: String, email: String, age: Int, phone: String): User {
    return User(name, email, age, phone)
}
```

**回答:**
```kotlin
// 改善後
data class User(
    val name: String,
    val email: String? = null,
    val age: Int? = null,
    val phone: String? = null
) {
    companion object {
        fun create(name: String) = User(name)
        fun create(name: String, email: String) = User(name, email)
        fun create(name: String, email: String, age: Int) = User(name, email, age)
        fun create(name: String, email: String, age: Int, phone: String) = User(name, email, age, phone)
    }
}
```

**理由:**
- データクラスのデフォルト引数により、オーバーロードを削減
- ファクトリーメソッドパターンでより明確なAPIを提供
- コードの重複を大幅に削減

### 問題6: 再帰関数の最適化
以下の再帰関数を末尾再帰に最適化してください。

```kotlin
// 改善前
fun factorial(n: Int): Int {
    if (n <= 1) {
        return 1
    } else {
        return n * factorial(n - 1)
    }
}

fun sumList(numbers: List<Int>): Int {
    if (numbers.isEmpty()) {
        return 0
    } else {
        return numbers.first() + sumList(numbers.drop(1))
    }
}
```

**回答:**
```kotlin
// 改善後
fun factorial(n: Int): Int {
    tailrec fun factorialHelper(n: Int, acc: Int = 1): Int {
        return if (n <= 1) acc else factorialHelper(n - 1, acc * n)
    }
    return factorialHelper(n)
}

fun sumList(numbers: List<Int>): Int {
    tailrec fun sumHelper(remaining: List<Int>, acc: Int = 0): Int {
        return if (remaining.isEmpty()) acc else sumHelper(remaining.drop(1), acc + remaining.first())
    }
    return sumHelper(numbers)
}
```

**理由:**
- 末尾再帰により、スタックオーバーフローを防止
- コンパイラによる最適化が可能
- 大きな値でも安全に処理できる
