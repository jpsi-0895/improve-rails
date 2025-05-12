Here’s a list of **common Rails functions** and **ActiveRecord methods** that you can use in your models, controllers, and views. This will help give you a clear overview of the built-in functionality that Rails provides and when to use them.

### **ActiveRecord Methods (Model Functions)**

#### **Basic CRUD Operations:**

1. **`create`**: Creates and saves a new record in the database.

   ```ruby
   user = User.create(name: 'John Doe', email: 'john@example.com')
   ```

2. **`new`**: Instantiates a new object but doesn't save it to the database.

   ```ruby
   user = User.new(name: 'John Doe', email: 'john@example.com')
   ```

3. **`save`**: Saves the object to the database (returns `true` if saved, `false` if validation fails).

   ```ruby
   user.save
   ```

4. **`update`**: Updates the attributes of a record and saves it to the database.

   ```ruby
   user.update(email: 'newemail@example.com')
   ```

5. **`update_attributes`** (deprecated in Rails 6.1, use `update` instead).

6. **`destroy`**: Deletes a record from the database.

   ```ruby
   user.destroy
   ```

7. **`find`**: Finds a record by its primary key (raises `ActiveRecord::RecordNotFound` if not found).

   ```ruby
   user = User.find(1)
   ```

8. **`find_by`**: Finds a record by the first matching attribute (returns `nil` if not found).

   ```ruby
   user = User.find_by(email: 'john@example.com')
   ```

9. **`find_or_create_by`**: Finds a record by given attributes, or creates it if it doesn't exist.

   ```ruby
   user = User.find_or_create_by(email: 'john@example.com')
   ```

10. **`find_or_initialize_by`**: Similar to `find_or_create_by`, but returns an unsaved object if not found.

    ```ruby
    user = User.find_or_initialize_by(email: 'john@example.com')
    ```

11. **`first`** / **`last`**: Returns the first or last record in the table.

    ```ruby
    first_user = User.first
    last_user = User.last
    ```

12. **`all`**: Returns all records in the table.

    ```ruby
    users = User.all
    ```

13. **`count`**: Counts the number of records matching the conditions.

    ```ruby
    user_count = User.count
    ```

14. **`destroy_all`**: Deletes all records that match a condition.

    ```ruby
    User.where(active: false).destroy_all
    ```

15. **`delete_all`**: Deletes all records that match a condition **without callbacks**.

    ```ruby
    User.where(active: false).delete_all
    ```

16. **`exists?`**: Returns `true` if a record exists, based on a condition.

    ```ruby
    User.exists?(id: 1)
    ```

#### **Querying (Where, Limit, etc.):**

1. **`where`**: Filters records by specified conditions.

   ```ruby
   active_users = User.where(active: true)
   ```

2. **`or`**: Performs a logical OR operation between conditions.

   ```ruby
   active_or_suspended = User.where(active: true).or(User.where(suspended: true))
   ```

3. **`order`**: Orders records by specified attribute(s).

   ```ruby
   ordered_users = User.order(:name)
   ```

4. **`limit`**: Limits the number of records returned.

   ```ruby
   users = User.limit(10)
   ```

5. **`pluck`**: Retrieves a single attribute's value from all records (more efficient than `map`).

   ```ruby
   user_names = User.pluck(:name)
   ```

6. **`select`**: Selects specific columns to load.

   ```ruby
   users = User.select(:name, :email)
   ```

7. **`group`**: Groups records by a specific attribute (commonly used with aggregate functions like `count`, `sum`).

   ```ruby
   users_count_by_country = User.group(:country).count
   ```

8. **`having`**: Used with `group` to apply conditions to groups.

   ```ruby
   User.group(:country).having("count(id) > 10").count
   ```

9. **`joins`**: Performs an SQL join on associated tables.

   ```ruby
   users_with_posts = User.joins(:posts)
   ```

10. **`includes`**: Eager loads associations to avoid N+1 queries.

    ```ruby
    users = User.includes(:posts).all
    ```

11. **`left_joins`**: Performs a left join, ensuring that the left table records appear even if there’s no match.

    ```ruby
    users = User.left_joins(:posts).where(posts: { id: nil })
    ```

12. **`distinct`**: Removes duplicate records.

    ```ruby
    distinct_countries = User.select(:country).distinct
    ```

13. **`find_each`**: Efficiently iterates over records in batches.

    ```ruby
    User.find_each(batch_size: 100) do |user|
      # process user
    end
    ```

14. **`find_in_batches`**: Similar to `find_each`, but yields a batch of records.

    ```ruby
    User.find_in_batches(batch_size: 100) do |batch|
      # process each batch of users
    end
    ```
#### **Scopes:**

1. **`scope`**: Defines a reusable query scope.

   ```ruby
   class Post < ApplicationRecord
     scope :published, -> { where(status: 'published') }
     scope :recent, -> { order(created_at: :desc) }
   end
   ```

   Usage:

   ```ruby
   Post.published.recent
   ```

#### **Callbacks (Lifecycle Hooks):**

1. **`before_validation`**: Runs before validation.
2. **`after_validation`**: Runs after validation.
3. **`before_create`**: Runs before creating a record.
4. **`after_create`**: Runs after creating a record.
5. **`before_save`**: Runs before saving a record.
6. **`after_save`**: Runs after saving a record.
7. **`before_update`**: Runs before updating a record.
8. **`after_update`**: Runs after updating a record.
9. **`before_destroy`**: Runs before destroying a record.
10. **`after_destroy`**: Runs after destroying a record.

```ruby
class User < ApplicationRecord
  before_save :normalize_name

  private

  def normalize_name
    self.name = name.titleize
  end
end
```

#### **Custom Instance Methods:**

1. **`def method_name`**: Custom method for instance logic.

   ```ruby
   class Product < ApplicationRecord
     def formatted_price
       "$#{price.round(2)}"
     end
   end
   ```

2. **`self.method_name`**: Class methods for class-level logic.

   ```ruby
   class User < ApplicationRecord
     def self.active_users
       where(active: true)
     end
   end
   ```

#### **Custom Validations:**

1. **`validate`**: Custom validation method.

   ```ruby
   class User < ApplicationRecord
     validate :password_complexity

     private

     def password_complexity
       return if password =~ /^(?=.*[A-Z])(?=.*\d)/
       errors.add(:password, "must be complex")
     end
   end
   ```

### **Other Useful Methods:**

1. **`touch`**: Updates the `updated_at` timestamp.

   ```ruby
   post.touch
   ```

2. **`merge`**: Combines scopes or queries.

   ```ruby
   users = User.where(active: true).merge(User.order(:name))
   ```

3. **`to_param`**: Returns a string representation for URL generation.

   ```ruby
   post.to_param  # Often returns `id`, but can be overridden
   ```

### **ActiveRecord Query Methods:**

* `pluck`, `group`, `count`, `select`, `order`, `distinct`, `joins`, `includes`, `where`, `having`, etc.

Designing a **well-structured database** with **associations** in Rails is essential for building maintainable, efficient, and scalable applications. Given your 5 years of experience in Rails, you're likely already familiar with the core concepts, so let's explore **advanced database design** principles with a focus on **associations**, **scalability**, and **performance optimization**.

### 1. **Understanding Rails Associations**

Rails provides several types of associations to help manage relationships between models. Each association has a corresponding database constraint and needs to be carefully designed to balance **data integrity**, **query performance**, and **ease of maintenance**.

#### **Types of Associations:**

1. **`belongs_to`**: This indicates that the current model contains a foreign key pointing to another model. It's typically a **one-to-one** relationship, but it can also be a **many-to-one** relationship when multiple records from the model refer to a single record in another model.

   * Example: `Post` belongs to `User` (a post has one user).

   ```ruby
   class Post < ApplicationRecord
     belongs_to :user
   end
   ```

2. **`has_many`**: This indicates that the model contains a collection of other models. It represents a **one-to-many** relationship.

   * Example: `User` has many `Posts`.

   ```ruby
   class User < ApplicationRecord
     has_many :posts
   end
   ```

3. **`has_one`**: This indicates that the model contains one instance of another model. It's typically a **one-to-one** relationship.

   * Example: `User` has one `Profile`.

   ```ruby
   class User < ApplicationRecord
     has_one :profile
   end
   ```

4. **`has_and_belongs_to_many` (HABTM)**: This represents a **many-to-many** relationship without a join model. Rails will automatically create a join table with the foreign keys.

   * Example: `User` has and belongs to many `Roles`.

   ```ruby
   class User < ApplicationRecord
     has_and_belongs_to_many :roles
   end
   ```

5. **`has_many :through`**: A more explicit form of **many-to-many** relationships that requires a join model. This gives you more flexibility, allowing you to add additional attributes to the join table.

   * Example: `User` has many `Posts` through `Comments`.

   ```ruby
   class User < ApplicationRecord
     has_many :comments
     has_many :posts, through: :comments
   end
   ```

### 2. **Database Design Best Practices**

#### **1. Normalization vs. Denormalization**

* **Normalization** aims to reduce redundancy and dependency by organizing the data into tables that adhere to the principles of **1NF**, **2NF**, and **3NF**.
* **Denormalization** might be required for **read-heavy applications** or when performance is critical, but at the cost of **data integrity** and **storage space**.

#### **2. Use Foreign Keys and Indexes**

* Always add foreign keys to **`belongs_to`** associations, and ensure that the foreign key column is **indexed** for faster lookups.
* Rails automatically creates **foreign key constraints** with migrations, but for **complex relationships**, you might want to define them explicitly.

  Example of adding a foreign key with an index:

  ```ruby
  add_reference :posts, :user, foreign_key: true, index: true
  ```

#### **3. Avoiding N+1 Queries**

* An N+1 query problem arises when you fetch a list of records and then loop over them to fetch associated records (one query per record).
* Use **eager loading** (`includes`, `preload`, `eager_load`) to optimize the number of database queries.
  Example:

  ```ruby
  posts = Post.includes(:comments).where(user_id: 1)
  ```

#### **4. Foreign Key Constraints for Data Integrity**

* For **referential integrity**, always use **foreign keys** to ensure the relationships are maintained when records are deleted or updated. Use **`dependent: :destroy`** or **`dependent: :nullify`** based on your business rules for cascading deletes.

  Example:

  ```ruby
  class Post < ApplicationRecord
    belongs_to :user, dependent: :destroy
  end
  ```

#### **5. Optimizing for Performance with Indexes**

* Indexes are critical for performance, especially in **large datasets**. For foreign keys, it's standard practice to create **indexes** to speed up lookups.
* Common indexes:

  * On foreign key columns (`user_id`, `post_id`, etc.).
  * On frequently queried columns (e.g., `email`, `created_at`, `status`).

  Example of adding an index to a column:

  ```ruby
  add_index :users, :email, unique: true
  ```

#### **6. Use of Polymorphic Associations**

* Polymorphic associations allow a model to belong to more than one other model on a single association. While powerful, they can lead to performance trade-offs, so use them carefully.

  Example:

  ```ruby
  class Comment < ApplicationRecord
    belongs_to :commentable, polymorphic: true
  end

  class Post < ApplicationRecord
    has_many :comments, as: :commentable
  end
  ```

### 3. **Handling Complex Relationships**

#### **1. Self-Referencing Associations**

For hierarchical or tree-like data structures, use **self-referencing associations** (e.g., a **category** has many **subcategories**).

* In Rails, this is often implemented using a **parent-child** relationship with `belongs_to` and `has_many`.

Example:

```ruby
class Category < ApplicationRecord
  belongs_to :parent, class_name: 'Category', optional: true
  has_many :subcategories, class_name: 'Category', foreign_key: 'parent_id'
end
```

#### **2. Many-to-Many with Join Tables**

When dealing with **many-to-many relationships**, use **`has_many :through`** for better flexibility and performance. This allows you to add extra attributes on the join table.

Example: **User** has many **Roles** through a join table **UserRoles**:

```ruby
class User < ApplicationRecord
  has_many :user_roles
  has_many :roles, through: :user_roles
end

class Role < ApplicationRecord
  has_many :user_roles
  has_many :users, through: :user_roles
end

class UserRole < ApplicationRecord
  belongs_to :user
  belongs_to :role
end
```

#### **3. Using `counter_cache` for Efficient Counting**

Instead of running a query like `Post.count`, use `counter_cache` to store the count of associated records in a column. This can drastically improve performance by reducing unnecessary queries.

```ruby
class Post < ApplicationRecord
  belongs_to :user, counter_cache: true
end
```

Then, add a column `posts_count` to the `users` table, which will be automatically updated when a post is created or deleted.

### 4. **Handling Soft Deletes**

* **Soft deletes** (where you don't actually delete the row but mark it as deleted) can be implemented using gems like `paranoia` or custom flags.

  Example using `paranoia`:

  ```ruby
  class Post < ApplicationRecord
    acts_as_paranoid
  end
  ```

  With soft deletes, ensure that you have appropriate scopes (e.g., `only_deleted`, `with_deleted`).

### 5. **Database Migrations: Best Practices**

* **Avoid changing column types** in production directly, as this could cause data loss. Instead, create a new column and migrate data manually.
* When adding an index, always check if it will be beneficial for performance, as unnecessary indexes can degrade write performance.
* Use `null: false` for columns that should always have a value and `unique: true` where necessary (e.g., for email addresses).

Example of a migration:

```ruby
class AddEmailToUsers < ActiveRecord::Migration[6.0]
  def change
    add_column :users, :email, :string, null: false, unique: true
    add_index :users, :email, unique: true
  end
end
```

### 6. **Optimizing for Scalability**

#### **1. Use Background Jobs for Heavy Tasks**

* When doing tasks that require processing large volumes of data (e.g., sending emails, processing payments), offload them to background jobs (using `Sidekiq`, `Resque`, or `DelayedJob`).

#### **2. Partitioning Data**

* For **very large tables**, consider **partitioning** data (splitting a large table into smaller ones based on some condition, such as `created_at`).

#### **3. Caching**

* Use caching techniques like **Fragment Caching** and **Russian Doll Caching** for frequently accessed data that doesn't change often (e.g., user profile details).


### 7. **Advanced Considerations**

* **Event Sourcing**: In complex applications, consider using **event sourcing** to capture and persist changes to application state. This can be combined with **CQRS** (Command Query Responsibility Segregation) for improved performance.

* **Graph Databases**: For applications with very complex relationships (social networks, recommendation engines, etc.), you may want to explore **graph databases** like **Neo4j** or \*\*Amazon Neptune

### Conclusion:

With 5 years of Rails experience, you likely already know the fundamentals of database design, but the key to **excellent database design** lies in creating scalable, efficient, and maintainable systems. Always consider:

* **Indexes**, **foreign keys**, and **query optimization** for performance.
* **Polymorphic associations**, **self-referencing models**, and **many-to-many** relationships for flexibility.
* **Denormalization** and **caching** for handling high-traffic applications.
* **Counter caches**, **background jobs**, and **soft deletes** for managing state and improving efficiency.
