Mastering **loops in Ruby** goes beyond just using `for` and `while`. Ruby provides **elegant, expressive loop constructs** that can make your code cleaner, faster, and more "Rubyish." Here’s a guide to some **exceptional Ruby looping techniques** every developer should know:

---

## 🔁 1. `each` – The Go-To Loop

```ruby
[1, 2, 3].each do |num|
  puts num
end
```

✅ Clean and idiomatic for iterating over arrays, hashes, etc.

---

## 🔂 2. `times` – Loop N Times

```ruby
5.times { puts "Hello" }
```

✅ Useful when you need a fixed number of iterations.

---

## 🔢 3. `upto` / `downto`

```ruby
1.upto(5) { |n| puts n }
5.downto(1) { |n| puts n }
```

✅ More readable than manual loop counters.

---

## 💡 4. `map` – Transform a Collection

```ruby
squared = [1, 2, 3].map { |n| n * n }
```

✅ Returns a **new array** with transformed values.

---

## 🎯 5. `select` / `reject` – Filtering

```ruby
evens = [1, 2, 3, 4].select(&:even?)   # => [2, 4]
odds = [1, 2, 3, 4].reject(&:even?)    # => [1, 3]
```

✅ Great for **filtering** collections.

---

## 🚦 6. `while` / `until` – Control Flow Loops

```ruby
i = 0
while i < 3
  puts i
  i += 1
end

x = 0
until x == 3
  puts x
  x += 1
end
```

---

## 🧨 7. `loop` with `break` and `next`

```ruby
i = 0
loop do
  i += 1
  next if i == 2
  break if i > 4
  puts i
end
```

✅ Gives full control; use `next` to skip, `break` to exit.

---

## 💎 8. Hash Looping

```ruby
{a: 1, b: 2}.each do |key, value|
  puts "#{key} => #{value}"
end
```

---

## 🧙 9. Enumerator Chaining (Advanced)

```ruby
(1..10).select(&:even?).map { |n| n * 2 }
```

✅ Chain `select`, `map`, etc., for clean transformations.

---

## ⚠️ BONUS: Exception-Safe Looping

Wrap loops in `begin/rescue` to catch and handle errors inside iterations.

```ruby
[10, 0, 5].each do |num|
  begin
    puts 100 / num
  rescue ZeroDivisionError
    puts "Cannot divide by zero"
  end
end
```

---

Would you like an advanced example using `Enumerator`, `lazy`, or looping over files or APIs?
