Great! Let’s break down **Value Objects** in **Rails** step-by-step — with rules, purpose, and practical usage.

---

## 🧠 What Is a Value Object?

A **Value Object** is a small object that:

* Represents a concept in your domain (e.g. `Money`, `Address`, `DateRange`)
* Is **immutable**
* Is compared by **value**, not identity
* Encapsulates logic around that value

### 🔄 Example

```ruby
money1 = Money.new(100, "USD")
money2 = Money.new(100, "USD")
money1 == money2  # true
```

---

## ✅ Why Use Value Objects?

| Without VO                             | With VO                     |
| -------------------------------------- | --------------------------- |
| Scattered business logic               | Centralized, testable logic |
| String or float everywhere             | Strongly typed concepts     |
| Duplicated formatting/calculation code | Reusable objects            |
| Logic in views or helpers              | Logic stays near the data   |

---

## 📏 Rules of Value Objects

1. ✅ Must be **immutable**
2. ✅ Must be **compared by value**
3. ✅ Must **encapsulate behavior**, not just data
4. 🚫 Should not rely on ActiveRecord
5. ✅ Can include validation, math, formatting, etc.

---

## 🪜 Step-by-Step: Creating a Value Object in Rails

### 🎯 Use Case: `Money` Object (`amount + currency`)

---

### **Step 1: Define the Class**

```ruby
# app/domains/billing/value_objects/money.rb
class Money
  attr_reader :amount, :currency

  def initialize(amount, currency = "USD")
    @amount = amount
    @currency = currency
    freeze  # Ensures immutability
  end
end
```

---

### **Step 2: Add Behavior**

```ruby
def to_s
  "#{currency} %.2f" % amount
end

def +(other)
  raise "Currency mismatch" unless other.currency == currency
  Money.new(amount + other.amount, currency)
end
```

---

### **Step 3: Add Value-Based Equality**

```ruby
def ==(other)
  other.is_a?(Money) && amount == other.amount && currency == other.currency
end
```

---

### **Step 4: Use It in Your Model or Service**

```ruby
# app/models/order.rb
class Order < ApplicationRecord
  def total_money
    Money.new(total_cents / 100.0, currency)
  end
end

# OR in a service
price = Money.new(29.99)
tax = Money.new(5.00)
total = price + tax
puts total.to_s  # => "USD 34.99"
```

---

### **Step 5: Add Specs (Highly Recommended)**

```ruby
# spec/domains/billing/value_objects/money_spec.rb
RSpec.describe Money do
  it "adds correctly" do
    result = Money.new(10) + Money.new(5)
    expect(result.to_s).to eq("USD 15.00")
  end

  it "rejects currency mismatch" do
    expect {
      Money.new(10, "USD") + Money.new(5, "EUR")
    }.to raise_error("Currency mismatch")
  end
end
```

---

## 🧰 Other Common Value Object Examples

| Concept       | Example Attributes         |
| ------------- | -------------------------- |
| `Address`     | street, city, zip, country |
| `DateRange`   | start\_date, end\_date     |
| `Email`       | value + validation         |
| `PhoneNumber` | country code, number       |
| `Duration`    | value in minutes/hours     |
| `Percentage`  | value between 0–100        |

---

## 🧘 Summary

| Step | Description                                             |
| ---- | ------------------------------------------------------- |
| 1️⃣  | Define a plain Ruby class with attributes               |
| 2️⃣  | Make it **immutable** (use `freeze`)                    |
| 3️⃣  | Add useful **domain behavior** (math, formatting, etc.) |
| 4️⃣  | Implement **value equality** (`==`)                     |
| 5️⃣  | Use in models, services, forms, etc.                    |
| 6️⃣  | Write specs to test behavior                            |

---

## 🚀 Want to Add a Value Object to Your App?

Tell me your domain (e.g. shipping, pricing, invoices), and I’ll help design a clean value object for it.
