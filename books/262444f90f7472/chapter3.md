---
title: "コレクション操作"
---

## リーダブルコード問題

### 問題1: コレクション操作のチェーン化
以下のネストしたループをコレクション操作のチェーンに変更して、より読みやすくしてください。

```kotlin
// 改善前
fun processUsers(users: List<User>): List<String> {
    val result = mutableListOf<String>()
    for (user in users) {
        if (user.isActive) {
            for (order in user.orders) {
                if (order.total > 100) {
                    result.add("${user.name}: ${order.productName}")
                }
            }
        }
    }
    return result
}

data class User(val name: String, val isActive: Boolean, val orders: List<Order>)
data class Order(val productName: String, val total: Int)
```

**回答:**
```kotlin
// 改善後
fun processUsers(users: List<User>): List<String> {
    return users
        .filter { it.isActive }
        .flatMap { user -> 
            user.orders
                .filter { order -> order.total > 100 }
                .map { order -> "${user.name}: ${order.productName}" }
        }
}
```

**理由:**
- ネストしたループを関数型のアプローチに変更
- 各操作の意図が明確になる
- コードの流れが上から下に読みやすくなる
- 副作用のない純粋な関数として実装

### 問題2: groupByの適切な使用
以下の複雑なグルーピング処理をgroupByを使って簡潔にしてください。

```kotlin
// 改善前
fun groupProductsByCategory(products: List<Product>): Map<String, List<Product>> {
    val result = mutableMapOf<String, MutableList<Product>>()
    for (product in products) {
        val category = product.category
        if (!result.containsKey(category)) {
            result[category] = mutableListOf()
        }
        result[category]?.add(product)
    }
    return result
}

data class Product(val name: String, val category: String, val price: Int)
```

**回答:**
```kotlin
// 改善後
fun groupProductsByCategory(products: List<Product>): Map<String, List<Product>> {
    return products.groupBy { it.category }
}
```

**理由:**
- 複雑な手動グルーピング処理をgroupByで簡潔に実装
- コードの行数が大幅に削減
- 意図が明確になり、バグの発生リスクを削減

### 問題3: foldの活用
以下の合計計算をfoldを使って実装し、より柔軟性を持たせてください。

```kotlin
// 改善前
fun calculateTotalPrice(products: List<Product>): Int {
    var total = 0
    for (product in products) {
        total += product.price
    }
    return total
}
```

**回答:**
```kotlin
// 改善後
fun calculateTotalPrice(products: List<Product>): Int {
    return products.fold(0) { acc, product -> acc + product.price }
}

// より柔軟なバージョン
fun calculateTotalPriceWithDiscount(
    products: List<Product>, 
    discountRate: Double = 0.0
): Double {
    return products.fold(0.0) { acc, product -> 
        acc + (product.price * (1 - discountRate))
    }
}
```

**理由:**
- 命令的なループを宣言的なfoldに変更
- 初期値とアキュムレータの概念が明確
- より柔軟な計算ロジックを追加しやすい

### 問題4: 複雑なコレクション操作の分割
以下の複雑なコレクション操作を複数のステップに分割して、読みやすくしてください。

```kotlin
// 改善前
fun getTopCustomers(customers: List<Customer>): List<String> {
    return customers
        .filter { it.orders.isNotEmpty() }
        .map { customer -> 
            customer to customer.orders.sumBy { it.total } 
        }
        .sortedByDescending { it.second }
        .take(5)
        .map { it.first.name }
}
```

**回答:**
```kotlin
// 改善後
fun getTopCustomers(customers: List<Customer>): List<String> {
    val customersWithOrders = customers.filter { it.orders.isNotEmpty() }
    val customersWithTotalSpent = customersWithOrders.map { customer ->
        val totalSpent = customer.orders.sumBy { it.total }
        customer to totalSpent
    }
    val sortedCustomers = customersWithTotalSpent.sortedByDescending { it.second }
    val topCustomers = sortedCustomers.take(5)
    return topCustomers.map { it.first.name }
}

// または、意味のある関数に分割
fun getTopCustomers(customers: List<Customer>): List<String> {
    return customers
        .filter(::hasOrders)
        .map(::calculateTotalSpent)
        .sortedByDescending { it.second }
        .take(5)
        .map { it.first.name }
}

private fun hasOrders(customer: Customer): Boolean = customer.orders.isNotEmpty()

private fun calculateTotalSpent(customer: Customer): Pair<Customer, Int> {
    val totalSpent = customer.orders.sumBy { it.total }
    return customer to totalSpent
}
```

**理由:**
- 複雑なチェーンを中間変数に分割して可読性を向上
- 各ステップの意図が明確になる
- デバッグ時に中間結果を確認しやすい

### 問題5: コレクションの変換とフィルタリング
以下の複雑なデータ変換処理を読みやすく改善してください。

```kotlin
// 改善前
fun processOrders(orders: List<Order>): Map<String, Double> {
    val result = mutableMapOf<String, Double>()
    for (order in orders) {
        if (order.status == "completed") {
            val category = order.product.category
            val total = order.quantity * order.product.price
            if (result.containsKey(category)) {
                result[category] = result[category]!! + total
            } else {
                result[category] = total
            }
        }
    }
    return result
}
```

**回答:**
```kotlin
// 改善後
fun processOrders(orders: List<Order>): Map<String, Double> {
    return orders
        .filter { it.status == "completed" }
        .groupBy { it.product.category }
        .mapValues { (_, orders) ->
            orders.sumOf { it.quantity * it.product.price }
        }
}
```

**理由:**
- 命令的なループを関数型のアプローチに変更
- groupByとmapValuesを組み合わせて簡潔に実装
- ネストした条件分岐を削減し、可読性を向上

### 問題6: コレクションの並び替えと重複除去
以下の複雑な並び替え処理を改善してください。

```kotlin
// 改善前
fun getTopStudents(students: List<Student>): List<String> {
    val uniqueStudents = mutableSetOf<String>()
    val studentScores = mutableListOf<Pair<String, Int>>()
    
    for (student in students) {
        if (!uniqueStudents.contains(student.name)) {
            uniqueStudents.add(student.name)
            studentScores.add(Pair(student.name, student.score))
        }
    }
    
    studentScores.sortByDescending { it.second }
    
    val result = mutableListOf<String>()
    for (i in 0 until minOf(3, studentScores.size)) {
        result.add(studentScores[i].first)
    }
    return result
}
```

**回答:**
```kotlin
// 改善後
fun getTopStudents(students: List<Student>): List<String> {
    return students
        .distinctBy { it.name }
        .sortedByDescending { it.score }
        .take(3)
        .map { it.name }
}
```

**理由:**
- distinctByを使用して重複除去を簡潔に実装
- sortedByDescendingで並び替えを明確に表現
- takeで上位N件の取得を簡潔に実装
