
# Request Body

## What You'll Learn

Request bodies allow you to send complex data to your API through POST, PUT, and other HTTP methods. Following the official FastAPI tutorial, you'll learn how to handle structured data with automatic validation.

By the end of this lesson, you'll understand how to:

- Understand what request bodies are and when to use them
- Create Pydantic models for data validation
- Handle POST requests with request bodies
- Use automatic data validation and serialization
- Combine request bodies with path and query parameters

## What are Request Bodies?

A request body is data sent by the client to your API. Unlike query parameters that go in the URL, request bodies can contain large amounts of structured data like JSON objects.

### When to Use Request Bodies

- Creating data: POST requests to create new items
- Updating data: PUT/PATCH requests to modify existing items
- Complex data: When you need to send more than simple parameters
- Sensitive data: Data that shouldn't appear in URLs

### Real-World Examples

- User registration: Sending user details (name, email, password)
- Creating posts: Sending blog post content and metadata
- File uploads: Sending file data and descriptions
- API calls: Sending complex query parameters to search APIs

## Pydantic Models

FastAPI uses **Pydantic models** to define the structure of request bodies. This provides automatic validation, serialization, and documentation.

### Basic Model Definition

From the official tutorial:

```py
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
```

**Key features**:

- **Type hints**: Define the expected data types
- **Optional fields**: Use `| None = None` for optional fields
- **Automatic validation**: FastAPI validates the data automatically
- **JSON conversion**: Automatically converts to/from JSON

### Using the Model

```py
@app.post("/items/")
def create_item(item: Item):
    return item
```

What happens:

- Client sends JSON data
- FastAPI validates it against the Item model
- Creates an Item instance
- Passes it to your function
- Automatically converts the response back to JSON

### Official Tutorial Examples

1. **Basic Request Body**

   ```py
   from fastapi import FastAPI
   from pydantic import BaseModel
   
   app = FastAPI()
   
   class Item(BaseModel):
       name: str
       description: str | None = None
       price: float
       tax: float | None = None
   
   @app.post("/items/")
   def create_item(item: Item):
       return item
   ```
   
   Test with JSON:
   
   ```json
   {
       "name": "Foo",
       "description": "A very nice Item",
       "price": 10.5,
       "tax": 1.5
   }
   ```

2. **Request Body + Path Parameters**

   ```py
   @app.post("/items/{item_id}")
   def create_item_with_id(item_id: int, item: Item):
       return {"item_id": item_id, **item.dict()}
   ```
   
   URL: POST /items/123 Body: Same JSON as above Response: Includes both the item_id and all item fields
   
3. **Request Body + Path + Query Parameters**

   ```py
   @app.put("/items/{item_id}")
   def update_item(item_id: int, item: Item, q: str | None = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
   ```
   
   URL: PUT /items/123?q=search Body: Item JSON Response: Combines all three parameter types

## Advanced Pydantic Features

### Field Validation

```py
from pydantic import BaseModel, Field

class Item(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    description: str | None = Field(None, max_length=500)
    price: float = Field(..., gt=0)  # Greater than 0
    tax: float | None = Field(None, ge=0)  # Greater than or equal to 0
```

### Nested Models

```py
class Image(BaseModel):
    url: str
    name: str

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []
    images: list[Image] | None = None
```

### Model Configuration

```py
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    
    class Config:
        schema_extra = {
            "example": {
                "name": "Foo",
                "description": "A very nice Item",
                "price": 35.4,
                "tax": 3.2,
            }
        }
```

### Best Practices

1. **Use Descriptive Model Names**

   ✅ Good - clear purpose
   ```py
   class UserRegistration(BaseModel):
       username: str
       email: str
       password: str
   ```
   
   ❌ Wrong - Avoid generic names
   ```py
   class Data(BaseModel):
       field1: str
       field2: str
   ```

2. **Add Field Descriptions**

   ✅ Good - Documented fields
   ```py
   class Item(BaseModel):
       name: str = Field(..., description="The name of the item")
       price: float = Field(..., gt=0, description="The price must be greater than zero")
   ```

3. **Use Appropriate Defaults**

   ✅ Good - sensible defaults
   ```py
   class Item(BaseModel):
       name: str
       description: str | None = None  # Optional description
       price: float
       tax: float | None = None        # Optional tax
       active: bool = True             # Default to active
   ```

### Common Beginner Mistakes

1. **Forgetting to Import BaseModel**

   ❌ Wrong - missing import
   ```py
   class Item:  # Should inherit from BaseModel
       name: str
       price: float
   ```

2. **Wrong Optional Syntax**

   ❌ Wrong - old syntax
   ```py
   from typing import Optional
   
   class Item(BaseModel):
       description: Optional[str] = None
   ```

   ✅ Correct - modern syntax
   ```py
   class Item(BaseModel):
       description: str | None = None
   ```

3. **Not Using the Model Parameter**

   ❌ Wrong - expecting raw dict
   ```py
   @app.post("/items/")
   def create_item(data: dict):  # Should use Pydantic model
       return data
   ```
   
   ✅ Correct - using Pydantic model
   ```py
   @app.post("/items/")
   def create_item(item: Item):
       return item
   ```

4. **Wrong HTTP Method**

   ❌ Wrong - using GET for data creation
   ```py
   @app.get("/items/")  # Should be POST
   def create_item(item: Item):
       return item
   ```
   
   ✅ Correct - using POST for creation
   ```py
   @app.post("/items/")
   def create_item(item: Item):
       return item
   ```

## How FastAPI Handles Request Bodies

### Automatic Validation

FastAPI automatically:

- Reads the request body as JSON
- Validates the data against your Pydantic model
- Converts to the appropriate types
- Creates an instance of your model
- Passes it to your function

### Error Handling

If validation fails, FastAPI automatically returns a detailed error response:

```json
{
    "detail": [
        {
            "loc": ["body", "price"],
            "msg": "field required",
            "type": "value_error.missing"
        }
    ]
}
```

### Interactive Documentation

FastAPI automatically generates interactive docs that show:

- The expected request body structure
- Field types and requirements
- Example values
- Try-it-out functionality

## What's Next?

Congratulations! You now understand how to handle request bodies with Pydantic models. This is essential for building APIs that can create and update data.

You've learned:

- How to create Pydantic models for data validation
- Using POST endpoints with request bodies
- Combining request bodies with path parameters
- Automatic validation and serialization

Coming up next: Query Parameters and String Validations - learning advanced query parameter features with validation!

## Additional Resources

- FastAPI Request Body Guide
- Pydantic Documentation
- HTTP Methods Explained
