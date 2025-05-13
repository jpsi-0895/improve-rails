To show **exceptional skills with Rails models**, you need to go far beyond basic `ActiveRecord` usage. Great Rails developers write models that are clean, flexible, performant, and maintainable — without stuffing business logic or creating tight coupling.

Here’s what **exceptional Rails model usage** looks like:

---

## 💎 1. **Lean Models – Fat Services**

* Exceptional devs **keep models focused** on persistence logic.
* Use **Service Objects**, **Form Objects**, and **Value Objects** to offload logic.

```ruby
# app/services/order_processor.rb
class OrderProcessor
  def initialize(order)
    @order = order
  end

  def call
    @order.calculate_total!
    @order.save!
  end
end
```

✅ This keeps models lean and logic reusable.

---

## 💡 2. **Advanced Validations**

### Conditional Validations

```ruby
validates :discount_code, presence: true, if: :used_coupon?
```

### Custom Validator Class

```ruby
# app/validators/email_blacklist_validator.rb
class EmailBlacklistValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    record.errors.add(attribute, 'is blacklisted') if Blacklist.include?(value)
  end
end

# in model
validates :email, email_blacklist: true
```

---

## 🧠 3. **Use Scopes Intelligently**

Don't cram complex logic in controller queries — define expressive scopes:

```ruby
scope :active, -> { where(status: 'active') }
scope :recent, -> { order(created_at: :desc).limit(10) }
```

Combine scopes:

```ruby
User.active.recent
```

---

## 🔄 4. **Use Callbacks Thoughtfully (or Avoid Them)**

Overusing `before_save` and `after_create` can hurt maintainability. Great devs:

* Use them only for **side-effect-free logic**
* Prefer **service objects** or **event systems** for complex flows

If used:

```ruby
before_save :normalize_username

def normalize_username
  self.username = username.strip.downcase
end
```

---

## 🧩 5. **Composed Models / Value Objects**

Extract complex value logic from models:

```ruby
# app/models/money.rb
class Money
  attr_reader :amount, :currency

  def initialize(amount, currency = "USD")
    @amount = amount
    @currency = currency
  end

  def to_s
    "#{currency} #{'%.2f' % amount}"
  end
end
```

Use in model:

```ruby
def price_object
  Money.new(price_cents / 100.0)
end
```

---

## 🔍 6. **Custom Query Objects**

Avoid cluttering models with raw SQL or multi-join queries:

```ruby
# app/queries/inactive_users_query.rb
class InactiveUsersQuery
  def self.call
    User.where(active: false).where("last_login_at < ?", 6.months.ago)
  end
end
```

---

## 🧮 7. **Efficient Use of Database**

### Use `pluck`, `select`, `exists?`, and `includes`:

```ruby
User.where(admin: true).pluck(:email)
User.includes(:posts).find(1) # avoids N+1 queries
User.exists?(email: "test@example.com")
```

---

## 🎯 8. **Composed Associations & Polymorphism**

### Polymorphic Association

```ruby
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end

# In other models:
has_many :comments, as: :commentable
```

---

## 🧪 9. **Strong Model Testing (RSpec / Minitest)**

Exceptional devs write **isolated, clear model specs** that test:

* Validations
* Scopes
* Methods
* Edge cases (nulls, invalid formats)

```ruby
describe User do
  it "is invalid without an email" do
    expect(User.new(email: nil)).not_to be_valid
  end
end
```

---

## 🔗 10. **Use `ActiveModel` for Non-DB Models**

Build form objects or adapters with `ActiveModel`:

```ruby
class ContactForm
  include ActiveModel::Model

  attr_accessor :name, :email, :message
  validates :email, presence: true
end
```

---

## 🧘 Summary: Exceptional Rails Model Skills

| Skill                | Why It Matters                        |
| -------------------- | ------------------------------------- |
| Lean models          | Improves separation of concerns       |
| Custom validators    | Cleaner and reusable validation logic |
| Scopes               | Make queries declarative and readable |
| Value objects        | Encapsulate domain-specific logic     |
| Query objects        | Avoid bloated ActiveRecord classes    |
| Avoid callback abuse | Improves testability and clarity      |
| ActiveModel use      | Enables non-DB-backed logic cleanly   |
| Smart querying       | Better performance and DB usage       |

---

Would you like examples of any of these patterns in your own model context (e.g., user, invoice, payment)?
