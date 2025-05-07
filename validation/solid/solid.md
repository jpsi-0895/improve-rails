Great! Let’s integrate **request validations in Rails** using the **SOLID principles** to keep your code clean, maintainable, and scalable.

---

### 🚧 Context: You have an endpoint to create a `User`, and you want to validate input using **SOLID principles**.

---

## ✅ SOLID Breakdown in Rails Request Validations

---

### **S — Single Responsibility Principle (SRP)**

Each class or module should have one responsibility.

🔸 **Separate your validation logic from controllers and models.**

```ruby
# app/validators/create_user_validator.rb
class CreateUserValidator
  include ActiveModel::Model

  attr_accessor :email, :age

  validates :email, presence: true, format: URI::MailTo::EMAIL_REGEXP
  validates :age, numericality: { only_integer: true, greater_than: 0 }

  def initialize(params)
    @email = params[:email]
    @age = params[:age]
  end
end
```

🔸 The validator is only responsible for validating request data, not for saving it.

---

### **O — Open/Closed Principle**

Code should be open for extension, closed for modification.

🔸 Add new validation rules by subclassing or extending.

```ruby
# Add extra validations by subclassing
class AdminUserValidator < CreateUserValidator
  validates :admin_code, presence: true
end
```

---

### **L — Liskov Substitution Principle**

Subclasses should be substitutable for their base classes.

🔸 `AdminUserValidator` can be used wherever `CreateUserValidator` is expected.

---

### **I — Interface Segregation Principle**

Clients should not be forced to depend on methods they do not use.

🔸 Keep your validator focused. If you need different validations for `UpdateUser`, use a new class:

```ruby
class UpdateUserValidator
  include ActiveModel::Model

  attr_accessor :email

  validates :email, format: URI::MailTo::EMAIL_REGEXP, allow_blank: true
end
```

---

### **D — Dependency Inversion Principle**

Depend on abstractions, not concretions.

🔸 Let controller depend on an interface (like a validator class), not on specific implementation:

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def create
    validator = CreateUserValidator.new(user_params)

    if validator.valid?
      user = User.create!(validator.attributes)
      render json: user, status: :created
    else
      render json: { errors: validator.errors.full_messages }, status: :unprocessable_entity
    end
  end

  private

  def user_params
    params.require(:user).permit(:email, :age)
  end
end
```

---

## ✅ Benefits

* **Testable:** You can unit-test `CreateUserValidator` independently.
* **Reusability:** Use the validator in other services or commands.
* **Clean controller:** No bloated `create` methods.

---

Would you like a full working Rails API example or to expand this into a service object pattern as well?
