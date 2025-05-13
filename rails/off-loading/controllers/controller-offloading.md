Offloading logic from **Rails controllers** is critical for building clean, maintainable, and testable applications. Controllers should **only coordinate**, not perform business logic.

---

## 🧠 Why Offload Controller Logic?

Controllers often become bloated by:

* Business rules
* Side effects (emails, payments, PDFs)
* Data preparation
* Conditional logic
* JSON transformations

💡 **Exceptional controllers are skinny and orchestrate only.**

---

## 🛠️ Step-by-Step: How to Offload Controller Logic

### ✅ 1. **Use Service Objects** for business processes

Instead of doing everything in the controller:

### ❌ Before:

```ruby
def create
  order = Order.new(order_params)
  if order.save
    PaymentGateway.charge(order.total)
    NotificationMailer.order_confirmation(order).deliver_now
    redirect_to order_path(order)
  else
    render :new
  end
end
```

### ✅ After:

```ruby
def create
  result = OrderCreator.call(order_params)

  if result.success?
    redirect_to order_path(result.order)
  else
    @order = result.order
    render :new
  end
end
```

```ruby
# app/services/order_creator.rb
class OrderCreator
  def self.call(params)
    new(params).call
  end

  attr_reader :order

  def initialize(params)
    @params = params
    @order = Order.new(params)
  end

  def call
    return failure unless order.save

    PaymentGateway.charge(order.total)
    NotificationMailer.order_confirmation(order).deliver_later

    success
  end

  def success?
    @success
  end

  private

  def success
    @success = true
    self
  end

  def failure
    @success = false
    self
  end
end
```

---

### ✅ 2. **Use Form Objects** for complex validation

Forms that affect multiple models or have complex validations shouldn't live in the controller.

```ruby
# app/forms/user_signup_form.rb
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

In controller:

```ruby
def create
  @form = UserSignupForm.new(signup_params)
  if @form.save
    redirect_to dashboard_path
  else
    render :new
  end
end
```

---

### ✅ 3. **Use Decorators/Presenters** for view logic

```ruby
# app/presenters/order_presenter.rb
class OrderPresenter < SimpleDelegator
  def formatted_total
    "$%.2f" % total
  end

  def status_label
    case status
    when 'paid' then '✅ Paid'
    when 'pending' then '⏳ Pending'
    else '❌ Unknown'
    end
  end
end
```

In controller:

```ruby
@order = OrderPresenter.new(Order.find(params[:id]))
```

---

### ✅ 4. **Use Policies or Pundit** for authorization

```ruby
def update
  authorize @user
  @user.update(user_params)
  redirect_to @user
end
```

---

### ✅ 5. **Use Query Objects for data access**

```ruby
# app/queries/active_users_query.rb
class ActiveUsersQuery
  def self.call
    User.where(active: true).order(last_login_at: :desc)
  end
end
```

In controller:

```ruby
@users = ActiveUsersQuery.call
```

---

## 📚 Controller Golden Rule

> **If it's not about handling the request or response, it probably doesn’t belong in the controller.**

---

## 💡 Controller Checklist

| Task                          | Belongs in Controller? |
| ----------------------------- | ---------------------- |
| Parsing params                | ✅ Yes                  |
| Calling services              | ✅ Yes                  |
| Performing side effects       | ❌ Use service object   |
| Data transformations for view | ❌ Use presenter        |
| Multi-model form validation   | ❌ Use form object      |
| Authorization logic           | ❌ Use policy           |
| Sending emails                | ❌ Offload              |
| DB queries with conditions    | ❌ Use query object     |

---

## 🚀 Want to Refactor a Specific Controller?

Tell me the name of your controller or paste some code — I can help you refactor it using services, forms, and queries cleanly.
