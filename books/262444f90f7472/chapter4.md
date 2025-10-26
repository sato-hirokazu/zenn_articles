---
title: "Null安全とSmart Cast"
---

## リーダブルコード問題

### 問題1: 安全なnullチェックの実装
以下のunsafeなnullチェックをKotlinのnull安全機能を使って安全にしてください。

```kotlin
// 改善前
fun processUser(user: User?): String {
    if (user != null) {
        if (user.name != null) {
            if (user.email != null) {
                return "Name: ${user.name}, Email: ${user.email}"
            } else {
                return "Name: ${user.name}, Email: Not provided"
            }
        } else {
            return "Name: Not provided"
        }
    } else {
        return "User not found"
    }
}

data class User(val name: String?, val email: String?)
```

**回答:**
```kotlin
// 改善後
fun processUser(user: User?): String {
    return when {
        user == null -> "User not found"
        user.name == null -> "Name: Not provided"
        user.email == null -> "Name: ${user.name}, Email: Not provided"
        else -> "Name: ${user.name}, Email: ${user.email}"
    }
}

// または、より簡潔に
fun processUser(user: User?): String {
    return user?.let { u ->
        val name = u.name ?: "Not provided"
        val email = u.email ?: "Not provided"
        "Name: $name, Email: $email"
    } ?: "User not found"
}
```

**理由:**
- ネストしたnullチェックをwhen式で簡潔に表現
- Elvis演算子（`?:`）を使用してデフォルト値を設定
- let関数を使用してnullでない場合の処理を明確に分離

### 問題2: Smart Castの活用
以下の型チェックとキャストをSmart Castを使って簡潔にしてください。

```kotlin
// 改善前
fun processAnimal(animal: Animal): String {
    if (animal is Dog) {
        val dog = animal as Dog
        return "Dog: ${dog.name}, Breed: ${dog.breed}"
    } else if (animal is Cat) {
        val cat = animal as Cat
        return "Cat: ${cat.name}, Color: ${cat.color}"
    } else {
        return "Unknown animal"
    }
}

sealed class Animal
data class Dog(val name: String, val breed: String) : Animal()
data class Cat(val name: String, val color: String) : Animal()
```

**回答:**
```kotlin
// 改善後
fun processAnimal(animal: Animal): String {
    return when (animal) {
        is Dog -> "Dog: ${animal.name}, Breed: ${animal.breed}"
        is Cat -> "Cat: ${animal.name}, Color: ${animal.color}"
        else -> "Unknown animal"
    }
}
```

**理由:**
- Smart Castにより明示的なキャストが不要
- when式で型チェックと処理を一箇所に集約
- 各分岐で自動的にキャストされた型として使用可能

### 問題3: 安全なコレクション操作
以下のnullを含む可能性のあるコレクション操作を安全にしてください。

```kotlin
// 改善前
fun processNumbers(numbers: List<Int?>): List<String> {
    val result = mutableListOf<String>()
    for (number in numbers) {
        if (number != null) {
            if (number > 0) {
                result.add("Positive: $number")
            } else {
                result.add("Zero or negative: $number")
            }
        } else {
            result.add("Null value")
        }
    }
    return result
}
```

**回答:**
```kotlin
// 改善後
fun processNumbers(numbers: List<Int?>): List<String> {
    return numbers.map { number ->
        when {
            number == null -> "Null value"
            number > 0 -> "Positive: $number"
            else -> "Zero or negative: $number"
        }
    }
}

// または、nullを除外して処理
fun processValidNumbers(numbers: List<Int?>): List<String> {
    return numbers
        .filterNotNull()
        .map { number ->
            if (number > 0) "Positive: $number" else "Zero or negative: $number"
        }
}
```

**理由:**
- 命令的なループを関数型のアプローチに変更
- filterNotNull()を使用してnullを安全に除外
- when式で条件分岐を明確に表現

### 問題4: 安全なプロパティアクセス
以下の深いネストしたプロパティアクセスを安全にしてください。

```kotlin
// 改善前
fun getAddress(user: User?): String {
    if (user != null) {
        if (user.profile != null) {
            if (user.profile.address != null) {
                if (user.profile.address.street != null) {
                    return user.profile.address.street
                } else {
                    return "Street not provided"
                }
            } else {
                return "Address not provided"
            }
        } else {
            return "Profile not provided"
        }
    } else {
        return "User not found"
    }
}

data class User(val profile: Profile?)
data class Profile(val address: Address?)
data class Address(val street: String?)
```

**回答:**
```kotlin
// 改善後
fun getAddress(user: User?): String {
    return user?.profile?.address?.street ?: "Address not available"
}

// より詳細なエラーメッセージが必要な場合
fun getAddressWithDetails(user: User?): String {
    return when {
        user == null -> "User not found"
        user.profile == null -> "Profile not provided"
        user.profile.address == null -> "Address not provided"
        user.profile.address.street == null -> "Street not provided"
        else -> user.profile.address.street
    }
}
```

**理由:**
- 安全呼び出し演算子（`?.`）を使用してネストしたnullチェックを簡潔に実装
- Elvis演算子（`?:`）でデフォルト値を設定
- 詳細なエラーメッセージが必要な場合はwhen式で段階的にチェック

### 問題5: エルビス演算子の活用
以下のnullチェック処理をエルビス演算子を使って簡潔にしてください。

```kotlin
// 改善前
fun getDisplayName(user: User?): String {
    if (user == null) {
        return "Guest"
    } else {
        if (user.displayName == null) {
            return user.username
        } else {
            return user.displayName
        }
    }
}

fun getConfigValue(config: Config?, key: String): String {
    if (config == null) {
        return "default"
    } else {
        val value = config.getValue(key)
        if (value == null) {
            return "default"
        } else {
            return value
        }
    }
}
```

**回答:**
```kotlin
// 改善後
fun getDisplayName(user: User?): String {
    return user?.displayName ?: user?.username ?: "Guest"
}

fun getConfigValue(config: Config?, key: String): String {
    return config?.getValue(key) ?: "default"
}
```

**理由:**
- エルビス演算子（`?:`）でnullの場合のデフォルト値を簡潔に設定
- ネストしたif文を削減し、可読性を向上
- チェーン形式で複数のnullチェックを一度に処理

### 問題6: 安全な型変換
以下のunsafeな型変換を安全にしてください。

```kotlin
// 改善前
fun processValue(value: Any?): String {
    if (value == null) {
        return "null"
    }
    
    if (value is String) {
        return value.uppercase()
    } else if (value is Int) {
        return value.toString()
    } else if (value is Double) {
        return String.format("%.2f", value)
    } else {
        return "unknown"
    }
}

fun getNumber(value: Any?): Int {
    if (value is String) {
        return value.toInt() // 例外が発生する可能性
    } else if (value is Int) {
        return value
    } else {
        return 0
    }
}
```

**回答:**
```kotlin
// 改善後
fun processValue(value: Any?): String {
    return when (value) {
        null -> "null"
        is String -> value.uppercase()
        is Int -> value.toString()
        is Double -> String.format("%.2f", value)
        else -> "unknown"
    }
}

fun getNumber(value: Any?): Int {
    return when (value) {
        is String -> value.toIntOrNull() ?: 0
        is Int -> value
        else -> 0
    }
}
```

**理由:**
- when式を使用して型チェックを簡潔に実装
- toIntOrNull()を使用して安全な型変換を実現
- 例外の発生を防止し、より堅牢なコードになる
