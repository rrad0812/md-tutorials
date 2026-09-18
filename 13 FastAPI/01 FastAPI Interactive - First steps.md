# First Steps

## What You'll Learn

This lesson follows the official FastAPI tutorial's "First Steps" section exactly. You'll create the simplest possible FastAPI file and understand the basic concepts that power every FastAPI application.

By the end of this lesson, you'll understand how to:

- Create a minimal FastAPI application
- Understand the basic FastAPI structure
- Create a simple path operation
- Return JSON responses automatically

## The Simplest FastAPI File

According to the official FastAPI tutorial, the simplest FastAPI file looks like this:

```py
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```

That's it! This is a complete, working FastAPI application.

### Breaking It Down

Let's understand each part of this simple application:

1. **Import FastAPI**

   ```py
   from fastapi import FastAPI
   ```
   
   This imports the FastAPI class, which is the main class that provides all the functionality for your API.

2. **Create a FastAPI Instance**

   ```py
   app = FastAPI()
   ```
   
   Here we create an instance of the FastAPI class. This app variable will be the main point of interaction to create all your API endpoints.

3. **Create a Path Operation**

   ```py
   @app.get("/")
   async def root():
       return {"message": "Hello World"}
   ```
   
   This creates a path operation:
   
       Path: / (the root path)  
       Operation: GET (the HTTP method)  
       Function: root() (the Python function that handles requests)  

   The **@app.get("/")** tells FastAPI that the function right below is in charge of handling requests that go to:
   
       The path /  
       Using a GET operation
   
   > [!Note]  
   > What is a "Path Operation"?
   >
   > A path operation is one of these HTTP methods:
   > 
   >     GET: To read data
   >     POST: To create data
   >     PUT: To update data
   >     DELETE: To delete data
   >     ...and a few more
   > 
   > In HTTP protocol, you communicate to each path using one (or more) of these "methods".

   **In the Wild**

   When you go to a URL like https://example.com/items/foo in your browser, your browser is making a request to that URL using the GET method.
   
   So, in FastAPI, you would create a path operation to handle that:
   
   ```py
   @app.get("/items/foo")
   def read_item():
       return {"item_id": "foo"}
   ```
   
   **The Decorator**

   The `@app.get("/")` refers to the decorator `@app.get()`.
   
   A decorator modifies the function below it. In this case, it tells FastAPI:
   
   - This function handles GET requests
   - To the path /
   
   You can also use the other operations:
   
       @app.post()
       @app.put()
       @app.delete()
   
   And the more exotic ones:
   
       @app.options()
       @app.head()
       @app.patch()
       @app.trace()

## Running the Application

To run this application, you would save it in a file (like main.py) and run:

```sh
fastapi dev main.py
```

This would start the development server and you could visit:

    http://127.0.0.1:8000 - to see the JSON response
    http://127.0.0.1:8000/docs - to see the automatic interactive documentation

## Key Concepts from the Official Tutorial

1. **Automatic JSON Conversion**

   FastAPI automatically converts Python dictionaries to JSON responses. When you return {"Hello": "World"}, users receive proper JSON.

2. **Interactive Documentation**

   FastAPI automatically generates interactive API documentation. This is one of its most powerful features!

3. **Type Safety**

   Even this simple example benefits from Python's type system and FastAPI's validation.

4. **Standards-Based**

   FastAPI is based on OpenAPI (previously known as Swagger) and JSON Schema.

### Why This Matters

This simple example demonstrates FastAPI's core philosophy:

- Minimal code for maximum functionality
- Automatic features (JSON conversion, documentation)
- Standards-based approach
- Developer-friendly design

## What's Next?

Once you complete this lesson, you'll have created the exact same application shown in the official FastAPI tutorial!

Following the official tutorial progression, the next lessons will cover:

- Path Parameters: Making URLs dynamic
- Query Parameters: Adding optional parameters
- Request Body: Handling POST requests
- Query Parameters and String Validations: Advanced parameter handling

## Additional Resources

- FastAPI Official Tutorial - First Steps
- FastAPI Features Overview
- OpenAPI Specification
