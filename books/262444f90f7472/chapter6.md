---
title: "拡張関数・拡張プロパティ"
---

## リーダブルコード問題

### 問題1: 拡張関数の適切な使用
以下のユーティリティ関数を拡張関数に変更して、より読みやすくしてください。

```kotlin
// 改善前
object StringUtils {
    fun isEmail(text: String): Boolean {
        return text.contains("@") && text.contains(".")
    }
    
    fun capitalizeWords(text: String): String {
        return text.split(" ").joinToString(" ") { it.capitalize() }
    }
    
    fun removeWhitespace(text: String): String {
        return text.replace("\\s+".toRegex(), "")
    }
}

// 使用例
val email = "user@example.com"
if (StringUtils.isEmail(email)) {
    val processed = StringUtils.capitalizeWords(StringUtils.removeWhitespace(email))
}
```

**回答:**
```kotlin
// 改善後
fun String.isEmail(): Boolean {
    return contains("@") && contains(".")
}

fun String.capitalizeWords(): String {
    return split(" ").joinToString(" ") { it.capitalize() }
}

fun String.removeWhitespace(): String {
    return replace("\\s+".toRegex(), "")
}

// 使用例
val email = "user@example.com"
if (email.isEmail()) {
    val processed = email.removeWhitespace().capitalizeWords()
}
```

**理由:**
- 拡張関数により、オブジェクトのメソッドのように自然に呼び出し可能
- メソッドチェーンが可能になり、処理の流れが読みやすくなる
- 静的ユーティリティクラスが不要になり、コードが簡潔になる

### 問題2: 拡張プロパティの活用
以下の計算処理を拡張プロパティに変更してください。

```kotlin
// 改善前
class DateUtils {
    companion object {
        fun isWeekend(date: LocalDate): Boolean {
            val dayOfWeek = date.dayOfWeek
            return dayOfWeek == DayOfWeek.SATURDAY || dayOfWeek == DayOfWeek.SUNDAY
        }
        
        fun getDaysUntilEndOfMonth(date: LocalDate): Int {
            val lastDayOfMonth = date.withDayOfMonth(date.lengthOfMonth())
            return ChronoUnit.DAYS.between(date, lastDayOfMonth).toInt()
        }
        
        fun getFormattedDate(date: LocalDate): String {
            return date.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"))
        }
    }
}
```

**回答:**
```kotlin
// 改善後
val LocalDate.isWeekend: Boolean
    get() = dayOfWeek == DayOfWeek.SATURDAY || dayOfWeek == DayOfWeek.SUNDAY

val LocalDate.daysUntilEndOfMonth: Int
    get() = ChronoUnit.DAYS.between(this, withDayOfMonth(lengthOfMonth())).toInt()

val LocalDate.formatted: String
    get() = format(DateTimeFormatter.ofPattern("yyyy-MM-dd"))

// 使用例
val today = LocalDate.now()
if (today.isWeekend) {
    println("Today is weekend! ${today.formatted}")
    println("Days until end of month: ${today.daysUntilEndOfMonth}")
}
```

**理由:**
- 計算処理をプロパティとして表現することで、より自然なAPIになる
- プロパティアクセスのように見えて、実際は計算処理が実行される
- コードがより読みやすく、意図が明確になる

### 問題3: 高階関数を活用した拡張関数
以下のコレクション操作を高階関数を活用した拡張関数に変更してください。

```kotlin
// 改善前
fun processNumbers(numbers: List<Int>): List<String> {
    val result = mutableListOf<String>()
    for (number in numbers) {
        if (number > 0) {
            val doubled = number * 2
            if (doubled > 10) {
                result.add("High: $doubled")
            } else {
                result.add("Low: $doubled")
            }
        }
    }
    return result
}

fun findFirstEven(numbers: List<Int>): Int? {
    for (number in numbers) {
        if (number % 2 == 0) {
            return number
        }
    }
    return null
}
```

**回答:**
```kotlin
// 改善後
fun <T> List<T>.filterMap(
    predicate: (T) -> Boolean,
    transform: (T) -> T
): List<T> {
    return filter(predicate).map(transform)
}

fun <T> List<T>.findFirst(predicate: (T) -> Boolean): T? {
    return firstOrNull(predicate)
}

fun List<Int>.processNumbers(): List<String> {
    return filterMap(
        predicate = { it > 0 },
        transform = { it * 2 }
    ).map { doubled ->
        if (doubled > 10) "High: $doubled" else "Low: $doubled"
    }
}

fun List<Int>.findFirstEven(): Int? {
    return findFirst { it % 2 == 0 }
}

// 使用例
val numbers = listOf(1, 2, 3, 4, 5, -1, 6)
val processed = numbers.processNumbers()
val firstEven = numbers.findFirstEven()
```

**理由:**
- 高階関数により、処理のロジックを柔軟にカスタマイズ可能
- 拡張関数として定義することで、コレクションのメソッドのように使用可能
- 再利用可能な汎用的な関数を作成

### 問題4: 型安全な拡張関数
以下の型安全でない拡張関数を型安全にしてください。

```kotlin
// 改善前
fun <T> List<T>.getOrThrow(index: Int): T {
    if (index < 0 || index >= size) {
        throw IndexOutOfBoundsException("Index $index is out of bounds")
    }
    return this[index]
}

fun String.toIntOrThrow(): Int {
    return try {
        toInt()
    } catch (e: NumberFormatException) {
        throw IllegalArgumentException("Cannot convert '$this' to Int", e)
    }
}
```

**回答:**
```kotlin
// 改善後
sealed class SafeResult<out T> {
    data class Success<T>(val value: T) : SafeResult<T>()
    data class Error(val message: String, val cause: Throwable? = null) : SafeResult<Nothing>()
}

fun <T> List<T>.getSafe(index: Int): SafeResult<T> {
    return if (index < 0 || index >= size) {
        SafeResult.Error("Index $index is out of bounds for list of size $size")
    } else {
        SafeResult.Success(this[index])
    }
}

fun String.toIntSafe(): SafeResult<Int> {
    return try {
        SafeResult.Success(toInt())
    } catch (e: NumberFormatException) {
        SafeResult.Error("Cannot convert '$this' to Int", e)
    }
}

// 使用例
val numbers = listOf(1, 2, 3)
val result = numbers.getSafe(1)
when (result) {
    is SafeResult.Success -> println("Value: ${result.value}")
    is SafeResult.Error -> println("Error: ${result.message}")
}

val stringResult = "123".toIntSafe()
when (stringResult) {
    is SafeResult.Success -> println("Number: ${stringResult.value}")
    is SafeResult.Error -> println("Error: ${stringResult.message}")
}
```

**理由:**
- 例外を投げる代わりに、結果を型で表現することで型安全性を向上
- 呼び出し側でエラーハンドリングを明示的に行う必要がある
- コンパイル時にエラーハンドリングの漏れを検出可能

### 問題5: 拡張関数のチェーン化
以下の複雑な文字列処理を拡張関数のチェーンで改善してください。

```kotlin
// 改善前
fun processText(text: String): String {
    val trimmed = text.trim()
    val withoutSpaces = trimmed.replace(" ", "")
    val lowercased = withoutSpaces.lowercase()
    val withoutNumbers = lowercased.replace(Regex("[0-9]"), "")
    val reversed = withoutNumbers.reversed()
    return reversed
}

fun validateEmail(email: String): Boolean {
    val trimmed = email.trim()
    val lowercased = trimmed.lowercase()
    val hasAt = lowercased.contains("@")
    val hasDot = lowercased.contains(".")
    val notEmpty = lowercased.isNotEmpty()
    return hasAt && hasDot && notEmpty
}
```

**回答:**
```kotlin
// 改善後
fun String.removeSpaces(): String = replace(" ", "")

fun String.removeNumbers(): String = replace(Regex("[0-9]"), "")

fun String.isValidEmail(): Boolean {
    return trim()
        .lowercase()
        .let { it.isNotEmpty() && it.contains("@") && it.contains(".") }
}

fun String.processText(): String {
    return trim()
        .removeSpaces()
        .lowercase()
        .removeNumbers()
        .reversed()
}

// 使用例
val processed = "Hello World 123".processText()
val isValid = "user@example.com".isValidEmail()
```

**理由:**
- 拡張関数のチェーンにより、処理の流れが明確になる
- 各処理が独立した関数として再利用可能
- コードの可読性と保守性が向上

### 問題6: カスタム拡張プロパティ
以下の計算処理をカスタム拡張プロパティに変更してください。

```kotlin
// 改善前
fun getFileSizeFormatted(bytes: Long): String {
    val kb = bytes / 1024.0
    val mb = kb / 1024.0
    val gb = mb / 1024.0
    
    return when {
        gb >= 1 -> String.format("%.2f GB", gb)
        mb >= 1 -> String.format("%.2f MB", mb)
        kb >= 1 -> String.format("%.2f KB", kb)
        else -> "$bytes bytes"
    }
}

fun isLeapYear(year: Int): Boolean {
    return year % 4 == 0 && (year % 100 != 0 || year % 400 == 0)
}
```

**回答:**
```kotlin
// 改善後
val Long.formattedSize: String
    get() {
        val kb = this / 1024.0
        val mb = kb / 1024.0
        val gb = mb / 1024.0
        
        return when {
            gb >= 1 -> String.format("%.2f GB", gb)
            mb >= 1 -> String.format("%.2f MB", mb)
            kb >= 1 -> String.format("%.2f KB", kb)
            else -> "$this bytes"
        }
    }

val Int.isLeapYear: Boolean
    get() = this % 4 == 0 && (this % 100 != 0 || this % 400 == 0)

// 使用例
val fileSize = 1024000L.formattedSize // "1000.00 KB"
val isLeap = 2024.isLeapYear // true
```

**理由:**
- 拡張プロパティにより、計算処理を自然なプロパティアクセスで表現
- コードがより読みやすく、意図が明確になる
- 型に特化した機能を提供