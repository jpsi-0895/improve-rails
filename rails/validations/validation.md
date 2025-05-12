In Ruby on Rails, **validations** are used to ensure that only valid data is saved into the database. Validations can be applied at the model level, meaning that before any data is saved or updated in the database, Rails checks whether it meets the criteria you define.

### **Common Validation Methods in Rails**

Here are some of the most commonly used validations:

---

### 1. **`validates_presence_of`**

Ensures that a field is not left empty.

```ruby
class User < ApplicationRecord
  validates_presence_of :name
end
```

* Ensures that the `name` attribute is **not null** or empty.

---

### 2. **`validates_uniqueness_of`**

Ensures that a field's value is unique in the database.

```ruby
class User < ApplicationRecord
  validates_uniqueness_of :email
end
```

* Ensures that the `email` attribute is **unique** in the `users` table.

---

### 3. **`validates_length_of`**

Validates the length of an attribute (minimum, maximum, or exact length).

```ruby
class User < ApplicationRecord
  validates_length_of :password, minimum: 8, maximum: 20
end
```

* Ensures the `password` is between **8 and 20 characters** long.

You can also use `validates` with `length`:

```ruby
validates :password, length: { minimum: 8, maximum: 20 }
```

---

### 4. **`validates_format_of`**

Validates the format of an attribute, typically for strings like emails, phone numbers, etc.

```ruby
class User < ApplicationRecord
  validates_format_of :email, with: URI::MailTo::EMAIL_REGEXP
end
```

* Ensures the `email` follows a standard email format (e.g., `user@example.com`).

---

### 5. **`validates_numericality_of`**

Ensures that an attribute is a number.

```ruby
class Product < ApplicationRecord
  validates_numericality_of :price
end
```

* Ensures the `price` attribute is a valid **number**.

---

### 6. **`validates_inclusion_of`**

Ensures that an attribute's value is included in a specified list.

```ruby
class User < ApplicationRecord
  validates_inclusion_of :role, in: ['admin', 'user', 'guest']
end
```

* Ensures the `role` is one of `'admin'`, `'user'`, or `'guest'`.

---

### 7. **`validates_exclusion_of`**

Ensures that an attribute's value is not included in a specified list.

```ruby
class User < ApplicationRecord
  validates_exclusion_of :username, in: ['admin', 'root', 'system']
end
```

* Ensures the `username` is **not** one of `'admin'`, `'root'`, or `'system'`.

---

### 8. **`validates_confirmation_of`**

Ensures that a field has a matching confirmation (commonly used for password fields).

```ruby
class User < ApplicationRecord
  validates_confirmation_of :password
end
```

* Ensures that the `password` and `password_confirmation` fields match.

---

### 9. **`validates_acceptance_of`**

Ensures that a field has been accepted (commonly used for terms and conditions checkboxes).

```ruby
class User < ApplicationRecord
  validates_acceptance_of :terms_of_service
end
```

* Ensures the `terms_of_service` checkbox is **checked** (typically boolean `true`).

---

### 10. **`validates_associated`**

Validates the associated objects of a model (useful for associations).

```ruby
class User < ApplicationRecord
  has_many :posts
  validates_associated :posts
end
```

* Ensures that the associated `posts` records are valid.

---

### 11. **`validates_absence_of`**

Ensures that an attribute is **empty or nil**.

```ruby
class User < ApplicationRecord
  validates_absence_of :nickname
end
```

* Ensures that the `nickname` attribute is **empty**.

---

### **Custom Validations**

Sometimes, the built-in validations are not enough. In that case, you can write **custom validations** by creating methods inside your model.

#### Example: Custom Validation for Email Domain

```ruby
class User < ApplicationRecord
  validate :email_domain

  private

  def email_domain
    unless email =~ /@example\.com$/
      errors.add(:email, "must be from example.com domain")
    end
  end
end
```

* The custom validation `email_domain` ensures that the `email` field ends with `@example.com`. If not, it adds an error message to the `email` field.

---

### **Validation Contexts**

You can define validation contexts to apply validations only in specific situations (e.g., creating, updating).

#### Example: Conditional Validations with `on`:

```ruby
class User < ApplicationRecord
  validates :email, presence: true, on: :create
  validates :password, presence: true, on: :update
end
```

* The `email` validation will only apply when creating a `User`.
* The `password` validation will only apply when updating a `User`.

---

### **Skipping Validations**

You can skip certain validations when needed using `:skip`:

```ruby
user = User.new(name: "John", email: "john@example.com")
user.save(validate: false)  # Skips validations
```

---

### **Using `valid?` and `errors`**

To check if a record is valid without saving it to the database, you can use the `valid?` method:

```ruby
user = User.new(name: "John", email: "john@example.com")
user.valid?  # Returns true or false based on validations
user.errors.full_messages  # Returns an array of error messages
```

---

### **Validation Callbacks**

Validations can be triggered using callbacks like `before_validation` and `after_validation`.

```ruby
class User < ApplicationRecord
  before_validation :normalize_email

  private

  def normalize_email
    self.email = email.downcase.strip if email
  end
end
```

* `before_validation` runs before the validation process. In this case, the `normalize_email` method ensures the email is downcased and stripped of spaces before being validated.

---

### **Summary of Common Validations:**

| Validation Method           | Description                                      | Example                                                       |
| --------------------------- | ------------------------------------------------ | ------------------------------------------------------------- |
| `validates_presence_of`     | Ensures that a field is not blank or nil         | `validates_presence_of :name`                                 |
| `validates_uniqueness_of`   | Ensures the value is unique in the database      | `validates_uniqueness_of :email`                              |
| `validates_length_of`       | Validates the length of the field                | `validates_length_of :password, minimum: 8`                   |
| `validates_format_of`       | Validates the format of the field (e.g., regex)  | `validates_format_of :email, with: URI::MailTo::EMAIL_REGEXP` |
| `validates_numericality_of` | Ensures the value is a number                    | `validates_numericality_of :price`                            |
| `validates_inclusion_of`    | Ensures the value is included in a list          | `validates_inclusion_of :status, in: ['active', 'inactive']`  |
| `validates_exclusion_of`    | Ensures the value is not included in a list      | `validates_exclusion_of :username, in: ['admin']`             |
| `validates_confirmation_of` | Ensures the field has a matching confirmation    | `validates_confirmation_of :password`                         |
| `validates_acceptance_of`   | Ensures the field is accepted (e.g., a checkbox) | `validates_acceptance_of :terms_of_service`                   |
| `validates_associated`      | Validates associated objects' validity           | `validates_associated :posts`                                 |
| `validates_absence_of`      | Ensures the field is empty or nil                | `validates_absence_of :nickname`                              |
| Custom Validations          | Custom logic for validation                      | `validate :custom_email_format`                               |

---

Let me know if you'd like further examples or clarification on specific validations!