Great! Let’s break this down into three parts to give you a **complete understanding** of arrays in Ruby, **useful methods**, and then move into **project-style problems** you can build or solve using arrays.

---

## 🧱 1. **Arrays in Ruby: Basics Recap**

An **Array** in Ruby is an **ordered collection** that can hold any type of objects.

```ruby
arr = [1, "hello", :symbol, true]
```

---

## 🛠 2. **Most Useful Array Methods**

| Method              | Description             | Example                         |   |             |
| ------------------- | ----------------------- | ------------------------------- | - | ----------- |
| `push`, `<<`        | Add element             | `arr << 5`                      |   |             |
| `pop`               | Remove last element     | `arr.pop`                       |   |             |
| `shift`, `unshift`  | Remove/add at beginning | `arr.shift`, `arr.unshift("a")` |   |             |
| `map` / `collect`   | Transform all elements  | \`arr.map {                     | x | x \* 2 }\`  |
| `select` / `filter` | Pick matching items     | \`arr.select {                  | x | x.even? }\` |
| `reject`            | Remove matching items   | \`arr.reject {                  | x | x.nil? }\`  |
| `compact`           | Remove `nil` values     | `arr.compact`                   |   |             |
| `uniq`              | Remove duplicates       | `arr.uniq`                      |   |             |
| `sort`              | Sort values             | `arr.sort`                      |   |             |
| `include?`          | Check if contains       | `arr.include?("hello")`         |   |             |
| `each`              | Iterate over elements   | \`arr.each {                    | x | puts x }\`  |

---

## 🔧 3. **Project-Based Array Problems (With Real-World Flavor)**

### ✅ **Project 1: Todo List Manager**

**Problem:** Manage a to-do list with arrays. Add, remove, list, and mark complete.

```ruby
todos = []

def add_task(todos, task)
  todos << { name: task, done: false }
end

def complete_task(todos, index)
  todos[index][:done] = true if todos[index]
end

def list_tasks(todos)
  todos.each_with_index do |task, i|
    status = task[:done] ? "✅" : "❌"
    puts "#{i + 1}. #{task[:name]} - #{status}"
  end
end

add_task(todos, "Buy milk")
add_task(todos, "Clean room")
complete_task(todos, 0)
list_tasks(todos)
```

---

### ✅ **Project 2: Product Filter**

Given an array of hashes (products), filter by price, availability, or name.

```ruby
products = [
  { name: "Drill", price: 100, available: true },
  { name: "Helmet", price: 80, available: false },
  { name: "Nozzle", price: 50, available: true }
]

available = products.select { |p| p[:available] }
cheap = products.select { |p| p[:price] < 90 }

puts "Available Products:"
puts available.map { |p| p[:name] }

puts "Cheap Products:"
puts cheap.map { |p| "#{p[:name]} - $#{p[:price]}" }
```

---

### ✅ **Project 3: Array Merge & Deduplication (Data Clean-up)**

```ruby
emails1 = ["a@example.com", "b@example.com", "c@example.com"]
emails2 = ["b@example.com", "d@example.com"]

all_unique_emails = (emails1 + emails2).uniq
puts all_unique_emails
```

---

### ✅ **Project 4: Simple Analytics – Count Items**

```ruby
orders = ["apple", "banana", "apple", "orange", "banana", "apple"]

counts = orders.tally  # Ruby 2.7+
puts counts  # => {"apple"=>3, "banana"=>2, "orange"=>1}
```

If on older Ruby version, you can use:

```ruby
counts = orders.each_with_object(Hash.new(0)) { |item, hash| hash[item] += 1 }
```

---

## 💡 Want a Challenge?

Create a small **command-line program** using arrays to:

* Store students
* Let user add new student (`name`, `grade`)
* Filter students above grade 75
* List all students alphabetically

Want me to scaffold that project for you?
