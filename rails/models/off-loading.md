Using **Service Objects**, **Form Objects**, and **Value Objects** in Rails is a hallmark of well-structured, maintainable apps — and it's what exceptional developers do to **offload business logic** from bloated models or controllers.

Here's what each of these patterns is for, when to use it, and how to implement it.

---

## ✅ 1. **Service Objects** – for performing actions

**Use when**: You need to carry out a business process (e.g. charge a card, send emails, process an order).

### 🔧 Example: `app/services/order_processor.rb`

```ruby
class OrderProcessor
  def initialize(order)
    @order = order
  end

  def call
    charge_customer
    update_inventory
    send_confirmation_email
  end

  private

  def charge_customer
    Stripe::Charge.create(amount: @order.total_cents, currency: "usd", source: @order.user.stripe_token)
  end

  def update_inventory
    @order.line_items.each { |item| item.product.decrement!(:stock, item.quantity) }
  end

  def send_confirmation_email
    OrderMailer.confirmation(@order).deliver_later
  end
end
```

**Usage:**

```ruby
OrderProcessor.new(@order).call
```

➡️ Keeps all side effects and orchestration out of your model.

---

## ✅ 2. **Form Objects** – for complex forms or multiple models

**Use when**: A single form touches multiple models or has validations unrelated to DB schema.

### 🔧 Example: `app/forms/signup_form.rb`

```ruby
class SignupForm
  include ActiveModel::Model

  attr_accessor :user_email, :user_password, :profile_bio

  validates :user_email, presence: true
  validates :user_password, length: { minimum: 6 }

  def save
    return false unless valid?

    ActiveRecord::Base.transaction do
      user = User.create!(email: user_email, password: user_password)
      Profile.create!(user: user, bio: profile_bio)
    end

    true
  rescue => e
    errors.add(:base, e.message)
    false
  end
end
```

**Usage in Controller:**

```ruby
def create
  @form = SignupForm.new(signup_params)
  if @form.save
    redirect_to dashboard_path
  else
    render :new
  end
end
```

---

## ✅ 3. **Value Objects** – for encapsulating domain logic around data

**Use when**: You need to represent a concept with behavior but don't want to store it in the DB.

### 🔧 Example: `app/models/money.rb`

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

  def +(other)
    raise "Currency mismatch" unless other.currency == currency
    Money.new(amount + other.amount, currency)
  end
end
```

**Usage:**

```ruby
price = Money.new(10)
tax = Money.new(1.5)
total = price + tax
puts total.to_s # => "USD 11.50"
```

➡️ Keeps rich, reusable logic out of models and views.

---

## 🧠 Summary

| Object Type        | Use For                                     | Included in Rails?             |
| ------------------ | ------------------------------------------- | ------------------------------ |
| **Service Object** | Coordinating actions/business workflows     | No (Plain Ruby)                |
| **Form Object**    | Complex form input & cross-model validation | Yes (via `ActiveModel::Model`) |
| **Value Object**   | Encapsulating logic around simple data      | No (Plain Ruby)                |

---

## 🔥 Real-World Benefits

* **Code reusability**: Logic is portable across jobs, CLI, and web
* **Testability**: Objects are easy to isolate and unit test
* **Maintenance**: You won’t fear touching your models
* **Clarity**: Each object has a single, clear responsibility

---

Would you like a real example of refactoring a fat controller or model using these patterns?
