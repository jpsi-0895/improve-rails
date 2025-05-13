Creating a **clean, domain-focused architecture in Rails** means organizing your application around **domain logic**, not just around MVC. This moves you beyond "fat models" and "thin controllers" and toward a **modular, maintainable**, and **scalable** Rails application — especially important for growing or complex apps.

---

## 💡 What Is Domain-Focused Architecture?

Also called **Domain-Driven Rails** or **service-oriented Rails**, it's about structuring your app around **business capabilities**, not just ActiveRecord models and views.

### 🧱 It uses:

* **Bounded contexts / modules** per domain (e.g. Billing, Orders, Users)
* **Service objects**, **form objects**, and **value objects**
* **Policies, validators, interactors**
* **Minimal logic in controllers and models**

---

## 🗂️ 1. **Organize by Domain, Not by Type**

Instead of:

```
app/
├── controllers/
├── models/
├── views/
```

Prefer:

```
app/
├── domains/
│   ├── billing/
│   │   ├── services/
│   │   ├── models/
│   │   ├── forms/
│   │   └── policies/
│   └── users/
```

This groups related behavior together and improves code ownership and isolation.

---

## 🛠️ 2. **Use Service Objects**

Handle workflows and business processes in dedicated objects.

```ruby
# app/domains/orders/services/order_processor.rb
class OrderProcessor
  def initialize(order)
    @order = order
  end

  def call
    validate_order
    charge_payment
    notify_customer
  end
end
```

**Usage:** `OrderProcessor.new(order).call`

---

## 🧾 3. **Use Form Objects**

To validate and save complex forms (multiple models, nested data, etc.)

```ruby
# app/domains/users/forms/user_signup_form.rb
class UserSignupForm
  include ActiveModel::Model

  attr_accessor :email, :password, :terms

  validates :email, :password, presence: true
  validates :terms, acceptance: true

  def save
    return false unless valid?
    User.create(email:, password:)
  end
end
```

---

## 💵 4. **Use Value Objects**

Model domain data (e.g. Money, Duration, DateRange) as plain objects.

```ruby
# app/domains/billing/value_objects/money.rb
class Money
  attr_reader :amount, :currency

  def initialize(amount, currency = "USD")
    @amount = amount
    @currency = currency
  end

  def +(other)
    raise "Currency mismatch" unless other.currency == currency
    Money.new(amount + other.amount, currency)
  end
end
```

---

## 🔍 5. **Extract Query Objects**

Encapsulate business queries.

```ruby
# app/domains/orders/queries/overdue_orders.rb
class OverdueOrders
  def self.call
    Order.where("due_date < ?", Time.zone.today).where(paid: false)
  end
end
```

---

## 🧑‍⚖️ 6. **Use Policy Objects or Pundit for Authorization**

```ruby
# app/domains/users/policies/user_policy.rb
class UserPolicy
  attr_reader :current_user, :user

  def initialize(current_user, user)
    @current_user = current_user
    @user = user
  end

  def update?
    current_user.admin? || current_user == user
  end
end
```

---

## 🔄 7. **Use Interactors / Commands for Stateful Flows**

```ruby
# app/domains/payments/interactors/create_payment.rb
class CreatePayment
  include Interactor

  def call
    payment = Payment.new(context.params)
    context.fail!(error: "Invalid") unless payment.save
    context.payment = payment
  end
end
```

---

## ✅ 8. **Keep Models Lean**

Models should:

* Handle persistence
* Define associations
* Include simple validations and scopes
* Delegate heavy logic elsewhere

```ruby
class Order < ApplicationRecord
  belongs_to :user
  has_many :items

  scope :recent, -> { where("created_at >= ?", 1.week.ago) }

  def total
    items.sum(&:price)
  end
end
```

---

## 🧪 9. **Test by Domain**

Mirror your domain structure in tests:

```
spec/
├── domains/
│   ├── billing/
│   │   ├── services/
│   │   └── value_objects/
│   └── orders/
```

This reinforces structure and encourages fast, focused unit tests.

---

## 🧘 Summary: Clean Architecture in Rails

| Principle                      | Description                                 |
| ------------------------------ | ------------------------------------------- |
| **Domain modules**             | Organize by business concern, not file type |
| **Service/form/query objects** | Offload logic out of models and controllers |
| **Value objects**              | Represent rich domain data                  |
| **Interactor/policies**        | Handle workflows and access rules cleanly   |
| **Minimal Rails magic**        | Use plain Ruby where possible               |

---

## 🚀 Want Help Structuring a Specific Domain?

I can help you break down a real-world example — like refactoring a `Checkout` flow, `User signup`, or `Invoice generation` — into clean architecture. Just tell me the use case you're working on!
