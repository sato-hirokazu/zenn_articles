---
title: "基本文法"
---

## リーダブルコード問題

### 問題1: 変数名の改善
以下のコードをより読みやすく改善してください。

```kotlin
// 改善前
fun calc(a: Int, b: Int, c: Boolean): Int {
    val d = if (c) a + b else a - b
    return d * 2
}
```

**回答:**
```kotlin
// 改善後
fun calculateResult(firstNumber: Int, secondNumber: Int, shouldAdd: Boolean): Int {
    val result = if (shouldAdd) firstNumber + secondNumber else firstNumber - secondNumber
    return result * 2
}
```

**理由:**
- 変数名を意味のある名前に変更
- 関数名をより具体的に
- 一時変数にも意味のある名前を付与

### 問題2: when式の活用
以下のif-else文をwhen式に変更して、より読みやすくしてください。

```kotlin
// 改善前
fun getGrade(score: Int): String {
    if (score >= 90) {
        return "A"
    } else if (score >= 80) {
        return "B"
    } else if (score >= 70) {
        return "C"
    } else if (score >= 60) {
        return "D"
    } else {
        return "F"
    }
}
```

**回答:**
```kotlin
// 改善後
fun getGrade(score: Int): String = when {
    score >= 90 -> "A"
    score >= 80 -> "B"
    score >= 70 -> "C"
    score >= 60 -> "D"
    else -> "F"
}
```

**理由:**
- 条件分岐がより明確
- 式として記述することで簡潔性を向上
- 各条件が並列に並ぶことで可読性が向上

### 問題3: 文字列テンプレートの活用
以下の文字列連結を文字列テンプレートに変更してください。

```kotlin
// 改善前
fun createMessage(name: String, age: Int, city: String): String {
    return "Hello, " + name + "! You are " + age + " years old and live in " + city + "."
}
```

**回答:**
```kotlin
// 改善後
fun createMessage(name: String, age: Int, city: String): String {
    return "Hello, $name! You are $age years old and live in $city."
}
```

**理由:**
- 文字列テンプレートを使用することで簡潔性を向上
- 文字列連結の`+`演算子が不要になり、コードが読みやすくなる
- 文字列内での変数の位置が明確になる

### 問題4: 定数の適切な使用
以下のマジックナンバーを定数に置き換えて、より読みやすくしてください。

```kotlin
// 改善前
fun calculateDiscount(price: Double, isMember: Boolean): Double {
    if (isMember) {
        return price * 0.15
    } else {
        return price * 0.05
    }
}

fun isHighPriority(score: Int): Boolean {
    return score >= 80
}
```

**回答:**
```kotlin
// 改善後
object DiscountRates {
    const val MEMBER_DISCOUNT = 0.15
    const val REGULAR_DISCOUNT = 0.05
}

object PriorityThresholds {
    const val HIGH_PRIORITY_SCORE = 80
}

fun calculateDiscount(price: Double, isMember: Boolean): Double {
    val discountRate = if (isMember) DiscountRates.MEMBER_DISCOUNT else DiscountRates.REGULAR_DISCOUNT
    return price * discountRate
}

fun isHighPriority(score: Int): Boolean {
    return score >= PriorityThresholds.HIGH_PRIORITY_SCORE
}
```

**理由:**
- マジックナンバーを定数に置き換えることで意図が明確になる
- 定数の変更が一箇所で済むため、保守性が向上
- コードの可読性が大幅に向上

### 問題5: 条件式の簡潔化
以下の複雑な条件式を簡潔にしてください。

```kotlin
// 改善前
fun canAccess(user: User): Boolean {
    if (user.isActive) {
        if (user.hasPermission) {
            if (user.age >= 18) {
                return true
            } else {
                return false
            }
        } else {
            return false
        }
    } else {
        return false
    }
}
```

**回答:**
```kotlin
// 改善後
fun canAccess(user: User): Boolean {
    return user.isActive && user.hasPermission && user.age >= 18
}
```

**理由:**
- ネストしたif文を論理演算子で簡潔に表現
- 条件の組み合わせが一目で理解できる
- コードの行数が大幅に削減

### 問題6: 範囲チェックの改善
以下の範囲チェックをKotlinの範囲演算子を使って改善してください。

```kotlin
// 改善前
fun isValidAge(age: Int): Boolean {
    if (age >= 0 && age <= 120) {
        return true
    } else {
        return false
    }
}

fun getGradeLevel(score: Int): String {
    if (score >= 90 && score <= 100) {
        return "A"
    } else if (score >= 80 && score < 90) {
        return "B"
    } else if (score >= 70 && score < 80) {
        return "C"
    } else {
        return "F"
    }
}
```

**回答:**
```kotlin
// 改善後
fun isValidAge(age: Int): Boolean {
    return age in 0..120
}

fun getGradeLevel(score: Int): String {
    return when (score) {
        in 90..100 -> "A"
        in 80..89 -> "B"
        in 70..79 -> "C"
        else -> "F"
    }
}
```

**理由:**
- 範囲演算子（`in`）を使用して条件を簡潔に表現
- when式と組み合わせることで、範囲チェックが読みやすくなる
- コードの意図がより明確になる
