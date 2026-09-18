# Query Parameters

## What You'll Learn

Query parameters are a powerful way to make your APIs flexible by accepting optional parameters in the URL. Following the official FastAPI tutorial, you'll learn how to handle URL parameters that come after the ? symbol.

By the end of this lesson, you'll understand how to:

- Understand what query parameters are and how they work
- Create endpoints that accept optional query parameters
- Set default values for query parameters
- Use type hints for automatic query parameter validation
- Handle both path and query parameters together

## What are Query Parameters?

Query parameters are the optional parts of a URL that come after the ? symbol. They're used to modify or filter the response without changing the core endpoint.

### Real-World Examples

You see query parameters everywhere:

- Google Search: google.com/search?q=fastapi&lang=en
- YouTube: youtube.com/results?search_query=python
- Online stores: store.com/products?category=electronics&sort=price
- Pagination: api.com/users?page=2&limit=20

## How Query Parameters Work

### Basic Syntax

When you declare other function parameters that are not part of the path parameters, they are automatically interpreted as **query** parameters.

```py
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]

How it works:

    /items/ → Uses defaults: skip=0, limit=10
    /items/?skip=20 → Uses: skip=20, limit=10 (default)
    /items/?skip=20&limit=10 → Uses: skip=20, limit=10
```

## Query Parameter Features

### Default Values

Query parameters can have default values, making them optional:

```py
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

### Optional Parameters

You can make parameters truly optional by setting them to None:

```py
@app.get("/items/{item_id}")
def read_item(item_id: str, q: str | None = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

URL Examples:

    /items/foo → {"item_id": "foo"}
    /items/foo?q=search → {"item_id": "foo", "q": "search"}

### Type Conversion

FastAPI automatically converts query parameters to the specified types:

```py
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10, active: bool = True):
    return {
        "skip": skip,      # Converted to int
        "limit": limit,    # Converted to int  
        "active": active   # Converted to bool
    }
```

URL Examples:

    /items/?skip=5&limit=20&active=false
    /items/?active=1 (1 becomes True)
    /items/?active=0 (0 becomes False)

### Official Tutorial Examples

1. **Simple Query Parameters**

   From the official tutorial:
   
   ```py
   fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]
   
   @app.get("/items/")
   def read_items(skip: int = 0, limit: int = 10):
       return fake_items_db[skip : skip + limit]
   ```
   
   Test it:
   
       /items/ → Returns first 10 items
       /items/?skip=1 → Skips first item, returns next 10
       /items/?limit=2 → Returns first 2 items

2. **Combining Path and Query Parameters**

   ```py
   @app.get("/items/{item_id}")
   def read_item(item_id: str, q: str | None = None):
       if q:
           return {"item_id": item_id, "q": q}
       return {"item_id": item_id}
   ```
   
   Test it:
   
       /items/foo → {"item_id": "foo"}
       /items/foo?q=search → {"item_id": "foo", "q": "search"}

3. **Multiple Query Parameters**

   ```py
   @app.get("/users/")
   def read_users(skip: int = 0, limit: int = 10, active: bool = True):
       # In a real app, you'd filter from a database
       return {
           "skip": skip,
           "limit": limit, 
           "active_only": active,
           "message": f"Showing {limit} users, skipping {skip}, active: {active}"
       }
   ```

## Advanced Query Parameter Features

### Required Query Parameters

Make query parameters required by not providing a default value:

```py
@app.get("/search/")
def search_items(q: str):  # Required - no default value
    return {"query": q, "results": []}
```

### Multiple Types

```py
@app.get("/products/")
def get_products(
    category: str = "all",
    min_price: float = 0.0,
    max_price: float = 1000.0,
    in_stock: bool = True
):
    return {
        "category": category,
        "price_range": [min_price, max_price],
        "in_stock_only": in_stock
    }
```

### List Parameters

```py
@app.get("/items/")
def read_items(tags: list[str] = []):
    return {"tags": tags}
```

    URL: /items/?tags=python&tags=fastapi&tags=tutorial


### Best Practices

1. **Use Meaningful Default Values**

   ✅ Good - sensible defaults
   ```py
   @app.get("/items/")
   def read_items(skip: int = 0, limit: int = 10):
       pass
   ```

   ❌ Wrong - Avoid unclear defaults
   ```py
   @app.get("/items/")
   def read_items(skip: int = 999, limit: int = 1):
       pass
   ```

2. **Use Type Hints**

   ✅ Good - Automatic validation
   ```py
   @app.get("/items/")
   def read_items(skip: int = 0, limit: int = 10):
       pass
   ```

   ❌ Wrong - Missing, no validation
   ```py
   @app.get("/items/")
   def read_items(skip=0, limit=10):
       pass
   ```

3. **Handle Optional Parameters Properly**

   ✅ Good - explicit None handling
   ```py
   @app.get("/items/{item_id}")
   def read_item(item_id: str, q: str | None = None):
       if q:
           return {"item_id": item_id, "q": q}
       return {"item_id": item_id}
   ```

### Common Beginner Mistakes

1. **Forgetting Default Values**

   ❌ Wrong - Required parameters become mandatory
   ```py
   @app.get("/items/")
   def read_items(skip: int, limit: int):  # Users must provide both
       pass
   ```
   
   ✅ Correct - optional with defaults
   ```py
   @app.get("/items/")
   def read_items(skip: int = 0, limit: int = 10):
       pass
   ```

2. **Wrong Optional Syntax**

   ❌ Wrong - old Python syntax
   ```py
   @app.get("/items/{item_id}")
   def read_item(item_id: str, q: Optional[str] = None):
       pass
   ```
   
   ✅ Correct - modern Python syntax
   ```py
   @app.get("/items/{item_id}")
   def read_item(item_id: str, q: str | None = None):
       pass
   ```

3. **Not Handling None Values**

   ❌ Wrong - Will crash if q is None
   ```py
   @app.get("/items/{item_id}")
   def read_item(item_id: str, q: str | None = None):
       return {"item_id": item_id, "query_length": len(q)}  # Error if q is None
   ```
   
   ✅ Correct - check for None
   ```py
   @app.get("/items/{item_id}")
   def read_item(item_id: str, q: str | None = None):
       if q:
           return {"item_id": item_id, "q": q}
       return {"item_id": item_id}
   ```

## What's Next?

Congratulations! You now understand how to use query parameters to make your APIs flexible and user-friendly. This is essential for building real-world APIs that need filtering, pagination, and customization.

You've learned:

- How query parameters work in URLs
- Setting default values for optional parameters
- Combining path and query parameters
- Handling None values for optional parameters

Coming up next: Request Body - learning how to handle POST requests with data in the request body!

## Additional Resources

- FastAPI Query Parameters Guide
- HTTP Query Parameters Explained
- REST API Query Parameter Best Practices
