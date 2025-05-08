Awesome — let’s level up with **advanced Ruby looping** using `Enumerator`, `lazy evaluation`, and looping over more complex data sources like files and APIs.

---

## 🔮 1. **Enumerator & `lazy` Chaining**

Great for handling **large data sets** or **infinite sequences** without memory issues.

```ruby
evens = (1..Float::INFINITY).lazy
            .select { |n| n.even? }
            .map { |n| n * 2 }
            .take(5)
            .force

puts evens.inspect
# => [4, 8, 12, 16, 20]
```

✅ Uses **lazy evaluation** to only compute what's needed. Perfect for streams or paginated APIs.

---

## 📁 2. **Looping Over a File Line-by-Line**

Memory-efficient reading of large files.

```ruby
File.foreach("large_file.txt").with_index do |line, idx|
  puts "Line #{idx + 1}: #{line.strip}"
end
```

✅ Doesn’t load the entire file into memory.

---

## 🌐 3. **Loop Over API Pages (Paginated API)**

```ruby
require 'net/http'
require 'json'

page = 1
loop do
  uri = URI("https://api.example.com/items?page=#{page}")
  response = Net::HTTP.get(uri)
  data = JSON.parse(response)

  break if data["items"].empty?

  data["items"].each do |item|
    puts item["name"]
  end

  page += 1
end
```

✅ This fetches data **page-by-page** until there's nothing left — perfect for APIs with pagination.

---

## 🔂 4. **Custom Enumerator**

You can define your own lazy generator:

```ruby
def fibonacci
  Enumerator.new do |yielder|
    a, b = 0, 1
    loop do
      yielder << a
      a, b = b, a + b
    end
  end
end

fib = fibonacci.lazy.take(10).force
puts fib.inspect
# => [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

✅ Infinite sequences, but only compute what you need.

---

## 🛠 BONUS: `each_with_object` for Clean Accumulation

```ruby
[1, 2, 3].each_with_object([]) { |n, arr| arr << n * 10 }
# => [10, 20, 30]
```

---

These patterns are especially powerful for **scalable**, **stream-based**, or **functional-style Ruby**.

Would you like to build a small Ruby script using lazy enumerators and real data (e.g., GitHub API or local file processing)?
