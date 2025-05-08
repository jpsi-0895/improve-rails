Here’s a curated list of **important Ruby functions and methods** that every developer should know — especially if you’re doing backend work, scripting, or building apps with Rails.

---

## 🧰 Essential Ruby Functions & Methods

### 🔤 String Methods

| Method                  | Purpose                                   |
| ----------------------- | ----------------------------------------- |
| `.upcase` / `.downcase` | Converts to uppercase/lowercase           |
| `.strip`                | Removes leading/trailing whitespace       |
| `.include?("x")`        | Checks if string contains substring `"x"` |
| `.gsub("a", "b")`       | Replaces all `"a"` with `"b"`             |
| `.split(" ")`           | Splits string into array (by space here)  |

```ruby
"Hello World".downcase  # => "hello world"
" email@example.com ".strip  # => "email@example.com"
```

---

### 🔢 Integer / Numeric Methods

| Method             | Purpose                         |
| ------------------ | ------------------------------- |
| `.to_i`            | Converts to integer             |
| `.even?` / `.odd?` | Checks if number is even or odd |
| `.times`           | Iterates N times                |

```ruby
5.times { puts "Hello" }
"42".to_i + 1  # => 43
```

---

### 🔁 Array Methods

| Method                       | Purpose                                |
| ---------------------------- | -------------------------------------- |
| `.each`                      | Loop through each item                 |
| `.map`                       | Transform each item, returns new array |
| `.select` / `.reject`        | Filters based on condition             |
| `.include?(x)`               | Check if array includes element        |
| `.uniq`, `.sort`, `.reverse` | Self-explanatory list ops              |

```ruby
[1, 2, 3].map { |n| n * 2 }  # => [2, 4, 6]
[1, 2, 2, 3].uniq  # => [1, 2, 3]
```

---

### 📦 Hash Methods

| Method                  | Purpose                       |
| ----------------------- | ----------------------------- |
| `.each`                 | Iterate over key-value pairs  |
| `.keys`, `.values`      | Get all keys or values        |
| `.fetch(:key, default)` | Get key with default fallback |

```ruby
person = { name: "Alice", age: 30 }
person[:name]  # => "Alice"
person.fetch(:email, "N/A")  # => "N/A"
```

---

### 🧠 Core Language Constructs

| Method / Concept  | Purpose                       |
| ----------------- | ----------------------------- |
| `def`             | Define a method               |
| `class`, `module` | Define OOP structure          |
| `begin/rescue`    | Error handling                |
| `require`         | Load external file or library |

```ruby
def greet(name)
  "Hello, #{name}!"
end
```

---

### 🧪 Useful Standard Library Methods

| Function         | Module   | Purpose           |
| ---------------- | -------- | ----------------- |
| `Time.now`       | `Time`   | Current time      |
| `File.read("x")` | `File`   | Read file content |
| `rand`           | `Kernel` | Random number     |
| `puts`, `gets`   | `Kernel` | I/O               |

```ruby
puts "What's your name?"
name = gets.chomp
puts "Hello, #{name}!"
```

---

Would you like this as a cheat sheet or code snippet file?
