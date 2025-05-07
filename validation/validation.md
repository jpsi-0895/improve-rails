In Ruby on Rails, **request validations** ensure incoming data from clients (e.g., via forms or API calls) meets certain criteria before being processed. Rails provides several ways to handle these validations, depending on the context:

---

### 1. **Model Validations**

Use when data is persisted to the database.

```ruby
class User < ApplicationRecord
  validates :email, presence: true, uniqueness: true
  validates :age, numericality: { greater_than: 0 }
end
```

In your controller:

```ruby
def create
  @user = User.new(user_params)
  if @user.save
    render json: @user, status: :created
  else
    render json: { errors: @user.errors.full_messages }, status: :unprocessable_entity
  end
end
```

---

### 2. **Strong Parameters (Controller Level)**

Ensure only permitted attributes are passed.

```ruby
def user_params
  params.require(:user).permit(:name, :email, :age)
end
```

---

### 3. **Custom Request Validators (for APIs or non-model input)**

Use `ActiveModel::Validations` to validate raw input or params outside of ActiveRecord.

```ruby
class CreateUserValidator
  include ActiveModel::Model

  attr_accessor :email, :age
  validates :email, presence: true
  validates :age, numericality: { greater_than: 0 }

  def initialize(params)
    @email = params[:email]
    @age = params[:age]
  end
end
```

Then in your controller:

```ruby
def create
  validator = CreateUserValidator.new(params[:user])
  if validator.valid?
    # Proceed with business logic
  else
    render json: { errors: validator.errors.full_messages }, status: :unprocessable_entity
  end
end
```

---

### 4. **API Parameter Validation with `dry-validation` or `reform`**

For more complex APIs, libraries like [`dry-validation`](https://dry-rb.org/gems/dry-validation/) allow schema-based validations.

---

Would you like an example using Rails 7 API-only mode or integrated with a front-end framework?
