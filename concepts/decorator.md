Great question. Let's look at **decorators in Rails** — specifically in the context of **exceptions or error handling**, often referred to as *exception decoration* or *wrapping*. This is not built into Rails by default, but it's a common **clean-code pattern** used in production-grade systems.

---

## 🧰 What is a Decorator in Rails?

In general, a **decorator** is an object that wraps another object to add behavior without changing the original object. In Rails, decorators are often used to:

* Cleanly format model data for views
* Isolate presentation logic
* Avoid fat helpers or models

Popular tools: [`Draper`](https://github.com/drapergem/draper), `SimpleDelegator`

---

## 🚨 Exception Handling with Decorators

Now when it comes to **exceptions**, decorators can:

* Wrap exceptions in a domain-specific context
* Format or enrich error responses
* Avoid leaking low-level exceptions (like `ActiveRecord::RecordNotFound`) into API responses

---

## ✅ Example: Decorating Exceptions in Rails API

### 🎯 Goal:

Convert raw Rails exceptions into clean, structured, domain-aware error responses.

---

### Step 1: Create a Decorator for Exceptions

```ruby
# app/decorators/exception_decorator.rb
class ExceptionDecorator
  def initialize(exception)
    @exception = exception
  end

  def to_h
    {
      error: type,
      message: message,
      code: http_status
    }
  end

  private

  attr_reader :exception

  def type
    case exception
    when ActiveRecord::RecordNotFound then "NotFoundError"
    when Pundit::NotAuthorizedError   then "AuthorizationError"
    else "InternalServerError"
    end
  end

  def message
    exception.message
  end

  def http_status
    case exception
    when ActiveRecord::RecordNotFound then 404
    when Pundit::NotAuthorizedError   then 403
    else 500
    end
  end
end
```

---

### Step 2: Use It in `ApplicationController`

```ruby
class ApplicationController < ActionController::API
  rescue_from StandardError, with: :handle_exception

  private

  def handle_exception(exception)
    decorated = ExceptionDecorator.new(exception)

    render json: decorated.to_h, status: decorated.to_h[:code]
  end
end
```

---

### Output:

When a `RecordNotFound` occurs, the user sees:

```json
{
  "error": "NotFoundError",
  "message": "Couldn't find User with id=99",
  "code": 404
}
```

---

## 🧠 Benefits

* Clean separation between error logic and controller
* Reusable, testable decorator class
* Avoids exposing low-level details to the API client
* Easy to extend for custom exceptions or logging

---

## 🧪 Testing

You can unit-test the decorator separately:

```ruby
describe ExceptionDecorator do
  it 'returns 404 for RecordNotFound' do
    ex = ActiveRecord::RecordNotFound.new("Not found")
    decorated = ExceptionDecorator.new(ex)

    expect(decorated.to_h[:code]).to eq(404)
    expect(decorated.to_h[:error]).to eq("NotFoundError")
  end
end
```

---

Would you like to see how this integrates with `Draper` or `ActiveSupport::Rescuable` for even cleaner error pipelines?
