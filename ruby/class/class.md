In Ruby, class definitions and methods follow a straightforward syntax, but there are some important rules and conventions to be aware of when writing clean, maintainable code.

### ✅ **Class Definition**

To define a class in Ruby, you use the `class` keyword, followed by the name of the class (in **PascalCase**), and then the `end` keyword to close the class.

#### Example:

```ruby
class Car
  # Class body goes here
end
```

#### Naming Conventions:

* **Classes**: Should be written in **PascalCase** (first letter capitalized, no underscores).

  * Example: `Car`, `Person`, `UserAccount`.
* **Methods and variables**: Should use **snake\_case** (all lowercase with underscores between words).

  * Example: `drive_car`, `person_name`, `calculate_total`.

---

### ✅ **Instance Methods**

Instance methods belong to an instance of a class. To define an instance method, you use the `def` keyword, followed by the method name, and then `end` to close the method.

#### Example:

```ruby
class Car
  def start_engine
    puts "Engine started!"
  end
end
```

To call this method on an instance of `Car`:

```ruby
my_car = Car.new
my_car.start_engine  # Output: "Engine started!"
```

### ✅ **Class Methods**

Class methods are methods that belong to the **class** itself, not to an instance. You define class methods using the `self` keyword.

#### Example:

```ruby
class Car
  def self.total_cars
    puts "Total cars: 10"
  end
end

Car.total_cars  # Output: "Total cars: 10"
```

---

### ✅ **Access Control** (Public, Private, Protected)

Ruby has three types of access control for methods:

* **Public**: Methods are accessible from anywhere.
* **Private**: Methods are only accessible within the class or its instances (not from outside).
* **Protected**: Methods are accessible within the class and by instances of the same class or its subclasses.

#### Example:

```ruby
class Car
  def start_engine  # Public by default
    puts "Engine started!"
  end

  private

  def check_oil
    puts "Checking oil..."
  end
end

my_car = Car.new
my_car.start_engine  # Works fine
my_car.check_oil     # Error: private method `check_oil' called for #<Car:...>
```

---

### ✅ **Constructor Method (`initialize`)**

Ruby provides an `initialize` method to create and set up new objects. This method is automatically called when a new instance of the class is created via `new`.

#### Example:

```ruby
class Car
  def initialize(make, model)
    @make = make
    @model = model
  end

  def details
    "#{@make} #{@model}"
  end
end

my_car = Car.new("Toyota", "Camry")
puts my_car.details  # Output: "Toyota Camry"
```

---

### ✅ **Instance Variables**

Instance variables in Ruby are prefixed with an `@` symbol. These variables are tied to specific instances of a class and can be accessed and modified within instance methods.

#### Example:

```ruby
class Car
  def initialize(make, model)
    @make = make
    @model = model
  end

  def details
    "#{@make} #{@model}"
  end
end
```

---

### ✅ **Getter and Setter Methods**

Ruby has built-in support for getter and setter methods.

* **Getter**: Retrieve the value of an instance variable.
* **Setter**: Set the value of an instance variable.

Ruby provides the `attr_reader`, `attr_writer`, and `attr_accessor` methods to automatically create these methods.

* `attr_reader`: Creates a getter method.
* `attr_writer`: Creates a setter method.
* `attr_accessor`: Creates both getter and setter methods.

#### Example:

```ruby
class Car
  attr_accessor :make, :model  # Creates both getter and setter methods

  def initialize(make, model)
    @make = make
    @model = model
  end
end

my_car = Car.new("Toyota", "Camry")
puts my_car.make   # "Toyota"
my_car.make = "Honda"
puts my_car.make   # "Honda"
```

If you only need a getter or a setter, you can use `attr_reader` or `attr_writer`:

```ruby
class Car
  attr_reader :make  # Only creates a getter
  attr_writer :model # Only creates a setter
end
```

---

### ✅ **Inheritance**

In Ruby, a class can inherit from another class using the `<` symbol. The child class will inherit all methods and properties of the parent class.

#### Example:

```ruby
class Vehicle
  def move
    puts "Moving..."
  end
end

class Car < Vehicle
  def drive
    puts "Driving a car..."
  end
end

my_car = Car.new
my_car.move   # Inherited method
my_car.drive  # Method in Car class
```

---

### ✅ **Method Overriding**

A subclass can override methods inherited from the parent class by defining its own version of the method.

#### Example:

```ruby
class Vehicle
  def move
    puts "Vehicle moving..."
  end
end

class Car < Vehicle
  def move
    puts "Car moving..."
  end
end

my_car = Car.new
my_car.move  # "Car moving..." (Overrides Vehicle's move)
```

---

### ✅ **Modules (Mixins)**

Ruby uses **modules** for reusable code that can be mixed into classes. You can include a module in a class using `include` or `extend`.

* `include`: Includes instance methods from the module.
* `extend`: Includes class methods from the module.

#### Example:

```ruby
module Drivable
  def drive
    puts "Driving..."
  end
end

class Car
  include Drivable  # Instance method
end

class Boat
  extend Drivable  # Class method
end

car = Car.new
car.drive  # "Driving..."

Boat.drive  # "Driving..."
```

---

### ✅ **Summary of Method Rules**

| **Rule**                       | **Description**                                                         |
| ------------------------------ | ----------------------------------------------------------------------- |
| **Method Name Convention**     | Use snake\_case (e.g., `start_engine`)                                  |
| **Instance Methods**           | Defined with `def`, operate on instance vars (`@var`)                   |
| **Class Methods**              | Defined with `def self.method_name`                                     |
| **Access Control**             | `public`, `private`, and `protected` methods                            |
| **Constructor (`initialize`)** | Initializes an object upon creation (`new`)                             |
| **Instance Variables**         | Prefixed with `@` (e.g., `@make`, `@model`)                             |
| **Getter/Setter**              | Use `attr_reader`, `attr_writer`, `attr_accessor` for automatic methods |
| **Inheritance**                | Subclasses inherit methods from the parent class (`<`)                  |
| **Method Overriding**          | Overriding parent methods in subclasses                                 |
| **Modules**                    | Mix in reusable code via `include` or `extend`                          |

---

Let me know if you'd like more examples or deeper explanations on any of these concepts!
