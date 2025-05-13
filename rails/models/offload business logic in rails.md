Offloading business logic in a **Rails** application means **removing non-persistence logic** from models, controllers, and views — and instead placing it into **dedicated, testable, reusable objects**. This results in cleaner code, better organization, and easier maintenance.

---

## 🔧 Why Offload Business Logic?

Without offloading, Rails apps tend to accumulate:

* 💥 **Fat Models** (1000+ line ActiveRecord classes)
* 😵 **Bloated Controllers** (handling 3+ responsibilities)
* 🧪 **Hard-to-test logic** mixed with DB or UI code

---

## 🎯 Goals of Offloading

| Goal                       | Explanation                                                   |
| -------------------------- | ------------------------------------------------------------- |
| **Single Responsibility**  | Each class/object should do one thing                         |
| **Separation of Concerns** | Isolate business rules from infrastructure                    |
| **Reusability**            | Logic works across CLI, jobs, APIs, etc.                      |
| **Testability**            | Pure objects are easier to test than AR models or controllers |

---

## 🧰 Tools for Offloading Business Logic in Rails

| Pattern/Object                    | Use For                                                         |
| --------------------------------- | --------------------------------------------------------------- |
| ✅ **Service Object**              | Coordinating complex workflows                                  |
| ✅ **Form Object**                 | Handling form validation across multiple models                 |
| ✅ **Value Object**                | Wrapping rich domain concepts (e.g. Money, Period, Coordinates) |
| ✅ **Query Object**                | Encapsulating complex or reusable DB queries                    |
| ✅ **Presenter / Decorator**       | Formatting data for views                                       |
| ✅ **Policy Object**               | Authorization logic (`Pundit`, `AccessGranted`)                 |
| ✅ **Interactor / Command Object** | Encapsulating commands with undo/state transitions              |

---

## 🛠 Example Breakdown: Offloading from a Model

### ❌ Before (Fat Model):

```ruby
class Order < ApplicationRecord
  def process!
    charge_card
    notify_user
    send_invoice
  end

  def charge_card
    Stripe::Charge.create(...)
  end

  def notify_user
    OrderMailer.confirmation(self).deliver_later
  end

  def send_invoice
    PdfService.generate_invoice(self)
  end
end
```

### ✅ After (Lean Model + Service Object):

```ruby
# app/models/order.rb
class Order < ApplicationRecord
  # Just associations, validations, and scopes
end

# app/services/order_processor.rb
class OrderProcessor
  def initialize(order)
    @order = order
  end

  def call
    charge_card
    notify_user
    send_invoice
  end

  private

  def charge_card
    Stripe::Charge.create(...)
  end

  def notify_user
    OrderMailer.confirmation(@order).deliver_later
  end

  def send_invoice
    PdfService.generate_invoice(@order)
  end
end
```

**Usage:**

```ruby
OrderProcessor.new(order).call
```

---

## 📦 Real-World Use Cases

### 1. **Signup Form (Form Object)**

```ruby
class SignupForm
  include ActiveModel::Model

  attr_accessor :email, :password

  def save
    return false unless valid?
    User.create(email: email, password: password)
  end
end
```

---

### 2. **Invoice Amount Calculation (Value Object)**

```ruby
class Money
  attr_reader :amount, :currency

  def initialize(amount, currency = "USD")
    @amount = amount
    @currency = currency
  end

  def to_s
    "#{currency} %.2f" % amount
  end
end
```

---

### 3. **Reusable Queries (Query Object)**

```ruby
class OverdueInvoices
  def self.call
    Invoice.where("due_date < ?", Date.today).where(paid: false)
  end
end
```

---

### 4. **Policies (Authorization Logic)**

Using Pundit or writing your own:

```ruby
class PostPolicy
  def initialize(user, post)
    @user = user
    @post = post
  end

  def edit?
    user.admin? || post.user_id == user.id
  end
end
```

---

## ✅ Summary: When to Offload Logic

| Type of Logic              | Offload To          |
| -------------------------- | ------------------- |
| Multi-step processes       | Service Object      |
| Non-persistent form inputs | Form Object         |
| Custom business values     | Value Object        |
| Complex DB queries         | Query Object        |
| Data formatting for views  | Presenter/Decorator |
| Permissions/roles          | Policy Object       |

---

## Need help?

I can help you:

* Refactor fat models/controllers
* Build a structure for service/form/query objects
* Create clean domain-focused architecture

Just let me know what part of your app needs cleanup!
