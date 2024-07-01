# Controller Service Repository Pattern

This pattern of structuring code is a common practice in the software industry. It is a way to separate concerns and make the codebase maintainable such that if any new feature is added, it can be done with minimal changes to the existing codebase.

Let's see what each of these components does:

- **Controller**: This is the entrypoint to the application. If you're writing any APIs, this is where you write the code to handle the request. The only responsibility of the controller is to take external input, validate it, and pass it to the service layer. 

- **Service**: This is where you implement the business logic. The service layer is not concerned with how the request/response is handled or how the data is stored. It is only concerned with the business logic. This layer is called by the controller layer and it interacts with the repository layer to handle data.

- **Repository**: This is where you implement all the low level logic for managing the data. It generally provides a CRUD interface to interact with the database. 

In a typical microservice, this is how the flow of request would look like: 

1. The request comes to the controller.
2. The controller validates the request and calls the appropriate function in the service layer. Additionally, it may transform the request data to a format that the service layer understands.
3. The service layer performs the business logic. To do this, whenever any data operation is required, it interacts with the repository layer to manage data. 
4. The repository layer interacts with the database to manage data. 

This pattern separates the concerns of each layer and makes the codebase maintainable. 

Let's see how that can be useful for you. 

Consider the following scenario: 

```plaintext
You are building a backend application for a user management system. 

The application should have the following features:
    - Add a new user
    - Get a user by ID
    - Update a user
    - Delete a user
    - Get all users

And a user is defined by the following properties: 

    struct User {
        ID String,
        Name String,
        Email String,
        Age Integer
    }
```

You assume that the application won't have a lot of users initially, so you implement the application using simple **Flask + SQLite** stack.

Here is how you would structure your code in the Controller Service Repository pattern (I've removed all `__init__.py` files for brevity):

```bash
.
├── app
|   ├── controller
|   |   └── controller.py
|   ├── common
|   |   └── common.py
|   |   └── config.py
|   ├── models
|   |   └── user.py
|   ├── repository
|   |   └── repository.py
|   ├── service
|   |   └── service.py
|   ├── main.py
```

Here is, in breif, what each file implements:

`common/config.py`
```python
# Configure the values of HOST, PORT, DATABASE_ENDPOINT, etc. using environment variables
```

`models/common.py`
```python
# Define common functions like logger, etc.
```

`models/user.py`
```python
class User:
    # User model
    id: str
    name: str
    email: str
    age: int

class UserDB:
    # Database representation of User
    id: str
    name: str
    email: str
    age: int
    created_at: datetime
    updated_at: datetime

class UserRequest:
    # Request representation of User
    name: str
    email: str
    age: int
```

`repository/repository.py`
```python
def add_user(user: User):
    # Add a new user to the database
    # Generate the UserDB object from the User object with created_at and updated_at set to current time
    # Add the user_db to the database


def get_user_by_id(user_id: str) -> User:
    # Get a user by ID from the database
    # Convert the UserDB object to User object and return it

def update_user(user_id: str, user: User) -> User:
    # Update a user in the database
    # Generate the updated UserDB object from the User object with updated_at set to current time
    # Update the user_db in the database
    return user


def delete_user(user_id: str):
    # Delete a user from the database
    # Delete the user_db from the database

def get_users() -> List[User]:
    # Get all users from the database
    # Convert the UserDB objects to User objects and return them
```

`service/service.py`
```python
def add_user(user_request: UserRequest) -> User:
    # Generate the User object from the UserRequest object 
    # Call the repository function to add the user to the database
    # Return the User object

def get_user_by_id(user_id: str) -> User:
    # Call the repository function to get the user by ID
    # Return the User object

def update_user(user_id: str, user_request: UserRequest) -> User:
    # Generate the User object from the UserRequest object 
    # Call the repository function to update the user in the database
    # Return the User object

def delete_user(user_id: str):
    # Call the repository function to delete the user from the database

def get_users() -> List[User]:
    # Call the repository function to get all users from the database
    # Return the list of User objects
```

`controller/controller.py`
```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/user', methods=['POST'])
def add_user():
    # Get the user data from the request
    # Call the service function to add the user
    # Return the user data

@app.route('/user/<user_id>', methods=['GET'])
def get_user_by_id(user_id):
    # Call the service function to get the user by ID
    # Return the user data

@app.route('/user/<user_id>', methods=['PUT'])
def update_user(user_id):
    # Get the user data from the request
    # Call the service function to update the user
    # Return the user data

@app.route('/user/<user_id>', methods=['DELETE'])
def delete_user(user_id):
    # Call the service function to delete the user
    # Return the success message

@app.route('/users', methods=['GET'])
def get_users():
    # Call the service function to get all users
    # Return the list of users
```

`main.py`
```python
from controller.controller import app
from common.config import HOST, PORT

if __name__ == '__main__':
    app.run(host=HOST, port=PORT)
```

Now let's first understand the importance of the **Repository layer**.

You notice that the application is working fine, but the database is slow. You decide to switch to a faster database like **PostgreSQL**. 

**OR**

You decide to push your application to cloud and use a managed database service like **AWS RDS**.

To do this, you would only need to change the implementation of the repository functions in the `repository.py` file. The service and controller layers would remain the same. 

Now let's understand the importance of the **Service layer**.

You decide to add a new feature to the application. You want to call another microservice to send an email to the user when a new user is added.

To do this, you would only need to change the implementation of the service functions in the `service.py` file. The controller and repository layers would remain the same.

To understand the importance of the **Controller Layer**, let's say you decide to switch from **Flask** to **FastAPI**. You would only need to change the implementation of the controller functions in the `controller.py` file. The service and repository layers would remain the same.

This is the power of the Controller Service Repository pattern. It makes your codebase maintainable and scalable.

You can use this pattern to structure your code in any language or framework.