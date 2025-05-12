Using **HTTP caching headers** like `ETag` and `Last-Modified` in Rails can dramatically improve performance by **avoiding unnecessary rendering or data transfer**, especially for APIs or data-heavy views.

## **1. What Are ETag and Last-Modified?**

* **`ETag` (Entity Tag)**: A unique identifier (usually a hash) for the content of a response. If the client sends back the same `ETag`, the server can return `304 Not Modified`.
* **`Last-Modified`**: A timestamp of when the resource was last updated. The client sends this back in `If-Modified-Since`.

If nothing has changed, the server can **skip the response body**, saving time and bandwidth.


## **2. How to Use `ETag` in Rails**

Rails can automatically generate and compare `ETag`s based on the content or a key.

### **Example: In a Controller**

```ruby
def show
  @post = Post.find(params[:id])
  fresh_when(@post)
end
```

* `fresh_when(@post)` automatically sets the ETag and Last-Modified headers based on `@post.updated_at`.
* If the client sends matching headers (`If-None-Match`, `If-Modified-Since`), Rails responds with `304 Not Modified`.

## **3. How to Use `Last-Modified` Only**

```ruby
def show
  @post = Post.find(params[:id])
  if stale?(last_modified: @post.updated_at.utc, public: true)
    render json: @post
  end
end
```

* `stale?` checks if the content is fresh. If it is, it will render the response; otherwise, it returns `304`.

---

## **4. Use in APIs**

```ruby
def index
  @posts = Post.order(updated_at: :desc)

  last_post = @posts.first
  if stale?(etag: last_post, last_modified: last_post.updated_at)
    render json: @posts
  end
end
```
This is great for clients (mobile/web apps) polling for updates.

## **5. Cache Control Headers (Optional)**

Add these for control over how long the client should cache:

```ruby
expires_in 10.minutes, public: true
```

## **Summary**

| Method       | Use For                          |
| ------------ | -------------------------------- |
| `fresh_when` | Sets `ETag` & `Last-Modified`    |
| `stale?`     | Returns true if resource changed |
| `expires_in` | Controls browser cache duration  |
